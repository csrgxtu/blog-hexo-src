---
title: Template Engine Internal
date: 2017-07-09 16:38:28
tags:
---
* How template engine works? the basic principle behind this.
* Implement a template engine
* What about Jinjia2?
<!--more-->

#### Basic principle
##### Why we need a template or template engine?
Most programs contains lots of logic but little bit of textual data, and programming languages like Python, Java are designed to do this kind of job. but some programming tasks requires little logic but lots of textual data processing, for this kind of task, we need a better tool that suite for text-heavy problems -- such a tool named Template(Engine).  
Note: you may wonder that you can still use programming languages like Clang processing text-heavy tasks, like assembling the html source code for the client. i will say that is okay but not efficient, plus your code will be a messy with statements and string literals. don't we programmers like modualize everything and make programming easier afterwards?

##### Template: the big picture
Template is suite for text heavy tasks, especially where web server application need to return HTML page to the client. so basiclly it need to turn following template into a html string:
```html
<html>
  <head><title>Template</title></head>
  <body>
    <h3>Hello {{username}}</h3>
  </body>
</html>
```

```html
<html>
  <head><title>Template</title></head>
  <body>
    <h3>Hello archer</h3>
  </body>
</html>
```

How can we achieve this? there are two basic ways:
* with string substitution
* code execution

If chose string substitution, then we can use Python's format() replace the username with the value we want as follows:
```Python
#!/usr/bin/env python
Template_Page = """
<html>
  <head><title>Template</title></head>
  <body>
    <h3>Hello {{username}}</h3>
  </body>
</html>
"""
html = Template_Page.format(username='string substitution')
```

Here is the output of the variable html:
```bash
<html>
  <head><title>Template</title></head>
  <body>
    <h3>Hello string substitution</h3>
  </body>
</html>
```
If we chose to use code execution, then we need to convert this template html into a corresponding Python function, then we call this function with right parameters.

```Python
#!/usr/bin/env python

# string format
fuck_str = """
def render_function(context):
    c_person = context['username']
    result = []

    result.append('<html>\n<head><title>Template</title></head>\n<body>\n<h3>')
    result.append(str(c_person))
    result.append('</h3>\n</body>\n</html>')

    return ''.join(result)
"""
namespace = {}
exec(fuck_str, namespace) # will put render_function as a function into namespace, like JS's evil eval which will execute a string as a statements
html = namespace.get('render_function')({'username': 'code execution'})
print html
```
Should got this
```html
<html>
  <head><title>Template</title></head>
  <body>
    <h3>Hello code execution</h3>
  </body>
</html>
```
As you can see from upper, code execution tries to take template's username as a Python variable and will use it's value instead of the variable name when assembling the html string.

A template file is like a programming source file, and template engine are like programming interpreter or a compiler. it takes template file in and produce the corresponding html. normally a template contains two parts, Static Part and Dynamic Part, template engine will interpret Dynamic Part with real data, like in upper example will replace username with its real value.

What u just saw is a very simple illustration? there are a lot of things need to take into consideration when you want to totally understand the Template Engine. like how to interpret or support loop controls, what about if else? these are very often used statements in Programming languages like Python.

Some engine just use string substitution, some use code execution totally, some mixed them up, so all engines sits between these two basic techs as following shows:
```bash
code execution                                                   string substitution
<---------------------------------------|------------------------------------------->
moko                       Jinja2 | Django | etc ...           .format(**kwargs)
```


#### Implementation Detail
The goal is turn following template(html mixed string) into real html string as you see in Jinja2. so question is how can we do this conversion? especially how can we interpret the variables, control statements?
```html
<html>
  Hello, {{ username }}!
  <ul>
    {% for job in job_list %}
      <li>{{ job }}</li>
    {% endfor %}
  </ul>
</html>
```

```html
<html>
  Hello, archer!
  <ul>
      <li>engineer</li>
      <li>qa</li>
      <li>pm</li>
  </ul>
</html>
```

In an interpretation model, parsing produces a data structure representing the structure of the template. The rendering phase walks that data structure, assembling the result text based on the instructions it finds. For a real-world example, the Django template engine uses this approach.

In a compilation model, parsing produces some form of directly executable code. The rendering phase executes that code, producing the result. Jinja2 and Mako are two examples of template engines that use the compilation approach.

In this article, we use the compilation model, i.e in parsing phase, we will parse the template from string directly into Python code, i.e Python objects. and in rendering phase, we will execute the code and get the results.

Here is the design: class Templite responsible two things, first, parsing phase, accept template string, parse it and turn it into a render_func object, second, render phase, invoke and execute the render_func object.

In parsing phase, process template line by line, if met normal string literals, use CodeBuilder.add_line append this to the CodeBuilder.code list, especially in render_func's result list. so you should see this:
```Python
CodeBuilder.add_line('result.apend("<html>\nHello ") ')
```
If it is a expression, should append this variable into render_func's result list, as follows:
```Python
CodeBuilder.add_line(str(username))
```
when it finished parsing, CodeBuilder.code should contains all string code of the render_func, if you join them together, you will got following string:
```python
"""
def render_function(context, do_dots):
    c_username = context['username']
    c_job_list = context['job_list']
    result = []
    append_result = result.append
    extend_result = result.extend
    to_str = str
    extend_result(['\n<html>\n  Hello, ', to_str(c_username), '!\n  <ul>\n    '])
    for c_job in c_job_list:
        extend_result(['\n      <li>', to_str(c_job), '</li>\n    '])
    append_result('\n  </ul>\n</html>\n')
    return ''.join(result)
"""
```
then use Python's exec turn this string into a real Python function object.

In render phase, you call render function, actually, it executes the render_func returned in parsing phase, and thus you got the real html.


```Python
class Templite(object):
  """ Template Engine Implmentation """
  def __init__(self, template):
    """ responsible turn template from string into Python code object through CodeBuilder """
    pass

  def render(self, context):
    """ run the code object prepared by __init__ with context data, will return the interpreted template in html format """
    pass


class CodeBuilder(object):
  """ responsible for build a string representation of the render_function, and method that will return the string representation function to a Python code object """
  def __init__(self, indent = 0):
    self.code = []
    self.indent_level = indent

  def add_line(self, line):
    """ add a line into code with right indentation """
    pass

  def get_render_fun(self):
    """ get render function in Python object """
    pass
```

Here is the data flow:
1st, input following template
```html
<html>
  Hello, {{ person.username }}!
  <ul>
    {% for job in job_list %}
      <li>{{ job }}</li>
    {% endfor %}
  </ul>
</html>
```
2nd, Templite.__init__ will accept this template as input, and parse it, if necessary invoke CodeBuilder
3rd, CodeBuiler will receive the following phrase from Templite.__init__
* html string literal
* expressions
* controll tags

append each line into a code object after parse and conversion

append each line into a code object after parse and conversion
4th, here is what code object looks like
```Python
[
'def render_function(context, do_dots):', '\n',
'    ', 'result = []', '\n',
'    ', 'append_result = result.append', '\n',
'    ', 'extend_result = result.extend', '\n',
'    ', 'to_str = str', '\n',
'    ', "extend_result(['\\n<html>\\n  Hello, ', to_str(c_username), '!\\n  <ul>\\n    '])", '\n',
'    ', 'for c_job in c_job_list:', '\n',
'        ', "extend_result(['\\n      <li>', to_str(c_job), '</li>\\n    '])", '\n',
 '    ', "append_result('\\n  </ul>\\n</html>\\n')", '\n',
 '    ', "return ''.join(result)", '\n'
]
```
5th, in render function, get the render function from code object through exec and  stringify
```python
def render_function(context, do_dots):
    c_username = context['username']
    c_job_list = context['job_list']
    result = []
    append_result = result.append
    extend_result = result.extend
    to_str = str
    extend_result(['\n<html>\n  Hello, ', to_str(c_username), '!\n  <ul>\n    '])
    for c_job in c_job_list:
        extend_result(['\n      <li>', to_str(c_job), '</li>\n    '])
    append_result('\n  </ul>\n</html>\n')
    return ''.join(result)
```

6th, run this render function with the context data you provide
```html
<html>
  Hello, archer!
  <ul>
      <li>engineer</li>
      <li>qa</li>
      <li>pm</li>
  </ul>
</html>
```

for detailed implementation, refer [TemplateEngine](https://github.com/csrgxtu/TemplateEngine/blob/500lines/templite.py)
