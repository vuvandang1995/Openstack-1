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
- NIC1 - ens3: Là dải mạng mà các máy ảo sẽ giao tiếp với bên ngoài. Dải mạng này sử dụng chế độ bridge hoặc NAT của VMware Workstation
- NIC2 - ens4: là dải mạng sử dụng cho các traffic MGNT + API + DATA VM. Dải mạng này sử dụng chế độ hostonly trong VMware Workstation


### 1.3. Mô hình

<img src="http://i.imgur.com/1NjX5NW.png">

### 1.4. IP Planning

<img src="http://i.imgur.com/JT1t7Mi.png">

## 2. Các bước cài đặt
### 2.1. Các bước chuẩn bị trên trên Controller

- Thiết lập hostname

	```sh
	hostnamectl set-hostname controller
	```

- Thiết lập IP 

	```sh
  echo "Setup IP  ens3"
  nmcli c modify ens3 ipv4.addresses 192.168.100.99/24
  nmcli c modify ens3 ipv4.gateway 192.168.100.1
  nmcli c modify ens3 ipv4.dns 8.8.8.8
  nmcli c modify ens3 ipv4.method manual
  nmcli con mod ens3 connection.autoconnect yes
  
  echo "Setup IP  ens4"
  nmcli c modify ens4 ipv4.addresses 10.10.10.99/24
  nmcli c modify ens4 ipv4.method manual
  nmcli con mod ens4 connection.autoconnect yes

  sudo systemctl disable firewalld
  sudo systemctl stop firewalld
  sudo systemctl disable NetworkManager
  sudo systemctl stop NetworkManager
  sudo systemctl enable network
  sudo systemctl start network

  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
  ```
- Khai báo repos cho OpenStack Newton

```sh
    sudo yum install -y centos-release-openstack-newton
    yum update -y

    sudo yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

### 2.2. Các bước chuẩn bị trên trên Compute1

- Thiết lập hostname

    ```sh
    hostnamectl set-hostname compute1
    ```

- Thiết lập IP 

  ```sh
  echo "Setup IP  ens3"
  nmcli c modify ens3 ipv4.addresses 192.168.100.100/24
  nmcli c modify ens3 ipv4.gateway 192.168.100.1
  nmcli c modify ens3 ipv4.dns 8.8.8.8
  nmcli c modify ens3 ipv4.method manual
  nmcli con mod ens3 connection.autoconnect yes
  
  echo "Setup IP  ens4"
  nmcli c modify ens4 ipv4.addresses 10.10.10.100/24
  nmcli c modify ens4 ipv4.method manual
  nmcli con mod ens4 connection.autoconnect yes

  sudo systemctl disable firewalld
  sudo systemctl stop firewalld
  sudo systemctl disable NetworkManager
  sudo systemctl stop NetworkManager
  sudo systemctl enable network
  sudo systemctl start network

  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
  ```
 
- Khai báo repos cho OpenStack Newton

```sh
    sudo yum install -y centos-release-openstack-newton
    yum update -y

    sudo yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

### 2.3. Các bước chuẩn bị trên trên Compute2

Làm tương tự như với compute1
