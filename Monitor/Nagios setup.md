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


