---
title: 使用Coverage.py统计uWSGI+Flask程序的代码覆盖率
date: 2022-06-15
tags:
- uwsgi
- uwsgi2
- python
- coverage
- 覆盖率
- 代码覆盖率
- Flask
- coverage.py
- atexit
categories:
- Tool
image: "cover.png"
url: "uwsgi-flask-coverage"
---

>`Coverage.py`是一款常用的`Python`程序的覆盖率统计工具，他的基础用法为
>
>```
>coverage run main.py
>```
>
>但生产环境中的`Python`程序很多都不会直接运行`python`文件，而是通过`uWSGI`等技术实现服务托管，需要使用`uWSGI`的配置文件启动。这样就无法直接使用`coverage run`了，本文提供一种通过改造`Flask`入口文件来实现`uWSGI + Flask`代码覆盖率采集的方式。



## 实现原理

`Coverage.py`的用法主要有两种，分别为

- 命令行调用

- API调用

其中API调用的方式需要修改待测程序的源码，较少用到，但是在当前案例中是可行的选择。

`Coverage.py`提供了`Coverage`类，以及成员方法`start`, `stop`等，理论上我们只需要在`Flask`启动前以及停止后插入对应的代码即可。启动前，可以直接写，而停止后则需要用特定的信号捕获来实现了，本文使用`atexit`



## demo准备

- 安装 `uWSGI`

```bash
brew install uwsgi # mac或者linuxbrew用户适用
```



- 安装 `coverage`

```
pip install coverage
```



- `Flsak`代码`run.py`

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def index():
    return '/'


@app.route('/hello')
def hello():
    return 'hello'


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)

```



- `uWSGI`配置文件`run.ini`

```ini
[uwsgi]
http=0.0.0.0:5001
chdir=/Developer/PycharmProjects/flask-demo
wsgi-file=run.py
callable=app
processes=4
threads=2
```



- 运行一下试试

```
yanbo92@yanbo92mbp13 flask-demo % uwsgi --ini run.ini               
[uWSGI] getting INI configuration from run.ini
*** Starting uWSGI 2.0.20 (64bit) on [Wed Jun 15 23:00:00 2022] ***
compiled with version: Apple LLVM 13.1.6 (clang-1316.0.21.2.3) on 15 June 2022 12:01:01
os: Darwin-21.2.0 Darwin Kernel Version 21.2.0: Sun Nov 28 20:28:54 PST 2021; root:xnu-8019.61.5~1/RELEASE_X86_64
nodename: yanbo92mbp13.lan
machine: x86_64
clock source: unix
detected number of CPU cores: 8
current working directory: /Developer/PycharmProjects/flask-demo
detected binary path: /Library/Frameworks/Python.framework/Versions/3.9/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
chdir() to /Developer/PycharmProjects/flask-demo
*** WARNING: you are running uWSGI without its master process manager ***
your processes number limit is 1392
your memory page size is 4096 bytes
detected max file descriptor number: 10240
lock engine: OSX spinlocks
thunder lock: disabled (you can enable it with --thunder-lock)
uWSGI http bound on 0.0.0.0:5001 fd 4
spawned uWSGI http 1 (pid: 29244)
uwsgi socket 0 bound to TCP address 127.0.0.1:63603 (port auto-assigned) fd 3
Python version: 3.9.12 (v3.9.12:b28265d7e6, Mar 23 2022, 18:17:11)  [Clang 6.0 (clang-600.0.57)]
Python main interpreter initialized at 0x7fde17104aa0
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 333312 bytes (325 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x7fde17104aa0 pid: 29243 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI worker 1 (pid: 29243, cores: 2)
spawned uWSGI worker 2 (pid: 29247, cores: 2)
spawned uWSGI worker 3 (pid: 29248, cores: 2)
spawned uWSGI worker 4 (pid: 29249, cores: 2)

```



##  Coverage代码插入

```python
...
def collect_coverage():
    # 捕获中断信号之后执行的函数
    print("cov stop")
    cov.stop()
    cov.combine()
    cov.xml_report(outfile='report.xml')
    cov.html_report(directory='report')

# 注册到atexit
atexit.register(collect_coverage)

# Flask启动之前开始采集
print("cov start")
cov = Coverage()
cov.start()

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```



## 简单验证一下

- 重新运行`uWSGI`，可见输出中包含`cov start`

```
...
mapped 333312 bytes (325 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
cov start
WSGI app 0 (mountpoint='') ready in 0 seconds on interpreter 0x7f85e5904750 pid: 30690 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI worker 1 (pid: 30690, cores: 2)
...
```



- 调用一个接口，模拟测试过程

```
yanbo92@yanbo92mbp13 flask-demo % curl 0.0.0.0:5001/hello
hello%                              
```

- `Ctrl + C`结束程序

```
[pid: 31325|app: 0|req: 1/1] 127.0.0.1 () {28 vars in 305 bytes} [Wed Jun 15 23:10:45 2022] GET /hello => generated 5 bytes in 7 msecs (HTTP/1.1 200) 2 headers in 78 bytes (1 switches on core 0)
^Ccov stop
cov stop
cov stop
cov stop
```

- 检查当前目录，已产出报告

```
yanbo92@yanbo92mbp13 flask-demo % ls
report          report.xml      run.ini         run.py
```





## 相关文章

[使用Coverage分析WSGI项目的代码覆盖率](https://segmentfault.com/a/1190000003806169)

[uwsgi flask gevent 测试代码覆盖率（coverage）](https://www.cnblogs.com/daryl-blog/p/11369563.html)

[coverage api](https://coverage.readthedocs.io/en/latest/api_coverage.html)
