# cài đặt Nagios

Cài đặt Apache, httpd
```
[root@www ~]# yum -y install httpd

# Xóa welcome page
[root@www ~]# rm -f /etc/httpd/conf.d/welcome.conf
```
- Cấu hình httpd, đổi tên server
```
[root@www ~]# vi /etc/httpd/conf/httpd.conf
# line 86: change to admin's email address
ServerAdmin root@192.168.40.61
# line 95: change to your server's name
ServerName 192.168.40.61:80
# line 151: change
AllowOverride All
# line 164: add file name that it can access only with directory's name
DirectoryIndex index.html index.cgi index.php
```

- Restart dịch vụ httpd
```
[root@www ~]# systemctl start httpd 
[root@www ~]# systemctl enable httpd 
```
- Mở firewalld cho dịch vụ httpd
```
[root@www ~]# firewall-cmd --add-service=http --permanent 
success
[root@www ~]# firewall-cmd --reload 
success
```
- Tạo 1 HTML test page để client có thể truy cập đến 
```
[root@www ~]# vi /var/www/html/index.html

 <html>
<body>
<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
Test Page
</div>
</body>
</html>
```

<img src="https://i.imgur.com/Ahg6pBk.png">

-------------

- Cài đặt php
```
[root@www ~]# yum -y install php php-mbstring php-pear
[root@www ~]# vi /etc/php.ini

date.timezone = "Asia/Tokyo"

[root@www ~]# systemctl restart httpd 
```
- Tạo 1 PHP test page cho client truy cập
```
[root@www ~]# vi /var/www/html/index.php
 <html>
<body>
<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
<?php
   print Date("Y/m/d");
?>
</div>
</body>
</html>
```

<img src="https://i.imgur.com/G6bEWtU.png">

--------------

- Cài đặt Nagios server

Trước tiên cài đặt epel yum install epel-release -y

Cài các plug-in :
```
yum -y install nagios nagios-plugins-ping nagios-plugins-disk nagios-plugins-users nagios-plugins-procs nagios-plugins-load nagios-plugins-swap nagios-plugins-ssh nagios-plugins-http
```

- Cấu hình Nagios
```
[root@dlp ~]# vi /etc/httpd/conf.d/nagios.conf
# line 24-26, change settings to set access permissionlike follows ( set for line 54-56, too )
# Require all granted
# Require local
Require host 127.0.0.1 192.168.40.62/24

# Thêm nagios admin user và đặt password
[root@dlp ~]# htpasswd /etc/nagios/passwd nagiosadmin 
New password:     
Re-type new password:
Adding password for user nagiosadmin


[root@dlp ~]# systemctl start nagios 
[root@dlp ~]# systemctl enable nagios 
[root@dlp ~]# systemctl restart httpd 
```
- Mở port http và https trên firewall
```
[root@dlp ~]# firewall-cmd --add-service={http,https} --permanent 
success
[root@dlp ~]# firewall-cmd --reload 
success
```
- Truy cập web với user nagiosadmin và nhập password

<img src="https://i.imgur.com/WBykj4d.png">


<img src="https://i.imgur.com/vbHQ6S5.png">

--------------
Đặt ngưỡng 

- đặt ngưỡng cho việc sử dụng disk
```
[root@dlp ~]# vi /etc/nagios/objects/localhost.cfg
# Define a service to check the disk space of the root partition
# on the local machine.  Warning if > 20% free, critical if
# > 10% free space on partition.

# the thresholds are set as Warning if > 20% free, critical if 10% > 10% free
# change these values if you'd like to change them
define service{
        use                       local-service
        host_name                 localhost
        service_description       Root Partition
        check_command             check_local_disk!20%!10%!/
        }

[root@dlp ~]# systemctl restart nagios 
```

- Trường hợp add thêm plugin mới thì làm như sau (*ví dụ thêm plugin check_ntp_time)
```
# Hiển thị lựa chọn cho 1 plugin
[root@dlp ~]# /usr/lib64/nagios/plugins/check_ntp_time -h 
...
...
 -w, --warning=THRESHOLD
    Offset to result in warning status (seconds)
 -c, --critical=THRESHOLD
...
...

# Tạo các command cho 1 plugin với các ngưỡng được cài đặt 
[root@dlp ~]# vi /etc/nagios/objects/commands.cfg
# add follows to the end
define command{
        command_name    check_ntp_time
        command_line    $USER1$/check_ntp_time -H $ARG1$ -w $ARG2$ -c $ARG3$
        }

# Tạo các service tương ứng với các ngưỡng
[root@dlp ~]# vi /etc/nagios/objects/localhost.cfg
# add follows to the end ( Warning if it has 1 sec time difference, Critical if it has 2 sec )
define service{
        use                             local-service
        host_name                       localhost
        service_description             NTP_TIME
        check_command                   check_ntp_time!ntp1.jst.mfeed.ad.jp!1!2
        notifications_enabled           1
        }

[root@dlp ~]# systemctl restart nagios 
```

--------------

- Thêm một số tính năng monitor khác:
```
yum install -y nagios-plugins-all nagios-plugins-apt nagios-plugins-bdii nagios-plugins-bonding nagios-plugins-breeze nagios-plugins-ups nagios-plugins-users nagios-plugins-wave
```

- Thêm plugin check_NTP để theo dõi thời gian chênh lệch giữa ntp server và trong hệ thống
```
[root@dlp ~]# yum -y install nagios-plugins-ntp
[root@dlp ~]# vi /etc/nagios/objects/commands.cfg
# add follows to the end
define command{
        command_name    check_ntp_time
        command_line    $USER1$/check_ntp_time -H $ARG1$ -w $ARG2$ -c $ARG3$
        }

[root@dlp ~]# vi /etc/nagios/objects/localhost.cfg
# add follows to the end ( Warning if it has 1 sec time difference, Critical if it has 2 sec )
define service{
        use                             local-service
        host_name                       localhost
        service_description             NTP_TIME
        check_command                   check_ntp_time!ntp1.jst.mfeed.ad.jp!1!2
        notifications_enabled           1
        }

[root@dlp ~]# systemctl restart nagios 
```




