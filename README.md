shadowsocks manyuser branch
===========
Which people need this branch
------------------
1.share shadowsocks server

2.create multi server by shadowsocks

3.manage server (transfer / account)

Change
-------
从efd106dfb5这个提交开始能够独立于前端运行了，原因在于考虑到udp的话可靠性不好。在数据库中修改passwd，switch，enable，流量都会立即生效(比如改密码不需要再通过Managesocket去手动stop某个服务了)。不过内部实现还是通过数据库线程发送一个udp包来触发的，因为数据库这个操作改成异步的会灰常灰常麻烦，所以千万不要在iptabls里面把udp全给堵死了。<del>我的线上环境变成了瘟都死所以这个没有时间和环境测试，对运行过程中出现服务崩溃，服务器冒烟不负任何责任。</del>发现有问题的话给我个提交？诶哟好累英语太拙计就写中文好了。

Install
-------
install MySQL 5.x.x

`pip install cymysql`

create a database named `shadowsocks`

import `shadowsocks.sql` into `shadowsocks`

edit Config.py
Example:

	#Config
	MYSQL_HOST = 'mdss.mengsky.net'
	MYSQL_PORT = 3306
	MYSQL_USER = 'ss'
	MYSQL_PASS = 'ss'
	MYSQL_DB = 'shadowsocks'

	MANAGE_PASS = 'ss233333333'
	#if you want manage in other server you should set this value to global ip
	MANAGE_BIND_IP = '127.0.0.1'
	#make sure this port is idle
	MANAGE_PORT = 23333

TestRun `cd shadowsocks` ` python server.py`

if no exception server will startup. you will see such like
Example:

	db start server at port [%s] pass [%s]

Database user table column
------------------
`passwd` server pass

`port` server port

`t` last keepalive time

`u` upload transfer

`d` download transer

`transfer_enable` if u + d > transfer_enable this server will be stop (db_transfer.py del_server_out_of_bound_safe)

Manage socket
------------------
Manage server work in UDP at `MANAGE_BIND_IP` `MANAGE_PORT`

use `MANAGE_PASS:port:passwd:0` to del a server at port `port`

use `MANAGE_PASS:port:passwd:1` to run a server at port `port` password is `passwd`

Python Eg:

	udpsock.sendto('MANAGE_PASS:65535:123456:1', (MANAGE_BIND_IP, MANAGE_PORT))
	
PHP Eg:

	$sock = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
	$msg = 'MANAGE_PASS:65535:123456:1';
	$len = strlen($msg);
	socket_sendto($sock, $msg, $len, 0, MANAGE_BIND_IP, MANAGE_PORT);
	socket_close($sock);

NOTICE
------------------
If error such like `2014-09-18 09:02:37 ERROR    [Errno 24] Too many open files`

edit /etc/security/limits.conf

Add:

	*                soft    nofile          8192
	*                hard    nofile          65535


add `ulimit -n 8192` in your startup script before runing

shadowsocks
===========

[![PyPI version]][PyPI]
[![Build Status]][Travis CI]

A fast tunnel proxy that helps you bypass firewalls.

Features:
- TCP & UDP support
- User management API
- TCP Fast Open
- Workers and graceful restart
- Destination IP blacklist

Server
------

### Install

Debian / Ubuntu:

    apt-get install python-pip
    pip install git+https://github.com/shadowsocks/shadowsocks.git@master

CentOS:

    yum install python-setuptools && easy_install pip
    pip install git+https://github.com/shadowsocks/shadowsocks.git@master

For CentOS 7, if you need AEAD ciphers, you need install libsodium
```
dnf install libsodium python34-pip
pip3 install  git+https://github.com/shadowsocks/shadowsocks.git@master
```
Linux distributions with [snap](http://snapcraft.io/):

    snap install shadowsocks

Windows:

See [Install Shadowsocks Server on Windows](https://github.com/shadowsocks/shadowsocks/wiki/Install-Shadowsocks-Server-on-Windows).

### Usage

    ssserver -p 443 -k password -m aes-256-cfb

To run in the background:

    sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start

To stop:

    sudo ssserver -d stop

To check the log:

    sudo less /var/log/shadowsocks.log

Check all the options via `-h`. You can also use a [Configuration] file
instead.

If you installed the [snap](http://snapcraft.io/) package, you have to prefix the commands with `shadowsocks.`,
like this:

    shadowsocks.ssserver -p 443 -k password -m aes-256-cfb
    
### Usage with Config File

[Create configuration file and run](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)

To start:

    ssserver -c /etc/shadowsocks.json


Documentation
-------------

You can find all the documentation in the [Wiki](https://github.com/shadowsocks/shadowsocks/wiki).

License
-------

Apache License







[Build Status]:      https://img.shields.io/travis/shadowsocks/shadowsocks/master.svg?style=flat
[PyPI]:              https://pypi.python.org/pypi/shadowsocks
[PyPI version]:      https://img.shields.io/pypi/v/shadowsocks.svg?style=flat
[Travis CI]:         https://travis-ci.org/shadowsocks/shadowsocks

