# Phần này hướng dẫn cài openstack trên packstack-queens và tạo máy ảo trên dashboard
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
- OpenStack Queens
- NIC1 - eth0: Là dải mạng mà các máy ảo sẽ giao tiếp với bên ngoài. Dải mạng này sử dụng chế độ bridge hoặc NAT của VMware Workstation
- NIC2 - eth1: là dải mạng sử dụng cho các traffic MGNT + API + DATA VM. Dải mạng này sử dụng chế độ hostonly trong VMware Workstation


### 1.3. Mô hình

 - Ở phần này sẽ gộp dải manager +API +DATA vào dải provider network và dải self service network riêng

<img src="https://i.imgur.com/8qTHAh0.png">

**Trong openstack có 2 chế độ card mạng là Provider và self-service:**
<ul>
<li>Provider: Là chế độ cho phép các máy ảo bên trong các compute kết nối trực tiếp ra ngoài internet thông qua dải provider</li>
<li>Self service: Là chế độ cho phép các máy ảo kết nối tới 1 dải ip private và từ dải đó sẽ được NAT hoặc không NAT ra giải provider qua 1 con router ảo có các interface  đấu nối 1 đầu tới dải provider và 1 đầu tới giải private để đi internet, mục đích là để chia các máy ảo vào các vùng mạng khác nhau để thuận tiện cho việc quản lý hoặc theo yêu cầu của người sử dụng (ở mô hình này sẽ sử dụng dải self service bằng dải 10.10.10.0/24 trùng với giải management)</li>
</ul>
			
			
 

### 1.4. IP Planning

<img src="https://i.imgur.com/4UYKYHk.png">

## 2. Các bước cài đặt
### 2.1. Các bước chuẩn bị trên trên Controller
- Đầu tiên start network manager : `systemctl start NetworkManager`

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
  nmcli c modify eth0 connection.autoconnect yes
  
  echo "Setup IP  eth1"
  nmcli c modify eth1 ipv4.addresses 10.10.10.70/24
  nmcli c modify eth1 ipv4.method manual
  nmcli c modify eth1 connection.autoconnect yes

  sudo systemctl disable firewalld
  sudo systemctl stop firewalld
  sudo systemctl disable NetworkManager
  sudo systemctl stop NetworkManager
  sudo systemctl enable network
  sudo systemctl start network

  sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
  ```
- Khai báo repos cho OpenStack Queens

```
	yum install centos-release-openstack-queens.x86_64 -y

	sudo yum install -y wget crudini fping   (vi /etc/yum.repos.d/CentOS-QEMU-EV.repo sửa lại đường dẫn repos 'baseurlhttp://mirror.centos.org/centos-7/7/virt/x86_64/kvm-common/' nếu repos báo lỗi sai)
	yum install -y openstack-packstack
	init 6

```

### 2.2. Các bước chuẩn bị trên trên Compute1

- Đầu tiên start network manager : `systemctl start NetworkManager`

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
	nmcli c modify eth0 connection.autoconnect yes

	echo "Setup IP  eth1"
	nmcli c modify eth1 ipv4.addresses 10.10.10.69/24
	nmcli c modify eth1 ipv4.method manual
	nmcli c modify eth1 connection.autoconnect yes

	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

  ```
 
- Khai báo repos cho OpenStack Queens

```
    yum install centos-release-openstack-queens.x86_64 -y
    yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

### 2.3. Các bước chuẩn bị trên trên Compute2

- Đầu tiên start network manager : `systemctl start NetworkManager`

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
	nmcli c modify eth0 connection.autoconnect yes

	echo "Setup IP  eth1"
	nmcli c modify eth1 ipv4.addresses 10.10.10.68/24
	nmcli c modify eth1 ipv4.method manual
	nmcli c modify eth1 connection.autoconnect yes

	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

  ```
 
- Khai báo repos cho OpenStack Queens

```
    yum install centos-release-openstack-queens.x86_64 -y
    yum install -y wget crudini fping
    yum install -y openstack-packstack
    init 6
```

Thực hiện lệnh fping từ máy controller để kiểm tra các IP trên các máy đã thiết lập đúng hay chưa.
fping 10.10.10.68 10.10.10.69

**Để tránh trường hợp đang cà openstack trên packstack mà bị mất phiên ssh ta cài đặt byobu trên centos 7 sẽ giữ được trạng thái cài đặt của openstack ngay cả khi ngắt phiên ssh**

- Cài đặt epel-release: `yum install -y epel-release`

- Cài đặt byobu:  `yum install byobu -y`

Note: sau khi cài đặt, tiến hành gõ "byobu" và thực hiện các tác vụ trên terminal của byobu.
Trường hợp tắt phiên, sau khi ssh lại máy chủ, tiến hành gõ "byobu", giao diện terminal lúc đầu sẽ hiện ra.


### 2.4. Cài đặt openstack

```
	packstack --allinone \
	--default-password=Welcome123 \
	--os-cinder-install=y \
	--os-neutron-ovs-bridge-mappings=extnet:br-ex \
	--os-neutron-ovs-bridge-interfaces=br-ex:eth0 \
	--os-neutron-ovs-bridges-compute=br-ex \
	--os-neutron-ml2-type-drivers=vxlan,flat \
	--os-controller-host=192.168.40.70 \
	--os-compute-hosts=192.168.40.69,192.168.40.68 \
	--os-neutron-ovs-tunnel-if=eth1 \
	--provision-demo=n
```
- Upload image
```
	curl http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img | glance \
	image-create --name='cirros' \
	--visibility=public \
	--container-format=bare \
	--disk-format=qcow2
```

- Tạo network public
```
	neutron net-create external_network --provider:network_type flat \
	--provider:physical_network extnet  \
	--router:external \
	--shared
```

- Tạo subnet trong network public
```
	neutron subnet-create --name public_subnet \
	--enable_dhcp=False \
	--allocation-pool=start=192.168.40.3,end=192.168.40.254 \
	--gateway=192.168.40.1 external_network 192.168.40.0/24
```

- Tạo network private (dải self service)

```
	neutron net-create private_network
	neutron subnet-create --name private_subnet private_network 10.0.0.0/24 \
	--dns-nameserver 8.8.8.8
```

- Tạo router và addd các interface (đây chính là router ảo để nat dải private(self service network) ra giải provider
```
	neutron router-create router
	neutron router-gateway-set router external_network
	neutron router-interface-add router private_subnet
```

### 3.Fix lỗi nếu có

#### Trước khi vào dashboard để tạo máy ảo ta cần chỉnh sửa một vài lỗi sau

**Không nhận metadata:**

- Sửa file /etc/neutron/dhcp_agent.ini trên controller node

	 `vi /etc/neutron/dhcp_agent.ini`

- Bỏ comment và sửa thành true

	`enable_isolated_metadata = True`
	
**Đứng trên Controller thực hiện lệnh dưới để sửa các cấu hình cần thiết:**

	sed -i -e 's/enable_isolated_metadata=False/enable_isolated_metadata=True/g' /etc/neutron/dhcp_agent.ini

	ssh -o StrictHostKeyChecking=no root@192.168.40.69 "sed -i -e 's/compute1/192.168.40.69/g' /etc/nova/nova.conf"

	ssh -o StrictHostKeyChecking=no root@192.168.40.68 "sed -i -e 's/compute2/192.168.40.68/g' /etc/nova/nova.conf"
	
**Khởi động lại cả 03 node Controller, Compute1, Compute2**

	ssh -o StrictHostKeyChecking=no root@192.168.40.68 "init 6"

	ssh -o StrictHostKeyChecking=no root@192.168.40.69 "init 6"

	init 6
	
**Đăng nhập lại vào Controller bằng quyền root và kiểm tra hoạt động của openstack sau khi cài:**

- Khai báo biến môi trường  
	
	`source ~/keystonerc_admin`

- Kiểm tra hoạt động của openstack  (lưu ý: có thể phải mất vài phút để các service của OpenStack khởi động xong)

	`openstack token issue`
	
<img src="https://i.imgur.com/N3hZ1i0.png">

- Ngoài ra có thể kiểm tra thêm bằng cách lệnh khác: openstack user list , openstack service list, openstack catalog list

- Kiểm tra trạng thái của các service NOVA bằng lệnh `openstack compute service list`, nếu state là `up` thì có thể tiếp tục các bước dưới

<img src="https://i.imgur.com/HrqgvgN.png">

- Kiểm tra trạng thái của dịch vụ neutron bằng lệnh `neutron agent-list`, nếu có biểu tượng :-) thì có thể tiếp tục bước dưới

<img src="https://i.imgur.com/DS2ogLx.png">



#### Cuối cùng truy cập vào web theo địa chỉ http://192.168.40.70/dashboard , tài khoản là admin, mật khẩu là Welcome123 để quản trị openstack








