---
title: 'Gunicorn, an inside view of it'
date: 2020-04-10 07:13:00
tags: ['Python', 'Web Server', 'Gunicorn']
---
Gunicorn 是一个Python Web Server实现，兼容WSGI协议。其迁移自Ruby世界的Unicorn。在我们的生产环境中，有大量使用Gunicorn作为Web 服务器，所以本文分析Gunicorn的相关几个问题： Server model，Async worker.
<!--more-->

### Server model
如果你写过socket编程，应该知道socket server一般是如何支持并发的，比如在socket server启动后，会对每一个进来的请求fork一个新的进程或者线程去处理它。但是这种实现存在问题，比如高并发量情况下，socket server不得不创建很多进程或者线程，创建进程或者线程需要sys call，大量的创建或者销毁会使系统负载过高，进而处理请求能力下降。极限情况是可以创建的进程线程数量会达到OS的limit进而程序崩溃，或者大量的进程线程之间切换耗费太多资源（我们知道OS 进程管理器在做switch操作的时候，需要保存当前进程的状态到内存数据结构中，然后将ready的进程相关资源load进入相关处理器）。  

一种改进的方案是池化复用。创建固定数量的进程或者线程，所有并发进来的请求都轮流使用池子中的进程线程处理。这种情况下，进程线程只会创建一次，后续会被请求复用，在整个服务器生命周期的最后阶段才会被销毁。这种优化思想在工程中的很多地方都有使用，比如Web app到数据库会有一个连接池，Django celery模型中会使用消息队列池化待处理的任务等等。

如果大部分请求处理涉及IO，那么更加优化的方案是通过OS提供的IO复用机制（select/poll/epoll）来处理。在一个服务主循环中，IO multiplexing会返回当前可读可写的socket，这样就可以直接使用对应handler处理方法处理read、write ready的socket。这种方式的优点是单进程即可支持很高的并发量，且没有进程创建销毁等操作，资源占用率比较低。但是它一定是处理IO bound task才会有这种优势。

Gunicorn 采用的是池化复用的技巧，通过服务启动时候，根据你的配置信息，由master进程提前创建好对应的worker进程，然后后续的请求都会通过轮流复用被这些worker进程处理。从单个sync worker进程的视角来看，在某一个时刻，其只能处理一个请求，其它请求要么被其它worker处理中，要么在socket队列pending中。下面是一张其逻辑视图：

![sync worker](https://www.spirulasystems.com/wp-content/plugins/phastpress/phast.php?service=images&width=512&height=290&src=https%3A%2F%2Fwww.spirulasystems.com%2Fwp-content%2Fuploads%2F2015%2F01%2Fsync_worker_type1.png&cacheMarker=1557312233-22541&token=a271e359b84a7e6c)

总体上，master进程负责启动worker进程并对其管理，包括创建、销毁、新增、减少等操作。worker进程负责监听端口处理请求，其创建启动的时候，会实例化配置好的Python Web application，将接受的http 请求parse，然后调用web application处理，得到返回结果后整理成Http Response通过tcp返回给客户端。如下图：

![master-worker](https://circus.readthedocs.org/en/0.5/_images/classical-stack.png)  

#### source code analysis

The following code in Gunicorn show it show to start master and workers process. when you type following command in bash: `gunicorn -w 4 myapp:app`, gunicorn will enter `Arbiter.run` method, it first start the master by initialize `arbiter` class instance, like set pid and create socket. it then will create worker process with `Arbiter.spawn_worker` method, actually use os.fork to create or clone an child process. also in `Arbiter.manage_workers` you can see how master process manages worker process.


```Python
class Arbiter(object):
    """
    Arbiter maintain the workers processes alive. It launches or
    kills them if needed. It also manages application reloading
    via SIGHUP/USR2.
    """
    ...

    def run(self):
        "Main master loop."
        self.start()
        util._setproctitle("master [%s]" % self.proc_name)

        try:
            self.manage_workers()

            ...
        except Exception:
            self.log.info("Unhandled exception in main loop",
                          exc_info=True)
            self.stop(False)
            if self.pidfile is not None:
                self.pidfile.unlink()
            sys.exit(-1)

    def start(self):
        """\
        Initialize the arbiter. Start listening and set pidfile if needed.
        """
        ...

        self.init_signals()

        if not self.LISTENERS:
            fds = None
            listen_fds = systemd.listen_fds()
            if listen_fds:
                self.systemd = True
                fds = range(systemd.SD_LISTEN_FDS_START,
                            systemd.SD_LISTEN_FDS_START + listen_fds)

            elif self.master_pid:
                fds = []
                for fd in os.environ.pop('GUNICORN_FD').split(','):
                    fds.append(int(fd))

            self.LISTENERS = sock.create_sockets(self.cfg, self.log, fds)

    def manage_workers(self):
        """\
        Maintain the number of workers by spawning or killing
        as required.
        """
        if len(self.WORKERS) < self.num_workers:
            self.spawn_workers()

        workers = self.WORKERS.items()
        workers = sorted(workers, key=lambda w: w[1].age)
        while len(workers) > self.num_workers:
            (pid, _) = workers.pop(0)
            self.kill_worker(pid, signal.SIGTERM)

        active_worker_count = len(workers)
        if self._last_logged_active_worker_count != active_worker_count:
            self._last_logged_active_worker_count = active_worker_count
            self.log.debug("{0} workers".format(active_worker_count),
                           extra={"metric": "gunicorn.workers",
                                  "value": active_worker_count,
                                  "mtype": "gauge"})

    def spawn_worker(self):
        self.worker_age += 1
        worker = self.worker_class(self.worker_age, self.pid, self.LISTENERS,
                                   self.app, self.timeout / 2.0,
                                   self.cfg, self.log)
        self.cfg.pre_fork(self, worker)
        pid = os.fork()
        if pid != 0:
            worker.pid = pid
            self.WORKERS[pid] = worker
            return pid

        # Do not inherit the temporary files of other workers
        for sibling in self.WORKERS.values():
            sibling.tmp.close()

        # Process Child
        worker.pid = os.getpid()
        try:
            util._setproctitle("worker [%s]" % self.proc_name)
            self.log.info("Booting worker with pid: %s", worker.pid)
            self.cfg.post_fork(self, worker)
            worker.init_process()
            sys.exit(0)
        except SystemExit:
            raise
        except AppImportError as e:
            self.log.debug("Exception while loading the application",
                           exc_info=True)
            print("%s" % e, file=sys.stderr)
            sys.stderr.flush()
            sys.exit(self.APP_LOAD_ERROR)
        except Exception:
            self.log.exception("Exception in worker process")
            if not worker.booted:
                sys.exit(self.WORKER_BOOT_ERROR)
            sys.exit(-1)
        finally:
            self.log.info("Worker exiting (pid: %s)", worker.pid)
            try:
                worker.tmp.close()
                self.cfg.worker_exit(self, worker)
            except Exception:
                self.log.warning("Exception during worker exit:\n%s",
                                 traceback.format_exc())
```

### Async worker type
Gunicorn支持多种worker type，例如Sync，Async，Tonardo，AsyncIO类型的worker。Sync的worker最简单，在上节已经阐述。  

Async worker是通过Gevent or Eventlet实现，Gevent & Eventlet都是通过Greenlet实现， Greenlet是Python的一个C extension module，其本质是C的co-routine。总体上来说Gevent就是通过co-rotine + 事件循环（libev）实现了异步模型。如果对这里的实现特别感兴趣，推荐看David beezly的[curious course on corontines and concurrency](https://www.youtube.com/watch?v=Z_OAlIhXziw) and Kavya Joshi的  [a deep dive into how gevent works](https://www.youtube.com/watch?v=GunMToxbE0E) 

Async模式下Gunicorn处理请求的流程图如下：

![async worker](https://www.spirulasystems.com/wp-content/plugins/phastpress/phast.php?service=images&width=275&height=300&src=https%3A%2F%2Fwww.spirulasystems.com%2Fwp-content%2Fuploads%2F2015%2F01%2Fsync_worker_type2-275x300.png&cacheMarker=1557312233-55428&token=c96103d27e004eef)   
一个async worker可以并发的处理很多个请求，每个请求实际上被一个coroutine处理，当其进入IO等待，会主动yield出控制权，这样下一个coroutine就可以执行。当IO ready，会触发事件，事件循环会将对应的coroutine的从yeild出控制权的地方恢复执行。  

Async worker的corotines会运行在一个进程中，通过来回切换函数来充分利用CPU，减少IO等待时间，其在某个时刻只会有一个routine在cpu上运行。相比于进程线程，coroutine更加轻量，因为其操作的维度是函数而不是整个进程线程。


Gunicorn have an base worker type for every support worker, that is `base.Wrorker`, from the def of it, you can see child worker must implement `run init_process`.

```Python
# base.py
class Worker(object):
    def __init__(self, age, ppid, sockets, app, timeout, cfg, log):
        """ called in pre-fork """
        ...

    def notify(self):
        """ inform master process that this worker is alive """
        ...

    def run(self):
        """\
        This is the mainloop of a worker process. You should override
        this method in a subclass to provide the intended behaviour
        for your particular evil schemes.
        """
        raise NotImplementedError()

    def init_process(self):
        """\
        If you override this method in a subclass, the last statement
        in the function should be to call this method with
        super().init_process() so that the ``run()`` loop is initiated.
        """
        ...
        self.wait_fds = self.sockets + [self.PIPE[0]]

        self.log.close_on_exec()

        self.init_signals()

        self.load_wsgi()
        ...
        # Enter main run loop
        self.booted = True
        self.run()
```
but for async worker, it will have second parent base type which is `base_async.AsyncWorker`, which is inherit from `base.Worker`.
it maily defs how to process or handle the request. also Gunicorn have the corresponding SyncWorker defined in `sync.py`.
```Python
# base_async.py

class AsyncWorker(base.Worker):
    def handle(self, listener, client, addr):
        req = None
        try:
            parser = http.RequestParser(self.cfg, client)
            try:
                listener_name = listener.getsockname()
                if not self.cfg.keepalive:
                    req = next(parser)
                    self.handle_request(listener_name, req, client, addr)
                else:
                    ...

    def handle_request(self, listener_name, req, sock, addr):
        request_start = datetime.now()
        environ = {}
        resp = None
        try:
            self.cfg.pre_request(self, req)
            resp, environ = wsgi.create(req, sock, addr,
                                        listener_name, self.cfg)
            environ["wsgi.multithread"] = True
            ...
```
Gevent worker implementation is defined in `ggevent.py`, which will implement following methods: `run`, `init_process`, `notify`

```Python
# ggevent.py
class GeventWorker(AsyncWorker):
    def notify(self):
        super().notify()
        if self.ppid != os.getppid():
            self.log.info("Parent changed, shutting down: %s", self)
            sys.exit(0)

    def init_process(self):
        self.patch()
        hub.reinit()
        super().init_process()

    def run(self):
        servers = []
        ssl_args = {}

        if self.cfg.is_ssl:
            ssl_args = dict(server_side=True, **self.cfg.ssl_options)

        for s in self.sockets:
            s.setblocking(1)
            pool = Pool(self.worker_connections)
            if self.server_class is not None:
                environ = base_environ(self.cfg)
                environ.update({
                    "wsgi.multithread": True,
                    "SERVER_SOFTWARE": VERSION,
                })
                server = self.server_class(
                    s, application=self.wsgi, spawn=pool, log=self.log,
                    handler_class=self.wsgi_handler, environ=environ,
                    **ssl_args)
            else:
                hfun = partial(self.handle, s)
                server = StreamServer(s, handle=hfun, spawn=pool, **ssl_args)
                if self.cfg.workers > 1:
                    server.max_accept = 1

            server.start()
            servers.append(server)

        while self.alive:
            self.notify()
            gevent.sleep(1.0)

        try:
            # Stop accepting requests
            for server in servers:
                if hasattr(server, 'close'):  # gevent 1.0
                    server.close()
                if hasattr(server, 'kill'):  # gevent < 1.0
                    server.kill()

            # Handle current requests until graceful_timeout
            ts = time.time()
            while time.time() - ts <= self.cfg.graceful_timeout:
                accepting = 0
                for server in servers:
                    if server.pool.free_count() != server.pool.size:
                        accepting += 1

                # if no server is accepting a connection, we can exit
                if not accepting:
                    return

                self.notify()
                gevent.sleep(1.0)

            # Force kill all active the handlers
            self.log.warning("Worker graceful timeout (pid:%s)" % self.pid)
            for server in servers:
                server.stop(timeout=1)
        except Exception:
            pass
```

### 总结
一般来说我们的web application要query db，从第三方api拉取数据等，所以很大概率上这是一个IO bound的task，这就比较适合用Gunicorn的Async worker（gevent）来处理。如果你的服务是一个CPU bound的task，那么用Gunicorn的Sync task就会达到很好的性能。

### Reference
[A deep dive into how gevent works](https://www.youtube.com/watch?v=GunMToxbE0E)  
[Gunicorn worker types](https://www.spirulasystems.com/blog/2015/01/20/gunicorn-worker-types/)  
[Understand Gunicorn's async worker models](https://words.volant.is/articles/understanding-gunicorns-async-worker-concurrency-model/)  
[benoitc/gunicorn](https://github.com/benoitc/gunicorn)  
[Gunicorn Design](https://docs.gunicorn.org/en/stable/design.html)  
[Curious course on corontines and concurrency](https://www.youtube.com/watch?v=Z_OAlIhXziw)  
