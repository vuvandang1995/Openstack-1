
### Các bước thực hiện như sau:

- [1. Chuẩn bị Plugin](#1)
	- [1.1 Chuẩn bị trên host RabbitMQ](1.1)
	- [1.2 Chuẩn bị trên server OMD](#1.2)
- [2. Cấu hình trên Web UI](#2)
- [3. Kiểm tra](#3)

<a name="1" ></a>
### 1. Chuẩn bị Plugin

<a name="1.1" ></a>
#### 1.1 Chuẩn bị trên host RabbitMQ

Thông tin host RabbitMQ

```
OS: CentOS 7
IP: 192.168.40.64
Hostname: node1
Service: RabbitMQ
```
Cài rabbitmq trên rabbitmq-server: 
```
yum install epel-release net-tools
yum install -y rabbitmq-server
```
Start rabbitmq: `systemctl start rebbitmq-server`

Enable `rabbitmq_management`:  
```
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
```

Ta tạo một user `mon` với password à `1` có chức năng `monitoring` trên RabbitMQ:

```
rabbitmqctl add_user mon 1
rabbitmqctl set_user_tags mon monitoring
rabbitmqctl set_permissions -p / mon ".*" ".*" ".*" 
```
Kiểm tra user rabbitmq: 
```
rabbitmqctl list-users

[root@node1 ~]# rabbitmqctl list_users
Listing users ...
guest   [administrator]
mon     [monitoring]
...done.

```

Kiểm tra dịch vụ rabbitq có đang mở trên port 15672 không: 
```
[root@node1 ~]# netstat -autun | grep 15672
tcp        0      0 0.0.0.0:15672           0.0.0.0:*               LISTEN

```



<a name="1.2" ></a>
#### 1.2 Chuẩn bị trên server OMD

- **Bước 1**: Cài đặt perl và gói `Monitoring::Plugins` trên OMD server


	- Trên server OMD sử dụng OS CentOS

	```sh
	yum install -y epel-release
	yum install -y perl-Monitoring-Plugin perl-Config-Tiny perl-JSON* perl-Math-Calc-Units
	```
	
- **Bước 2**: Cài đặt Plugin trên OMD
	- Tải git : 
	```
	yum install epel-release -y
	yum install git -y
	```
	
	- Tải plugin
	
	**Chú ý**: Thay thế `hanoi` trong đường dẫn phía dưới bằng tên site của bạn.
	
	```
	git clone https://github.com/nagios-plugins-rabbitmq/nagios-plugins-rabbitmq.git
	cp nagios-plugins-rabbitmq/scripts/* /opt/omd/sites/hanoi/lib/nagios/plugins
	```
	
	- Phân quyền cho plugin
	
	```
	cd /opt/omd/sites/hanoi/lib/nagios/plugins
	chmod +x check_rabbitmq_*
	```

- **Bước 3:** Thêm thông tin host RabbitMQ trên OMD server

	Chúng ta thêm thông tin của RabbitMQ server trên file host của OMD bằng cách sửa file `/etc/hosts`.
	
	```
	vi /etc/hosts
	```

	```
	...
	192.168.100.198 node1
	```
	
	**Chú ý**: Thông tin `node1` phải trùng với hostname mà server RabbitMQ đang sử dụng.
	
- **Bước 4:** Chạy thử plugin để biết cách sử dụng hoặc [tham khảo](https://gist.github.com/hoangdh/c86dc9d081882ac116322b45399f0442)

	```
	cd /opt/omd/sites/monitoring/lib/nagios/plugins
	./check_rabbitmq_aliveness -H node1 -u mon -p 1
	```
	
	<img src="https://i.imgur.com/DTTdiWB.png">
	
	Như vậy, ta thấy script chạy khá ổn. Tiếp đến chúng ta sẽ thêm vào `check_mk`.

<a name="2" ></a>
### 2. Cấu hình trên Web UI

Đầu tiên, chúng ta cài agent lên host RabbitMQ và thêm nó vào OMD theo [hướng dẫn](2.Install-agent.md#2-cài-đặt-agent-trên-host-giám-sát).

**Lưu ý**: Phần IP của host chúng ta điền `node1` vì đã khai báo ở phần trên trong file `hosts` như hình.

<img src="../images/25-rb-ah1.png" />

Trên Web UI, chúng ta tìm đến **WATO · Configuration** > **Host & Service Parameters** và chọn **Classical active and passive Monitoring checks**

<img src="../images/25-rb-ah2.png" />

<img src="../images/25-rb-ah3.png" />

Bấm vào **Create rule in folder:** để tạo thêm 1 rule mới

<img src="../images/25-rb-ah4.png" />

Điền thông tin của plugin:

<img src="../images/25-rb-ah5.png" />

- **Giải thích:**
	- `1`: Mô tả plugin
	- `2`: Tên hiển thị của plugin
	- `3`: Câu lệnh sử dụng plugin. Biến **HOSTADDRESS** được sử dụng để gọi địa chỉ của host mà chúng ta khai báo ở bên trên là `node1`.
	```
	check_rabbitmq_aliveness -H **HOSTADDRESS** -u mon -p 1 --vhost / 
	```
	- `4`: Cho phép OMD xử lý, phân tích dữ liệu thu thập được.
	
Tiếp theo, chúng ta kéo xuống bên dưới và chọn host `rabbitmq-01` vừa thêm:
	
<img src="../images/25-rb-ah6-2.png" />

Lưu lại các thông tin vừa cấu hình:

<img src="../images/25-rb-ah7.png" />

<img src="../images/25-rb-ah8.png" />

Kiểm tra thông tin, chúng ta chọn **Services > All services** trên tab **View** và chọn force check

<img src="../images/25-rb-ah9.png" />

Như vậy, chúng ta đã giám sát thành công trạng thái Aliveness của server RabbitMQ.

<img src="../images/25-rb-ah10.png" />

Để check các trạng thái khác của RabbitMQ, chúng ta thao tác như trên và thay thế câu lệnh ở bài viết [này](https://gist.github.com/hoangdh/c86dc9d081882ac116322b45399f0442).
