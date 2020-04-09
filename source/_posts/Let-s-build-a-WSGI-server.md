---
title: Let's build a WSGI server
date: 2020-03-22 19:58:34
tags: [WSGI, Python, Concurrency, Select]
---
Let's build a wsgi server ground up with sockets, multi-processing or Select/Poll/Epoll. to achive this, you need familiar with Python's socket programming, Python's cucurreny model, Linux Non-blocking I/O.
<!--more-->

WSGI(web server gateway interface) is a protocol in Python web programming, it defines how web application and web server communicate. with this protocol, you can chose various combinations about web servers and web frameworks which are wsgi-compatiable. for example, you can use Gunicorn + Django, or even Bjoern + Flask etc...

### What's WSGI Server
WSGI server is the server side wsgi protocol implementation. which is responsible accepting connections and invoke web application to process the result though wsgi protocol, and finally return the result back to clients. with wsgi server concentrate on accepting connections, wsgi framework(application) can focus on your bussiness logic.   

To summarize, as an wsgi server, it must contains following info:
* compose `env` variables which will be used in web framework
    ```Python
    environ['wsgi.input']        = sys.stdin
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True
    environ['wsgi.url_schema']   = 'http'
    environ['REQUEST_METHOD']    = 'GET'
    environ['PATH_INFO']         = '/hello'
    environ['SERVER_NAME']       = 'localhost'
    environ['SERVER_PORT']       = 8888
    ```
* start_response callable which will be called in web framework
    ```Python
    def start_response(status, response_header, exec_info=None):
        """
        status is the http status code, response_header is the header web framework wants to added
        """
        pass
    ```
for more detail, please reference: [PEP 333: Python web server gateway interfaces v1.0](https://www.python.org/dev/peps/pep-0333/#implementation-application-notes)


### Simple TCP echo server
As classic socket programming said, a socket server needs to bind/listen/accept/recv/send/close, a socket client needs to connect/send/recv/close. here is a simple example:
```Python
import socket


class SimpleTcpServer(object):
    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    request_queue_size = 1024

    def __init(self, server_address):
        # Create a listening socket
        self.listen_socket = listen_socket = socket.socket(
            self.address_family,
            self.socket_type
        )
        # Allow to reuse the same address
        listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # Bind
        listen_socket.bind(server_address)
        # Activate
        listen_socket.listen(self.request_queue_size)
        # Get server host name and port
        host, port = self.listen_socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port

    def server_forever(self):
        while True:
            self.client_connection, client_address = self.listen_socket.accept()
            self.handle_one_request()

    def handle_one_request(self):
        request_data = self.client_connection.recv(1024)
        self.client_connection.sendall(request_data)
        self.client_connection.close()
```

### A simple wsgi server
The simple wsgi server will listen on port, waitting for incoming connection to accept, for each connection, wsgi server will invoke web framework process(which is the main entrance implemented in web framework) the request and wait until web framework finish processing it and return the result to client. here is a simple example:
```Python
import io
import socket
import sys


class WSGIServer(object):

    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    request_queue_size = 1

    def __init__(self, server_address):
        # Create a listening socket
        self.listen_socket = listen_socket = socket.socket(
            self.address_family,
            self.socket_type
        )
        # Allow to reuse the same address
        listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # Bind
        listen_socket.bind(server_address)
        # Activate
        listen_socket.listen(self.request_queue_size)
        # Get server host name and port
        host, port = self.listen_socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port
        # Return headers set by Web framework/Web application
        self.headers_set = []

    def set_app(self, application):
        self.application = application

    def serve_forever(self):
        listen_socket = self.listen_socket
        while True:
            # New client connection
            self.client_connection, client_address = listen_socket.accept()
            # Handle one request and close the client connection. Then
            # loop over to wait for another client connection
            self.handle_one_request()

    def handle_one_request(self):
        request_data = self.client_connection.recv(1024)
        self.request_data = request_data = request_data.decode('utf-8')
        # Print formatted request data a la 'curl -v'
        print(''.join(
            f'< {line}\n' for line in request_data.splitlines()
        ))

        self.parse_request(request_data)

        # Construct environment dictionary using request data
        env = self.get_environ()

        # It's time to call our application callable and get
        # back a result that will become HTTP response body
        result = self.application(env, self.start_response)

        # Construct a response and send it back to the client
        self.finish_response(result)

    def parse_request(self, text):
        request_line = text.splitlines()[0]
        request_line = request_line.rstrip('\r\n')
        # Break down the request line into components
        (self.request_method,  # GET
         self.path,            # /hello
         self.request_version  # HTTP/1.1
         ) = request_line.split()

    def get_environ(self):
        env = {}
        # The following code snippet does not follow PEP8 conventions
        # but it's formatted the way it is for demonstration purposes
        # to emphasize the required variables and their values
        #
        # Required WSGI variables
        env['wsgi.version']      = (1, 0)
        env['wsgi.url_scheme']   = 'http'
        env['wsgi.input']        = io.StringIO(self.request_data)
        env['wsgi.errors']       = sys.stderr
        env['wsgi.multithread']  = False
        env['wsgi.multiprocess'] = False
        env['wsgi.run_once']     = False
        # Required CGI variables
        env['REQUEST_METHOD']    = self.request_method    # GET
        env['PATH_INFO']         = self.path              # /hello
        env['SERVER_NAME']       = self.server_name       # localhost
        env['SERVER_PORT']       = str(self.server_port)  # 8888
        return env

    def start_response(self, status, response_headers, exc_info=None):
        # Add necessary server headers
        server_headers = [
            ('Date', 'Mon, 15 Jul 2019 5:54:48 GMT'),
            ('Server', 'WSGIServer 0.2'),
        ]
        self.headers_set = [status, response_headers + server_headers]
        # To adhere to WSGI specification the start_response must return
        # a 'write' callable. We simplicity's sake we'll ignore that detail
        # for now.
        # return self.finish_response

    def finish_response(self, result):
        try:
            status, response_headers = self.headers_set
            response = f'HTTP/1.1 {status}\r\n'
            for header in response_headers:
                response += '{0}: {1}\r\n'.format(*header)
            response += '\r\n'
            for data in result:
                response += data.decode('utf-8')
            # Print formatted response data a la 'curl -v'
            print(''.join(
                f'> {line}\n' for line in response.splitlines()
            ))
            response_bytes = response.encode()
            self.client_connection.sendall(response_bytes)
        finally:
            self.client_connection.close()

```

### Concurrent with multi-processing
The upper wsgi server can process a request at a time, cuz it is a single process and will block in while loop, as following code shows:
```Python
def serve_forever(self):
    listen_socket = self.listen_socket
    while True:
        # New client connection
        self.client_connection, client_address = listen_socket.accept()
        # Handle one request and close the client connection. Then
        # loop over to wait for another client connection
        self.handle_one_request()
```
`handle_one_request` will block the while loop while processing the current connection. the time spend on blocking depends on your web application's corresonding method's processing speed.  

if the first connection didn't finished, the second connection will be blocked by the first one. how can we let the wsgi server handle multiple connections at the same time? the answer is multi-processing.  

here is an example of multi-processing version, which will fork an new python process for each incoming connections.
```Python
class WSGIServer(object):

    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    request_queue_size = 1024

    def __init__(self, server_address):
        # Create a listening socket
        self.listen_socket = listen_socket = socket.socket(
            self.address_family,
            self.socket_type
        )
        # Allow to reuse the same address
        listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # Bind
        listen_socket.bind(server_address)
        # Activate
        listen_socket.listen(self.request_queue_size)
        # Get server host name and port
        host, port = self.listen_socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port
        # Return headers set by Web framework/Web application
        self.headers_set = []

    def __reap_children(selflili, signum, frame):
        """
        collect zombie children
        """
        while True:
            try:
                # wait for all children, do not block
                pid, status = os.waitpid(-1, os.WNOHANG)
                if pid == 0:  # no more zombies
                    break
            except:
                # usally this would be OSError exception
                # with errno attribute set to errno.ECHILD
                # which means there are no more children
                break

    def set_app(self, application):
        self.application = application

    def serve_forever(self):
        signal.signal(signal.SIGCHLD, self.__reap_children)
        listen_socket = self.listen_socket
        while True:
            # New client connection
            try:
                self.client_connection, client_address = listen_socket.accept()
            except IOError as e:
                code, msg = e.args
                if code == errno.EINR:
                    continue
                else:
                    raise
            pid = os.fork()
            if pid == 0:  # child
                self.listen_socket.close()
                self.handle_one_request()
                os._exit(0)
            else:  # parent
                self.client_connection.close()


    def handle_one_request(self):
        request_data = self.client_connection.recv(1024)
        self.request_data = request_data = request_data.decode('utf-8')
        # Print formatted request data a la 'curl -v'
        print(''.join(
            f'< {line}\n' for line in request_data.splitlines()
        ))

        self.parse_request(request_data)

        # Construct environment dictionary using request data
        env = self.get_environ()

        # It's time to call our application callable and get
        # back a result that will become HTTP response body
        result = self.application(env, self.start_response)

        # Construct a response and send it back to the client
        self.finish_response(result)

    def parse_request(self, text):
        request_line = text.splitlines()[0]
        request_line = request_line.rstrip('\r\n')
        # Break down the request line into components
        (self.request_method,  # GET
         self.path,            # /hello
         self.request_version  # HTTP/1.1
         ) = request_line.split()

    def get_environ(self):
        env = {}
        # The following code snippet does not follow PEP8 conventions
        # but it's formatted the way it is for demonstration purposes
        # to emphasize the required variables and their values
        #
        # Required WSGI variables
        env['wsgi.version']      = (1, 0)
        env['wsgi.url_scheme']   = 'http'
        env['wsgi.input']        = io.StringIO(self.request_data)
        env['wsgi.errors']       = sys.stderr
        env['wsgi.multithread']  = False
        env['wsgi.multiprocess'] = False
        env['wsgi.run_once']     = False
        # Required CGI variables
        env['REQUEST_METHOD']    = self.request_method    # GET
        env['PATH_INFO']         = self.path              # /hello
        env['SERVER_NAME']       = self.server_name       # localhost
        env['SERVER_PORT']       = str(self.server_port)  # 8888
        return env

    def start_response(self, status, response_headers, exc_info=None):
        # Add necessary server headers
        server_headers = [
            ('Date', 'Mon, 15 Jul 2019 5:54:48 GMT'),
            ('Server', 'WSGIServer 0.2'),
        ]
        self.headers_set = [status, response_headers + server_headers]
        # To adhere to WSGI specification the start_response must return
        # a 'write' callable. We simplicity's sake we'll ignore that detail
        # for now.
        # return self.finish_response

    def finish_response(self, result):
        try:
            status, response_headers = self.headers_set
            response = f'HTTP/1.1 {status}\r\n'
            for header in response_headers:
                response += '{0}: {1}\r\n'.format(*header)
            response += '\r\n'
            for data in result:
                response += data.decode('utf-8')
            # Print formatted response data a la 'curl -v'
            print(''.join(
                f'> {line}\n' for line in response.splitlines()
            ))
            response_bytes = response.encode()
            self.client_connection.sendall(response_bytes)
        finally:
            self.client_connection.close()
```

### Concurrent with Linux's Non-blocking I/O
The multi-processing version still have problems, like when there are massive connections, the server will create massive processes for each connection. this will exaust the server resources, and also process-switching is expensive. to achive that, you will think using process pool.  

But let's think from another side, the server maily handles I/O tasks, i.e waitting socket to be readable, read the data and process it, and write to socket. for this kind of task, we use Non-Blocking I/O.

Suppose you’re a webserver. Every time you accept a connection with the accept system call (here’s the man page), you get a new file descriptor representing that connection.

If you’re a web server, you might have thousands of connections open at the same time. You need to know when people send you new data on those connections, so you can process and respond to them.

You could have a loop that basically does:
```Python
for x in open_connections:
    if has_new_input(x):
        process_input(x)
```

The problem with this is that it can waste a lot of CPU time. Instead of spending all CPU time to ask “are there updates now? how about now? how about now? how about now?“, instead we’d rather just ask the Linux kernel “hey, here are 100 file descriptors. Tell me when one of them is updated!“.

The 3 system calls that let you ask Linux to monitor lots of file descriptors are poll, epoll and select. Let’s start with poll and select because that’s where the chapter started.

I am not gonna talk the details between `select/poll/epoll`, you can read this post for more: [async io on linux select/poll/epoll](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/).

Python's select module have support for Linux's `select/poll/epoll`, let's use select rewrite our wsgi server:

```Python
class WSGIServer(object):

    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    request_queue_size = 1024

    def __init__(self, server_address):
        # Create a listening socket
        self.listen_socket = listen_socket = socket.socket(
            self.address_family,
            self.socket_type
        )
        # Allow to reuse the same address
        listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        listen_socket.setblocking(0)
        # Bind
        listen_socket.bind(server_address)
        # Activate
        listen_socket.listen(self.request_queue_size)
        # Get server host name and port
        host, port = self.listen_socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port
        # Return headers set by Web framework/Web application
        self.headers_set = []

    def set_app(self, application):
        self.application = application

    def serve_forever(self):
        rlist, wlist, elist = [self.listen_socket], [], []

        # listen_socket = self.listen_socket
        while True:
            readables, writables, exceptions = select.select(rlist, wlist, elist)
            for sock in readables:
                if sock is self.listen_socket:
                    try:
                        conn, client_address = self.listen_socket.accept()
                    except IOError as e:
                        code, msg = e.args
                        if code == errno.EINTR:
                            continue
                        else:
                            raise
                    rlist.append(conn)
                else:
                    try:
                        request_data = sock.recv(1024)
                    except ConnectionResetError as e:
                        request_data = None
                    if not request_data:
                        sock.close()
                        rlist.remove(sock)
                    else:
                        request_data = request_data.decode('utf-8')
                        print(''.join(
                            f'< {line}\n' for line in request_data.splitlines()
                        ))
                        # parse request
                        (request_method, path, request_version) = self.parse_request(request_data)
                        env = self.get_environ(
                            request_data, request_method, path,
                            self.server_name, self.server_port
                        )
                        result = self.application(env, self.start_response)
                        self.finish_response(result, sock)

    @classmethod
    def parse_request(cls, text):
        request_line = text.splitlines()[0]
        request_line = request_line.rstrip('\r\n')
        return request_line.split()

    def get_environ(self, request_data, request_method, path, server_name, server_port):
        env = {}
        # The following code snippet does not follow PEP8 conventions
        # but it's formatted the way it is for demonstration purposes
        # to emphasize the required variables and their values
        #
        # Required WSGI variables
        env['wsgi.version']      = (1, 0)
        env['wsgi.url_scheme']   = 'http'
        env['wsgi.input']        = io.StringIO(request_data)
        env['wsgi.errors']       = sys.stderr
        env['wsgi.multithread']  = False
        env['wsgi.multiprocess'] = False
        env['wsgi.run_once']     = False
        # Required CGI variables
        env['REQUEST_METHOD']    = request_method    # GET
        env['PATH_INFO']         = path              # /hello
        env['SERVER_NAME']       = server_name       # localhost
        env['SERVER_PORT']       = str(server_port)  # 8888
        return env

    def start_response(self, status, response_headers, exc_info=None):
        # Add necessary server headers
        server_headers = [
            ('Date', 'Mon, 15 Jul 2019 5:54:48 GMT'),
            ('Server', 'WSGIServer 0.2'),
        ]
        self.headers_set = [status, response_headers + server_headers]
        # To adhere to WSGI specification the start_response must return
        # a 'write' callable. We simplicity's sake we'll ignore that detail
        # for now.
        # return self.finish_response

    def finish_response(self, result, conn):
        status, response_headers = self.headers_set
        response = f'HTTP/1.1 {status}\r\n'
        for header in response_headers:
            response += '{0}: {1}\r\n'.format(*header)
        response += '\r\n'
        for data in result:
            response += data.decode('utf-8')
        # Print formatted response data a la 'curl -v'
        print(''.join(
            f'> {line}\n' for line in response.splitlines()
        ))
        response_bytes = response.encode()
        conn.sendall(response_bytes)
```
we use following code tell os that give the readable and writable file descriptors when possible:
```Python
readables, writables, exceptions = select.select(rlist, wlist, elist)
```
and then we loop the `readables`, if it is a listen_socket, we accept it. if it is a socket with income data, we read it.

### Summary
Let's benchmark our wsgi server with following web application, also against Gunicorn:
```Python
import time
from flask import Flask

app = Flask(__name__)

@app.route('/hello')
def hello_world():
    time.sleep(0.2)  # simulate that your bussiness logic will take 200 ms
    return 'Hello World'
```
to benchmark, we use `wrk` as following:
```bash
wrk -c 1000 -d 60 -t 8 http://localhost:8888/hello
```

Gunicorn config:
```bash
gunicorn -w 8 flask_hello:app
```

Here is the result for multi-processing version:
```bash
09:00 $ wrk -c 1000 -d 60 -t 8 http://localhost:8888/hello
Running 1m test @ http://localhost:8888/hello
  8 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   202.47ms  144.58ms   1.99s    94.69%
    Req/Sec    92.84     40.03   282.00     66.11%
  44396 requests in 1.00m, 6.35MB read
  Socket errors: connect 0, read 44536, write 437, timeout 214
Requests/sec:    739.07
Transfer/sec:    108.26KB
```

Here is the result for Non-Blocking IO version:
```bash
09:07 $ wrk -c 1000 -d 60 -t 8 http://localhost:8888/hello
Running 1m test @ http://localhost:8888/hello
  8 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   139.15ms   49.91ms   1.91s    77.01%
    Req/Sec   519.52    211.03     1.42k    73.73%
  247992 requests in 1.00m, 35.48MB read
  Socket errors: connect 0, read 38, write 0, timeout 417
Requests/sec:   4128.61
Transfer/sec:    604.78KB
```

Here is the result with Gunicorn multi-processing version
```bash
09:03 $ wrk -c 1000 -d 60 -t 8 http://localhost:8000/hello
Running 1m test @ http://localhost:8000/hello
  8 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    43.78ms   77.65ms   1.97s    97.56%
    Req/Sec   472.75    350.11     2.76k    81.24%
  217149 requests in 1.00m, 35.41MB read
  Socket errors: connect 0, read 78, write 147, timeout 94
Requests/sec:   3613.30
Transfer/sec:    603.39KB
```

here is the result on my Linux book:

|   | Single Process  |  Multi-Process |  Non-Blocking I/O | Gunicorn |
|---|---|---|---|---|
| QPS  |  todo |  739  |  4128 | 3613 |

And found an interesting thing, Non-Blocking IO will use one process but also can support high qps, at the same time, it takes less CPU than Gunicorn pre-fork model.


### Reference
[PEP 333: Python web server gateway interfaces v1.0](https://www.python.org/dev/peps/pep-0333/#implementation-application-notes)  
[Let's build a web server](https://ruslanspivak.com/lsbaws-part2/)  
[async io on linux select/poll/epoll](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/)  
