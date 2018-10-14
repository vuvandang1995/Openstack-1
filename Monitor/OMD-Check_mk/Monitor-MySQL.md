# monitor mysql

- [1. Chuẩn bị Plugin](#1)
	- [1.1 Chuẩn bị trên host MySQL](1.1)
	- [1.2 Chuẩn bị trên server OMD](#1.2)
- [2. Cấu hình trên Web UI](#2)
- [3. Kiểm tra](#3)

<a name="1" ></a>
### 1. Chuẩn bị Plugin

<a name="1.1" ></a>
#### 1.1 Chuẩn bị trên host MySQL

Thông tin host MySQL

```
OS: CentOS 7
IP: 192.168.100.140
Hostname: node1
Service: MySQL  `yum install -y mariadb-server`
```

- **Bước 1:** Bind địa chỉ của server Database

Sửa file:`vi /etc/my.cnf.d/server.cnf` (CentOS/RHEL) và thêm ở phần `[mysqld]`

```
...
[mysqld]
bind-address = 0.0.0.0
...
```

- **Bước 2:** Restart lại dịch vụ

```
systemctl restart mariadb
```

hoặc 

```
systemctl restart mysql
```

<a name="createuser" ></a>
- **Bước 3:** Tạo user kiểm tra cho OMD

Ở phần này chúng ta cần tạo một USER có quyền USAGE trên host DB (MySQL hoặc MariaDB).

Đăng nhập vào MySQL

```
mysql -u root -p
```

Thực hiện câu lệnh sau:

```
GRANT usage ON *.* TO 'checker'@'%' IDENTIFIED BY '123';
```

- `checker` : Tên của user dùng cho OMD có thể truy xuất vào DB (Tùy chọn)
- `123` : Mật khẩu của `checker` (Tùy chọn)

Như vậy là ta đã hoàn thành công việc ở host DB.

<a name="1.2" ></a>
#### 1.2 Chuẩn bị trên server OMD

- **Bước 1**: Cài đặt các gói đi kèm trên OMD server

	- Trên server OMD sử dụng OS CentOS

	Chúng ta thêm repo của MariaDB.
	**Lưu ý* không được để có dấu cách ở trước các dòng nếu không os sẽ không thể phân giải được
  
```
vi /etc/yum.repos.d/MariaDB.repo
	

[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
  
	 
	
  Cài gói MariaDB-client và DBI, DBD::mysql

	```
	yum install MariaDB-client perl-DBI perl-DBD-MySQL -y
	```
	
- **Bước 2**: Cài đặt Plugin trên OMD
	- Kiểm tra plugin
	
	**Chú ý**: Thay thế `monitoring` bằng tên site của bạn.
	
	Kiểm tra plugin `check_mysql_health` trong thư mục chứa plugin của OMD. Vì plugin này đã tích hợp sẵn vào OMD.
	
	```
	cd /opt/omd/sites/monitoring/lib/nagios/plugins
	ll check_mysql*
	```
	
	<img src="https://i.imgur.com/MJUpGaU.png">
	
	Nếu chưa có hoặc thiếu, chúng ta tải và thêm vào OMD theo bước sau
	
	```
	wget https://raw.githubusercontent.com/hoangdh/meditech-ghichep-omd/master/tools/check_mysql_health
	cp check_mysql_health /opt/omd/sites/monitoring/lib/nagios/plugins
	```
	
	- Phân quyền cho plugin
	
	```
	cd /opt/omd/sites/monitoring/lib/nagios/plugins
	chmod +x check_mysql_health
	```
	
- **Bước 3:** Chạy thử plugin để biết cách sử dụng hoặc [tham khảo](https://gist.github.com/hoangdh/c5aab8dc05dca9710d3b98a782251352).

	```
	cd /opt/omd/sites/monitoring/lib/nagios/plugins
	./check_mysql_health --hostname 192.168.100.140 --username checker --password 123 --mode connection-time --warning 3 --critical 5
	```
	**Chú ý**: Thay thế user và password của bạn vào câu lệnh nếu có thay đổi thông tin ở bước [tạo user](createuser).
	
	<img src="https://i.imgur.com/HwykARn.png">
	
	Như vậy, ta thấy script chạy khá ổn. Tiếp đến chúng ta sẽ thêm vào `check_mk`.

<a name="2" ></a>
### 2. Cấu hình trên Web UI

Đầu tiên, chúng ta cài agent lên host DB - MySQL và thêm nó vào OMD lên trước sau đó mới đến các bước bên dưới


Tiếp theo tạo host

<img src="https://i.imgur.com/3OPkQFd.png">

Trên Web UI, chúng ta tìm đến **WATO · Configuration** > **Host & Service Parameters** -> **Active check** và chọn **Classical active and passive Monitoring checks**

<img src="https://i.imgur.com/QiaD97O.png">


Bấm vào **Create rule in folder:** để tạo thêm 1 rule mới


Điền thông tin của plugin:

<img src="https://i.imgur.com/TPXTXvi.png" />

<img src="https://i.imgur.com/V5QMHK5.png">

- **Giải thích:**
	- `1`: Mô tả plugin
	- `2`: Tên hiển thị của plugin
	- `3`: Câu lệnh sử dụng plugin. Biến **HOSTADDRESS** được sử dụng để gọi địa chỉ của host mà chúng ta khai báo ở bên trên là IP `192.168.100.140`.
	```
	check_mysql_health --hostname $HOSTADDRESS$ --username checker --password 123 --mode connection-time --warning 3 --critical 5 
	```
	- `4`: Cho phép OMD xử lý, phân tích dữ liệu thu thập được.
	

Lưu lại các thông tin vừa cấu hình và click vào mục `Changes` để active lên, đồng thời vào `Hosts` -> `mysql-01` -> `Services` để `fixed all missing/vanished` hoặc `automatic refresh`

<img src="https://i.imgur.com/0fKebvg.png">


Tiếp theo chúng ta vào **Services > All services** và reschedule dịch vụ

<img src="https://i.imgur.com/3LeAx6A.png">


Như vậy, chúng ta đã giám sát thành công MODE `connection-time` của server MySQL.

<img src="https://i.imgur.com/BvH5gMT.png" />
