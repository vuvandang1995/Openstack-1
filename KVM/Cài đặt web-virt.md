# Mục đích cài ứng dụng web-virt để control các máy ảo từ xa

## Warnning: hướng dẫn cài đặt này (Install web-virt) chỉ đúng  trên Ubuntu 14.04 các phiên bản khác chưa test!


### Mô hình 

<img src="https://camo.githubusercontent.com/0a2918a589d98b9673e5ec85782976599965bf53/687474703a2f2f692e696d6775722e636f6d2f30444c6a6b71312e706e67">

Cả 2 máy đều sử dụng hệ điều hành là UbuntuServer 14.04.

### Bước 1: Tải các gói cần thiết để cài đặt

``sudo apt-get install git python-pip python-libvirt python-libxml2 novnc supervisor nginx ``

### Bước 2: Cài đặt python requirements và thiết lập môi trường Django

```sh
git clone git://github.com/retspen/webvirtmgr.git
cd webvirtmgr
sudo pip install -r requirements.txt # or python-pip (RedHat, Fedora, CentOS, OpenSuse)
./manage.py syncdb
./manage.py collectstatic
```

Làm theo hướng dẫn 

```sh
You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): yes (Put: yes)
Username (Leave blank to use 'admin'): admin (Put: your username or login)
E-mail address: username@domain.local (Put: your email)
Password: xxxxxx (Put: your password)
Password (again): xxxxxx (Put: confirm password)
Superuser created successfully.
```

### Bước 3: Cài đặt nginx

```sh
mkdir /var/www
cd ..
sudo mv webvirtmgr /var/www/
```
Kiểm tra trong thư mục /var/www/ đã có webvirtmgr chưa. Chưa có thì phải kiểm tra lại

Kết quả oke:

```sh
root@ubuntu:~# cd /var/www/webvirtmgr/
root@ubuntu:/var/www/webvirtmgr# ls -alh
total 168K
drwxr-xr-x 21 www-data www-data 4.0K Oct 26 23:26 .
drwxr-xr-x  3 root     root     4.0K Oct 26 23:18 ..
drwxr-xr-x  6 www-data www-data 4.0K Oct 26 23:14 conf
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 console
drwxr-xr-x  3 www-data www-data 4.0K Oct 26 23:22 create
drwxr-xr-x  4 www-data www-data 4.0K Oct 26 23:14 deploy
-rw-r--r--  1 www-data www-data   85 Oct 26 23:14 dev-requirements.txt
drwxr-xr-x  8 www-data www-data 4.0K Oct 26 23:22 .git
-rw-r--r--  1 www-data www-data  154 Oct 26 23:14 .gitignore
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 hostdetail
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:14 images
drwxr-xr-x  3 www-data www-data 4.0K Oct 26 23:22 instance
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 interfaces
drwxr-xr-x  5 www-data www-data 4.0K Oct 26 23:14 locale
-rwxr-xr-x  1 www-data www-data  253 Oct 26 23:14 manage.py
-rw-r--r--  1 www-data www-data  722 Oct 26 23:14 MANIFEST.in
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 networks
-rw-r--r--  1 www-data www-data 2.5K Oct 26 23:14 README.rst
-rw-r--r--  1 www-data www-data  147 Oct 26 23:14 requirements.txt
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 secrets
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:14 serverlog
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 servers
-rw-r--r--  1 www-data www-data 2.3K Oct 26 23:14 setup.py
drwxr-xr-x  6 www-data www-data 4.0K Oct 26 23:16 static
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 storages
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:14 templates
-rw-r--r--  1 www-data www-data  497 Oct 26 23:14 .travis.yml
-rw-r--r--  1 www-data www-data  580 Oct 26 23:14 Vagrantfile
drwxr-xr-x  2 www-data www-data 4.0K Oct 26 23:22 vrtManager
drwxr-xr-x  5 www-data www-data 4.0K Oct 26 23:22 webvirtmgr
-rw-r--r--  1 www-data www-data  42K Oct 26 23:26 webvirtmgr.sqlite3
```

Thêm file webvirtmgr.conf trong /etc/nginx/conf.d: 

``vi /etc/nginx/conf.d/webvirtmgr.conf``

```sh
server {
    listen 80 default_server;

    server_name $hostname;
    #access_log /var/log/nginx/webvirtmgr_access_log; 

    location /static/ {
        root /var/www/webvirtmgr/webvirtmgr; # or /srv instead of /var
        expires max;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M; # Set higher depending on your needs 
    }
}
```

Vào thư mục /etc/nginx/sites-enabled/default comment lại section Server

``vi /etc/nginx/sites-enabled/default``

```sh
#    server {
#        listen       80 default_server;
#        server_name  localhost;
#        root         /usr/share/nginx/html;
#
#        #charset koi8-r;
#
#        #access_log  /var/log/nginx/host.access.log  main;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        # redirect server error pages to the static page /40x.html
#        #
#        error_page  404              /404.html;
#        location = /40x.html {
#        }
#
#        # redirect server error pages to the static page /50x.html
#        #
#        error_page   500 502 503 504  /50x.html;
#        location = /50x.html {
#        }
#    }
```

Khởi động lại nginx

`` sudo service nginx restart``

 

### Bước 4: Setup supervisor

``sudo vi /etc/insserv/overrides/novnc``

Thêm vào đoạn sau

```sh
#!/bin/sh
### BEGIN INIT INFO
# Provides:          nova-novncproxy
# Required-Start:    $network $local_fs $remote_fs $syslog
# Required-Stop:     $remote_fs
# Default-Start:     
# Default-Stop:      
# Short-Description: Nova NoVNC proxy
# Description:       Nova NoVNC proxy
### END INIT INFO
```

Cấp quyền

``sudo chown -R www-data:www-data /var/www/webvirtmgr``

Thêm file webvirtmgr.conf vào thư mục /etc/supervisor/conf.d:

``vi /etc/supervisor/conf.d/webvirtmgr.conf``

```sh
[program:webvirtmgr]
command=/usr/bin/python /var/www/webvirtmgr/manage.py run_gunicorn -c /var/www/webvirtmgr/conf/gunicorn.conf.py
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr.log
redirect_stderr=true
user=www-data

[program:webvirtmgr-console]
command=/usr/bin/python /var/www/webvirtmgr/console/webvirtmgr-console
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=www-data
```

Restart supervisor 

```sh
sudo service supervisor stop
sudo service supervisor start
```
```sh
cd /var/www/webvirtmgr
sudo git pull
sudo ./manage.py collectstatic
sudo service supervisor restart
```

### Bước 6: 

Tuy nhiên sau khi cài web-virt chúng ta sẽ chưa console được các máy ảo đã cài đặt trước sẵn có thông qua novnc. Thế nên ta phải đi chỉnh sửa file VM.xml trên `máy KVM` như sau:

```sh 
virsh shutdown <name_VM>
virsh edit <name_VM>
.........................
<graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'>
      <listen type='address' address='0.0.0.0'/>
</graphics>
.........................
virsh start <name_VM>
```

### Bước 7: Chỉnh sửa file cấu hình của libvirt trên `KVM` như sau 

- file libvirt.conf
``vi /etc/libvirt/libvirtd.conf``

```sh
listen_tls = 0
listen_tcp = 1
listen_addr = "0.0.0.0"
unix_sock_group = "libvirtd"
unix_sock_ro_perms = "0777"
unix_sock_rw_perms = "0770"
auth_unix_ro = "none"
auth_unix_rw = "none"
auth_tcp = "none"
```

- file libvirt-bin

``vi /etc/default/libvirt-bin``

```sh 
start_libvirtd="yes"
libvirtd_opts="-l -d"
```

- Khởi động lại libvirt

``service libvirt-bin restart``

- Kiểm tra lại 

```sh
ps ax | grep [l]ibvirtd
netstat -pantu | grep libvirtd
virsh -c qemu+tcp://127.0.0.1/system (chỗ này là do cài web-virt trên chính con vkm nên sẽ để địa chỉ local là 127.0.0.1, nếu cài trên mô hình KVM riêng và web-virt riêng sẽ chỉnh địa chỉ ip thành địa chỉ của máy KVM)
```

Kết quả:

<img src="https://camo.githubusercontent.com/aef9dc44abfde392835c04289f8752915527c2eb/687474703a2f2f692e696d6775722e636f6d2f4a366f3964676d2e706e67">

### Bước 8: Cài đặt web-virt

- Trên trình duyệt, tiến hành gõ địa chỉ ip của máy chủ cài đặt Webvirt. Giao diện Webvirt hiện ra. Click vào Add Connection để tiến hành tạo một kết nối đến máy KVM :

<img src=http://i.imgur.com/09koSpE.png>

Lưu ý : tài khoản đăng nhập là tài khoản của máy KVM.

- Click vào đường dẫn kvm-154 vừa tạo, Chọn `Storages/New Storage` để tiến hành tạo thư mục ổ đĩa và thư mục chưa file iso cho KVM :

<img src=http://i.imgur.com/F2GCqF1.png>

Lưu trữ file iso :

<img src=http://i.imgur.com/dpPzW7C.png>

Sau khi tạo được 1 mục chứa các phân vùng đĩa và một mục chứa các file iso . Chúng ta tiến hành tạo Network

- Chuyển sang tab Network :

Có thể cài NAT hoặc Brigde hoăc OVS tùy theo cách cấu hình và nhu cầu sử dụng :


<img src=http://i.imgur.com/2emKN6C.png>

hoặc NAT

<img src=http://i.imgur.com/YuPkihG.png>

- Sau khi cài đặt mạng, tiến hành tạo img cho hệ điều hành muốn cài .Vào `Storages/Add Images`:

<img src=http://i.imgur.com/sHMUVgE.png>

- Tiếp theo chúng ta tiến hành cài đặt . Vào `Instances/New Instance/Custom Instance` :

<img src=http://i.imgur.com/nNhlYYl.png>

- Tiến hành add file iso và connect hệ điều hành cho máy vừa tạo :

<img src=http://i.imgur.com/Zz8NP5q.png>

- Chuyển sang tab Start và Console máy vừa tạo

<img src=http://i.imgur.com/pNTb9dI.png>

<img src=http://i.imgur.com/73wo3xT.png>

- noVNC hiện lên, tiến hành cài hệ điều hành như bình thường.

<img src=http://i.imgur.com/ejuIfPp.png>

