# 0x10. HTTPS SSL 🔒

```bash
███████ ███████ ██ 
██      ██      ██ 
███████ ███████ ██ 
     ██      ██ ██ 
███████ ███████ ███████
```

## Overview

* DNS
* Web stack debugging

## Environment

<!-- ubuntu -->
* OS: ``ubuntu`` 20.04
* Shell: ``bash``
* ``ssh``
* Style guidelines: ``shellcheck`` 0.3.7
* Editors: ``vim``, ``VS Code``
* ``awk``
* ``dig``
* ``snap`` snapd
* ``certbot``

## install certbot

> show linux distro, install snapd, install certbot

```bash
$ lsb_release -a
$ sudo apt update
$ sudo apt install snapd
$ sudo apt-get upgrade
$ sudo snap install hello-world
$ sudo snap install core; sudo snap refresh core
$ sudo apt-get remove certbot
$ sudo snap install --classic certbot
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

> stop haproxy, install certificate and restart haproxy

```bash
$ sudo service haproxy stop
$ sudo certbot certonly --standalone --preferred-challenges http --http-01-port 80 -d nagu.tech -d www.nagu.tech
$ sudo ls /etc/letsencrypt/live/nagu.tech
```

> copy certificate to haproxy

```bash
$ sudo mkdir -p /etc/haproxy/certs
DOMAIN='xelar.tech' sudo -E bash -c 'cat /etc/letsencrypt/live/$DOMAIN/fullchain.pem /etc/letsencrypt/live/$DOMAIN/privkey.pem > /etc/haproxy/certs/$DOMAIN.pem'
$ sudo chmod -R go-rwx /etc/haproxy/certs
```

> config.cfg in haproxy

_global_

```maxconn 2048```
```tune.ssl.default-dh-param 2048```

_defaults_

```option forwardfor```
```option http-server-close```

_frontend_

```
frontend loadbalancerssl
        bind *:443 ssl crt /etc/haproxy/certs/xelar.tech.pem
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        acl letsencrypt-acl path_beg /.well-known/acme-challenge/
        use_backend letsencrypt-backend if letsencrypt-acl
        default_backend webservers
```

_backend_

```redirect scheme https if !{ ssl_fc }```

only for renewal traffic

```
backend letsencrypt-backend
        server letsencrypt 127.0.0.1:54321
```

> verify if haproxy.cfg contain a valid configuration

```bash
$ sudo haproxy -f /etc/haproxy/haproxy.cfg -c
```

> resart haproxy server

```bash
$ sudo service haproxy restart
```
> test automatical renewal

```bash
$ sudo certbot renew --dry-run
```

## test ssl

```bash
web-01:~$ curl -sI https://www.nagu.tech
HTTP/1.1 200 OK
server: nginx/1.18.0 (Ubuntu)
date: Mon, 24 Jan 2022 06:38:37 GMT
content-type: text/html
content-length: 17
last-modified: Mon, 24 Jan 2022 06:34:06 GMT
etag: "61ee485e-11"
x-served-by: 183576-web-01
accept-ranges: bytes

web-02:~$ curl -sI https://www.nagu.tech
HTTP/1.1 200 OK
server: nginx/1.18.0 (Ubuntu)
date: Mon, 24 Jan 2022 06:38:42 GMT
content-type: text/html
content-length: 17
last-modified: Mon, 24 Jan 2022 06:38:25 GMT
etag: "61ee4961-11"
x-served-by: 183576-web-02
accept-ranges: bytes

183576-lb-01:~$ curl https://www.nagu.tech
Hello World!
```

## redirection 301 from http to https

> show header, follow redirect

```bash
183576@ubuntu-lb-01:~$ curl -sIL http://www.nagu.tech
HTTP/1.1 301 Moved Permanently
content-length: 0
location: https://www.nagu.tech/
connection: close

HTTP/1.1 200 OK
server: nginx/1.18.0 (Ubuntu)
date: Mon, 24 Jan 2022 17:46:49 GMT
content-type: text/html
content-length: 17
last-modified: Mon, 24 Jan 2022 06:34:06 GMT
etag: "61ee485e-11"
x-served-by: 3284-web-01
accept-ranges: bytes
```

# Author
```Chikaodiri Agu```
