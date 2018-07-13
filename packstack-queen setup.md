# Phần này hướng dẫn cài openstack trên packstack-queen
## 1. Các bước chuẩn bị
### 1.1. Giới thiệu

- Lưu ý: Trong tài liệu này chỉ thực hiện cài đặt OpenStack.
- Packstack là một công cụ cài đặt OpenStack nhanh chóng.
- Packstack được phát triển bởi redhat
- Chỉ hỗ trợ các distro: RHEL, Centos
- Tự động hóa các bước cài đặt và lựa chọn thành phần cài đặt.
- Nhanh chóng dựng được môi trường OpenStack để sử dụng làm PoC nội bộ, demo khách hàng, test tính năng.
- Nhược điểm 1 : Đóng kín các bước cài đối với người mới.
- Nhược điểm 2: Khó bug các lỗi khi cài vì đã được đóng gói cùng với các tool cài đặt tự động (puppet)


### 1.2. Môi trường thực hiện 

- Distro: CentOS 7.x
- OpenStack Newton
- NIC1 - eth0: Là dải mạng mà các máy ảo sẽ giao tiếp với bên ngoài. Dải mạng này sử dụng chế độ bridge hoặc NAT của VMware Workstation
- NIC2 - eth1: là dải mạng sử dụng cho các traffic MGNT + API + DATA VM. Dải mạng này sử dụng chế độ hostonly trong VMware Workstation


### 1.3. Mô hình

<img src="https://i.imgur.com/8qTHAh0.png">

### 1.4. IP Planning

<img src="https://i.imgur.com/4UYKYHk.png">

## 2. Các bước cài đặt
### 2.1. Các bước chuẩn bị trên trên Controller

- Thiết lập hostname

	```
	hostnamectl set-hostname controller
	```

- Thiết lập IP 

	```
  echo "Setup IP  eth0"
  nmcli c modify eth0 ipv4.addresses 192.168.40.70/24
  nmcli c modify eth0 ipv4.gateway 192.168.40.1
  nmcli c modify eth0 ipv4.dns 8.8.8.8
  nmcli c modify eth0 ipv4.method manual
  nmcli con mod eth0 connection.autoconnect yes
  
  echo "Setup IP  eth1"
  nmcli c modify eth1 ipv4.addresses 10.10.10.70/24
  nmcli c modify eth1 ipv4.method manual
  nmcli con mod eth1 connection.autoconnect yes

  sudo systemctl disable firewalld
  sudo systemctl stop firewalld
  sudo systemctl disable NetworkManager
  sudo systemctl stop NetworkManager
  sudo systemctl enable network
  sudo systemctl start network

  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
  ```
- Khai báo repos cho OpenStack Newton

```
	yum install centos-release-openstack-queens.x86_64 -y

	sudo yum install -y wget crudini fping   (vi /etc/yum.repos.d/CentOS-QEMU-EV.repo sửa lại đường dẫn repos 'baseurlhttp://mirror.centos.org/centos-7/7/virt/x86_64/kvm-common/' nếu repos báo lỗi sai)
	yum install -y openstack-packstack
	init 6

```

### 2.2. Các bước chuẩn bị trên trên Compute1

- Thiết lập hostname

    ```
    hostnamectl set-hostname compute1
    ```

- Thiết lập IP 

  ```
	echo "Setup IP  eth0"
	nmcli c modify eth0 ipv4.addresses 192.168.40.69/24
	nmcli c modify eth0 ipv4.gateway 192.168.40.1
	nmcli c modify eth0 ipv4.dns 8.8.8.8
	nmcli c modify eth0 ipv4.method manual
	nmcli con mod eth0 connection.autoconnect yes

	echo "Setup IP  eth1"
	nmcli c modify eth1 ipv4.addresses 10.10.10.69/24
	nmcli c modify eth1 ipv4.method manual
	nmcli con mod eth1 connection.autoconnect yes

	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

  ```
 
- Khai báo repos cho OpenStack Newton

```
    yum install centos-release-openstack-queens.x86_64 -y
    yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

### 2.3. Các bước chuẩn bị trên trên Compute2


- Thiết lập hostname

    ```
    hostnamectl set-hostname compute2
    ```

- Thiết lập IP 

  ```
	echo "Setup IP  eth0"
	nmcli c modify eth0 ipv4.addresses 192.168.40.68/24
	nmcli c modify eth0 ipv4.gateway 192.168.40.1
	nmcli c modify eth0 ipv4.dns 8.8.8.8
	nmcli c modify eth0 ipv4.method manual
	nmcli con mod eth0 connection.autoconnect yes

	echo "Setup IP  eth1"
	nmcli c modify eth1 ipv4.addresses 10.10.10.68/24
	nmcli c modify eth1 ipv4.method manual
	nmcli con mod eth1 connection.autoconnect yes

	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

  ```
 
- Khai báo repos cho OpenStack Newton

```
    yum install centos-release-openstack-queens.x86_64 -y
    yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

