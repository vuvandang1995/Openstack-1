## 1. Mô hình

<img src="https://i.imgur.com/2XqhdTK.png">

Môi trường lab: KVM

HĐH: CentOS 7.3 - 17.08

**Lưu ý:** Nếu bạn sử dụng hđh CentOS 7.3-1611 thì sẽ phải update trước khi cài keepalived

<a name="3"></a>
## 2. Cấu hình keepalived

- Cài đặt package

`yum install keepalived -y`

Để cho phép HAProxy ràng buộc vào các địa chỉ IP được chia sẻ, chúng ta thêm dòng sau vào /etc/sysctl.conf :

`vi /etc/sysctl.conf`

thêm vào file dòng sau:

`net.ipv4.ip_nonlocal_bind=1`

kiểm tra

`sysctl -p`

- Cấu hình keepalived

**Trên server haproxy1**

`vi /etc/keepalived/keepalived.conf`

``` sh
vrrp_script chk_haproxy {           
        script "killall -0 haproxy"     
        interval 2                      
        weight 2                        
}

vrrp_instance VI_1 {
        interface eth0
        state MASTER
        virtual_router_id 51
        priority 101                    
        virtual_ipaddress {
            192.168.40.70       
        }
        track_script {
            chk_haproxy
        }
}
```

**Trên server haproxy2**

``` sh
vrrp_script chk_haproxy {       
        script "killall -0 haproxy"     
        interval 2                      
        weight 2                        
}

vrrp_instance VI_1 {
        interface eth0
        state BACKUP
        virtual_router_id 51
        priority 100                    
        virtual_ipaddress {
            192.168.40.70             
        }
        track_script {
            chk_haproxy
        }
}
```

Note: Thử chạy `killall -0 haproxy` nếu server báo `command not found`, bạn cần cài gói `psmisc`.

- Sau khi cấu hình xong, start dịch vụ và cho phép khởi động cùng hệ thống

``` sh
systemctl start keepalived
systemctl enable keepalived
```

- Kiểm tra xem trên node MASTER đã có virtual ip chưa

<img src="https://i.imgur.com/s5fnlq9.png">

<a name="2"></a>
## 3. Cấu hình haproxy

**Trên server haproxy1**

- Tải và cài đặt package

`yum install haproxy -y`

- Cấu hình haproxy

`vi /etc/haproxy/haproxy.cfg`

``` sh
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
        stats auth root:123456

listen httpd
    bind 192.168.40.70:80
    balance  roundrobin
    mode  http
    server web1 192.168.40.61:80 check
    server web2 192.168.40.62
```

Trong đó 192.168.40.70 là ip VIP , web1 là hostname, 192.168.100.35:80 là ip của server chạy web server. Ở đây mình đã dựng sẵn 2 web-server đơn giản với apache.

Tiến hành tương tự với `haproxy2`.

- Sau khi cài đặt, tiến hành start và cho phép dịch vụ khởi động cùng hệ thống.

``` sh
systemctl start haproxy
systemctl enable haproxy
```

- Mở port 80 để người dùng có thể truy cập

``` sh
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
```

- Truy cập vào địa chỉ của haproxy và kiểm tra

<img src="https://i.imgur.com/PdoM45c.png">

<img src="https://i.imgur.com/XPbo5LD.png">


Như vậy haproxy đã hoạt động, vì ta sử dụng roundrobin nên hai các request sẽ được direct lần lượt qua hai server.

Tiến hành truy cập vào địa chỉ VIP để kiểm tra:

<img src="https://i.imgur.com/9669alP.png">

- Tiến hành tắt dịch vụ haproxy tại server `haproxy1` và kiểm tra tại server `haproxy2`

<img src="https://i.imgur.com/OX7vBdK.png">

- Như vậy VIP đã ngay lập tức được chuyển qua server BACKUP
