# Cho phép haproxy và http đều có thể listen trên port 80


Bind port 80 cho http

- sửa file httpd  (điền địa chỉ IP muốn bind port 80)

`vi /etc/httpd/conf/httpd.conf`
 
<img src="https://i.imgur.com/ILy4yvR.png">


Bind port 80 cho haproxy
```
vi /etc/haproxy/haproxy.cfg


global
        daemon
        maxconn 256

defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
        stats enable
        stats uri /monitor
        stats auth root:meditech2017

listen httpd
    bind 192.168.40.61:80  ( Địa chỉ ip muốn bind port 80)
    balance  roundrobin
    mode  http
    server web1 10.10.10.35:80 check
    server web2 10.10.10.36:80 check
```


