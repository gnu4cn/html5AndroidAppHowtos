# 部署API服务器的注意点

API服务器部署，涉及到Flask、Uwsgi、Supervisor与Nginx的配置。如配置不当，则会导致诸如Supervisor无法管理Uwsgi、Nginx返回404错误等问题。


## Uwsgi的配置文件

```config
[uwsgi]
# pwd
chdir           = /home/peng/ccna60d-apis

#虚拟环境环境路径
virtualenv = %v/.venv

#配合nginx使用
socket = %v/uwsgi.socket

#wsgi文件 run就是flask启动文件去掉后缀名 app是run.py里面的Flask对象 
module          = run:app

#指定工作进程
processes       = 4

#主进程
master          = true

#每个工作进程有2个线程
# enable-threads = true
threads = 2

#保存主进程的进程号
pidfile = %v/uwsgi.pid
```

注意这里去掉了 `daemonize` 配置，即取消uwsgi以守候进程方式允许，否则在下一步中Supervisor无法对其进行管理。

观察Supervisor中uwsgi的日志输出（`$sudo cat /var/log/supervisor/uwsgi-stderr*.log`）:

```log
...
your processes number limit is 30549
your memory page size is 4096 bytes
detected max file descriptor number: 1024
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
...
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
...
```

则是可能要修改 `/etc/sysctl.conf`中的一些参数。

## Supervisor的配置

`/etc/supervisor/conf.d/uwsgi.conf`：

```config
[program:uwsgi]
command=/home/peng/ccna60d-apis/.venv/bin/uwsgi /home/peng/ccna60d-apis/uwsgi.ini
user=peng
directory=/home/peng/ccna60d-apis
autostart=true
autorestart=true
startsecs=5
startretries=3
```

## Nginx配置

Nginx的配置文件（此处是编译安装的 Nginx最新版）：

`/etc/nginx/nginx.conf`:

```config
user  peng;
worker_processes  2;

...
```

__注意：__ 这里是以普通用户运行 Nginx，因为要读取普通用户目录下的 Socket 文件；也可以将Nginx配置为 `daemon off;`后，置于 Supervisor的管理之下。


`/etc/nginx/conf.d/default.conf`

```config
server {
    listen       80;
    server_name  localhost;

    charset utf-8;
    #access_log  /var/log/nginx/host.access.log  main;

    set $HOME /home/peng;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:$HOME/ccna60d-apis/uwsgi.socket;
        uwsgi_param UWSGI_PYHOME $HOME/ccna60d-apis/.venv;
        uwsgi_param UWSGI_CHDIR $HOME/ccna60d-apis;
        uwsgi_param UWSGI_SCRIPT run:app;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }


    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

__注意：__ 这里使用了`set`指令，以及后面的 `uwsgi_params`的使用。
