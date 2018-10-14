# Cài đặt HA Proxy

- Cài gói haproxy

` yum install -y haproxy`


- File cấu hình haproxy

```
vi /etc/haproxy/haproxy.cfg



  #---------------------------------------------------------------------
  # Example configuration for a possible web application.  See the
  # full configuration options online.
  #
  #   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
  #
  #---------------------------------------------------------------------

  #---------------------------------------------------------------------
  # Global settings
  #---------------------------------------------------------------------
  global
      # to have these messages end up in /var/log/haproxy.log you will
      # need to:
      #
      # 1) configure syslog to accept network log events.  This is done
      #    by adding the '-r' option to the SYSLOGD_OPTIONS in
      #    /etc/sysconfig/syslog
      #
      # 2) configure local2 events to go to the /var/log/haproxy.log
      #   file. A line like the following can be added to
      #   /etc/sysconfig/syslog
      #
      #    local2.*                       /var/log/haproxy.log
      #
      log         127.0.0.1 local2

      chroot      /var/lib/haproxy
      pidfile     /var/run/haproxy.pid
      maxconn     4000
      user        haproxy
      group       haproxy
      daemon

      # turn on stats unix socket
      stats socket /var/lib/haproxy/stats

  #---------------------------------------------------------------------
  # common defaults that all the 'listen' and 'backend' sections will
  # use if not designated in their block
  #---------------------------------------------------------------------
  defaults
      mode                    tcp
      log                     global
      option                  tcplog
      option                  dontlognull
      option http-server-close
      option forwardfor       except 127.0.0.0/8
      option                  redispatch
      retries                 3
      timeout http-request    10s
      timeout queue           1m
      timeout connect         10s
      timeout client          1m
      timeout server          1m
      timeout http-keep-alive 10s
      timeout check           10s
      maxconn                 3000

  #---------------------------------------------------------------------
  # main frontend which proxys to the backends
  #---------------------------------------------------------------------
  frontend  main *:80
      acl url_static       path_beg       -i /static /images /javascript /stylesheets
      acl url_static       path_end       -i .jpg .gif .png .css .js

      use_backend static          if url_static
      default_backend             app

  #---------------------------------------------------------------------
  # static backend for serving up images, stylesheets and such
  #---------------------------------------------------------------------
  backend static
      balance     roundrobin
      server      static 127.0.0.1:4331 check

  #---------------------------------------------------------------------
  # round robin balancing between the various backends
  #---------------------------------------------------------------------
  backend app
      balance     roundrobin
      server  app1 10.10.10.10:80 check
      server  app2 10.10.10.127:80 check