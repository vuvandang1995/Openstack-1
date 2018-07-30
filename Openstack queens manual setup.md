# Cài đặt openstack phiên bản queens thủ công

## Mục lục

[1. Chuẩn bị](#prepare)

[2. Setup môi trường cài đặt](#set-up)

- [2.1. Cài đặt trên controller](#controller)
- [2.2. Cài đặt trên 2 compute node và storage node](#compute)

[3. Cài đặt identity service - keystone](#keystone)

[4. Cài đặt image service - glance](#glance)

[5. Cài đặt compute service - nova](#nova)

  - [5.1 Cài đặt trên node controller](#5.1)
  - [5.2 Cài đặt trên node compute](#5.2)

[6. Cài đặt networking service - neutron](#neutron)

  - [6.1 Cài đặt trên node controller](#6.1)
  - [6.2 Cài đặt trên node compute](#6.2)

[7. Cài đặt self-service network](#self-service)

[8. Cài đặt dashboard - horizon](#horizon)

[9. Cài đặt block storage service - cinder](#cinder)

  - [9.1 Cài đặt trên node controller](#8.1)
  - [9.2 Cài đặt trên node storage](#8.2)

[10. Launch máy ảo](#launch)


--------

## <a name="prepare">1. Chuẩn bị </a>

**Distro:** CentOS 7.3, 2 NIC (1 Public Bridge, 1 Private Bridge)

**Môi trường lab:** KVM

**Mô hình**

<img src="http://i.imgur.com/RNQ3tW9.png">

**IP Planning**

<img src="http://i.imgur.com/izdpCT2.png">

## <a name="setup">2. Setup môi trường cài đặt </a>

### <a name="controller">2.1 Cài đặt trên controller </a>

Cấu hình file `/etc/hosts` và `/etc/resolv.conf`

``` sh

vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.100.197    controller
192.168.100.198    compute
192.168.100.199    compute2
192.168.100.200    block
```

``` sh
vi /etc/resolv.conf

nameserver 8.8.8.8
```

Kiểm tra ping ra internet và ra compute

```sh
ping -c 4 openstack.org
ping -c 4 compute
```

Tắt firewall và selinux

``` sh
systemctl disable firewalld
systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

Khởi động lại máy

`init 6`

**Cài đặt Network Time Protocol (NTP)**

Cài đặt package

`yum install chrony -y`

Chỉnh sửa cấu hình file `/etc/chrony.conf`

``` sh
sed -i "s/server 0.centos.pool.ntp.org iburst/server vn.pool.ntp.org iburst/g" /etc/chrony.conf

sed -i 's/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/g' /etc/chrony.conf

sed -i 's/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/g' /etc/chrony.conf

sed -i 's/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/g' /etc/chrony.conf

sed -i 's/#allow 192.168\/16/allow 192.168.100.0\/24/g' /etc/chrony.conf
```

Start dịch vụ và cho khởi động cùng hệ thống

``` sh
systemctl enable chronyd.service
systemctl start chronyd.service
```

Kiểm tra lại

``` sh
[root@controller ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* time.vng.vn                   3   6   377    37   -647us[ -871us] +/-  254ms
```

**Kích hoạt OpenStack repository**

`yum install centos-release-openstack-newton`

**Update packages**

`yum upgrade -y`

**Cài đặt OpenStack client**

`yum install python-openstackclient`

**Cài đặt openstack-selinux**

`yum install openstack-selinux`

**Cài đặt và cấu hình SQL database**

Cài đặt package

`yum install mariadb mariadb-server python2-PyMySQL -y`

Tạo file `/etc/my.cnf.d/openstack.cnf`

``` sh
vi /etc/my.cnf.d/openstack.cnf

[mysqld]

bind-address = 192.168.100.197
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

Start dịch vụ và cấu hình khởi động cùng hệ thống

``` sh
systemctl enable mariadb.service
systemctl start mariadb.service
```

Thực hiện security cho mysql, thực hiện theo các bước sau :

```sh
mysql_secure_installation

Enter current password for root (enter for none): [enter] (ban đầu thì pass sẽ không có gì)
Change the root password? [Y/n]: y
Set root password? [Y/n] y
New password:ducnm37
Re-enter new password:ducnm37
Remove anonymous users? [Y/n]: y
Disallow root login remotely? [Y/n]: y
Remove test database and access to it? [Y/n]: y
Reload privilege tables now? [Y/n]: y
```

**Cài đặt và cấu hình RabbitMQ**

Cài đặt package

`yum install rabbitmq-server`

Start dịch vụ và cấu hình khởi động cùng hệ thống

``` sh
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```

Thêm user `openstack`

``` sh
rabbitmqctl add_user openstack ducnm37

Creating user "openstack" ...
```

Phân quyền cho user openstack được phép config, write, read trên rabbitmq

`rabbitmqctl set_permissions openstack ".*" ".*" ".*"`

**Cài đặt và cấu hình Memcached**

Cài đặt packages

`yum install memcached python-memcached`

Sao lưu cấu hình memcache

`cp /etc/sysconfig/memcached /etc/sysconfig/memcached.origin`

Chính sửa cấu hình memcache

`sed -i 's/OPTIONS=\"-l 127.0.0.1,::1\"/OPTIONS=\"-l 192.168.100.197,::1\"/g' /etc/sysconfig/memcached`

Start dịch vụ và cấu hình khởi động cùng hệ thống

``` sh
systemctl enable memcached.service
systemctl start memcached.service
```

### <a name ="compute">2.2 Cài đặt trên compute node </a>

Cấu hình file `/etc/hosts` và `/etc/resolv.conf`

``` sh

vi /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.100.197    controller
192.168.100.198    compute
192.168.100.199    compute2
192.168.100.200    block
```

``` sh
vi /etc/resolv.conf

nameserver 8.8.8.8
```

Kiểm tra ping ra internet và ra controller

```sh
ping -c 4 openstack.org
ping -c 4 controller
```

Tắt firewall và selinux

``` sh
systemctl disable firewalld
systemctl stop firewalld
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

Khởi động lại máy

`init 6`

**Cài đặt Network Time Protocol (NTP)**

Cài đặt package

`yum install chrony -y`

Chỉnh sửa cấu hình file `/etc/chrony.conf`

``` sh
sed -i "s/server 0.centos.pool.ntp.org iburst/server controller iburst/g" /etc/chrony.conf

sed -i 's/server 1.centos.pool.ntp.org iburst/#server 1.centos.pool.ntp.org iburst/g' /etc/chrony.conf

sed -i 's/server 2.centos.pool.ntp.org iburst/#server 2.centos.pool.ntp.org iburst/g' /etc/chrony.conf

sed -i 's/server 3.centos.pool.ntp.org iburst/#server 3.centos.pool.ntp.org iburst/g' /etc/chrony.conf
```

Start dịch vụ và cho khởi động cùng hệ thống

``` sh
systemctl enable chronyd.service
systemctl start chronyd.service
```

Kiểm tra lại

``` sh
[root@compute ~]# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? controller                    0  10     0   10y     +0ns[   +0ns] +/-    0ns
```

**Kích hoạt OpenStack repository**

`yum install centos-release-openstack-newton -y`

**Update packages**

`yum upgrade -y`

**Cài đặt OpenStack client**

`yum install python-openstackclient -y`

**Cài đặt openstack-selinux**

`yum install openstack-selinux -y`

**Lưu ý**

Thực hiện các bước cài đặt tương tự với node compute2 và block.

## <a name="keystone">3. Cài đặt và cấu hình Keystone </a>

**Lưu ý:** Dịch vụ này chạy trên controller mode

Tạo database cho keystone

``` sh
mysql -u root -pducnm37
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'ducnm37';
exit
```
