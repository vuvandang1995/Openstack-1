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
192.168.40.61    controller
192.168.40.62    compute
192.168.40.63    compute2
192.168.40.64    block
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

bind-address = 192.168.40.61
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

`sed -i 's/OPTIONS=\"-l 127.0.0.1,::1\"/OPTIONS=\"-l 192.168.40.61,::1\"/g' /etc/sysconfig/memcached`

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
192.168.40.61    controller
192.168.40.62    compute
192.168.40.63    compute2
192.168.40.64    block
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

----------------------------------

## <a name="keystone">3. Cài đặt và cấu hình Keystone </a>

**Lưu ý:** Dịch vụ này chạy trên controller mode, Phiển bản Queens này sẽ không cần sử dụng port 35357 so với các phiên bản trước đó port 35357 dùng cho admin quản trị dịch vụ khác, port 5000 sử dụng quản trị các project người dùng, với phiên bản Queens đã cải tiến  chỉ cần sử dụng port 5000.

Tạo database cho keystone

``` sh
mysql -u root -pducnm37
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'ducnm37';
exit
```
Cài đặt keystone

`yum install -y openstack-keystone httpd mod_wsgi`

Sao lưu file cấu hình keystone

`cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.origin`

Chỉnh sửa file cấu hình keystone

`vi /etc/keystone/keystone.conf`

```
[database]
...
connection = mysql+pymysql://keystone:Welcome123@192.168.40.61/keystone

[token]
...
provider = fernet
```

Đồng bộ database keystone

`su -s /bin/sh -c "keystone-manage db_sync" keystone`

Tạo Fernet key repositories

```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

Bootstrap dịch vụ identity

```
keystone-manage bootstrap --bootstrap-password ducnm37 \
  --bootstrap-admin-url http://192.168.40.61:5000/v3/ \
  --bootstrap-internal-url http://192.168.40.61:5000/v3/ \
  --bootstrap-public-url http://192.168.40.61:5000/v3/ \
  --bootstrap-region-id RegionOne
```

Cấu hình Apache HTTP server

Sao lưu file  `/etc/httpd/conf/httpd.conf `

`cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.origin`

Chỉnh sửa file `/etc/httpd/conf/httpd.conf `
```
vi /etc/httpd/conf/httpd.conf

...
ServerName controller
```

Tạo liên kết tới file `/usr/share/keystone/wsgi-keystone.conf`

`ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/`

Start dịch vụ và cho khởi động cùng hệ thống
```
systemctl enable httpd.service
systemctl start httpd.service
```

Cấu hình tài khoản admin
```
export OS_USERNAME=admin
export OS_PASSWORD=ducnm37
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.40.61:5000/v3
export OS_IDENTITY_API_VERSION=3
```
Tạo service project

`openstack project create --domain default --description "Service Project" service`

Tạo demo project

`openstack project create --domain default --description "Demo Project" demo`

Tạo demo user
```
openstack user create --domain default --password-prompt demo

User Password: ducnm37
Repeat User Password: ducnm37
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | aeda23aa78f44e859900e22c24817832 |
| name                | demo                             |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
Tạo role demo

```
openstack role create user

+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 997ce8d05fc143ac97d83fdfb5998552 |
| name      | user                             |
+-----------+----------------------------------+
```
Gán role demo vào project và user demo

`openstack role add --project demo --user demo user`

Sao lưu file `/etc/keystone/keystone-paste.ini`

`cp /etc/keystone/keystone-paste.ini /etc/keystone/keystone-paste.ini.origin`



Unset biến

`unset OS_AUTH_URL `

Request authentication token cho admin user
```
$ openstack --os-auth-url http://192.168.40.61:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue

Password:ducnm37
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:14:07.056119Z                                     |
| id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
|            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
|            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
| project_id | 343d245e850143a096806dfaefa9afdc                                |
| user_id    | ac3377633149401296f6c0d92d79dc16                                |
+------------+-----------------------------------------------------------------+
```
Request authentication token cho demo user
```
$ openstack --os-auth-url http://192.168.40.61:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue
Password:ducnm37
+------------+-----------------------------------------------------------------+
| Field      | Value                                                           |
+------------+-----------------------------------------------------------------+
| expires    | 2016-02-12T20:15:39.014479Z                                     |
| id         | gAAAAABWvi9bsh7vkiby5BpCCnc-JkbGhm9wH3fabS_cY7uabOubesi-Me6IGWW |
|            | yQqNegDDZ5jw7grI26vvgy1J5nCVwZ_zFRqPiz_qhbq29mgbQLglbkq6FQvzBRQ |
|            | JcOzq3uwhzNxszJWmzGC7rJE_H0A_a3UFhqv8M4zMRYSbS2YF0MyFmp_U       |
| project_id | ed0b60bf607743088218b0a533d5943f                                |
| user_id    | 58126687cbcc4888bfa9ab73a2256f27                                |
+------------+-----------------------------------------------------------------+
```

Tạo các file source script admin-rc và demo-rc

```
vi admin-openrc

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=ducnm37
export OS_AUTH_URL=http://192.168.40.61:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

```
vi demo-openrc

export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=ducnm37
export OS_AUTH_URL=http://192.168.40.61:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

Kiểm tra script
```
. admin-openrc

openstack token issue
```

-------------------------------------

## 4. Cài đặt image service - glance

**Lưu ý:** Dịch vụ này thường chạy trên controller node và sử dụng file system làm backend lưu images.

Tạo database

``` sh
mysql -u root -pducnm37
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'ducnm37';
exit
```

Tạo glance user

``` sh
. admin-openrc
openstack user create glance --domain default --password ducnm37
```

Gán role admin cho user và service glance

`openstack role add --project service --user glance admin`

Tạo glance service entity

`openstack service create --name glance --description "OpenStack Image" image`

Tạo API endpoints cho Image service

``` sh
openstack endpoint create --region RegionOne image public http://192.168.40.61:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b34c3317e72545f89782ae0e917412f9 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d966b23fd6344790a811034e801341cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://192.168.40.61:9292        |
+--------------+----------------------------------+


openstack endpoint create --region RegionOne image internal http://192.168.40.61:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b1ce94efa98349c3994a5e6792be73c6 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d966b23fd6344790a811034e801341cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://192.168.40.61:9292        |
+--------------+----------------------------------+


openstack endpoint create --region RegionOne image admin http://192.168.40.61:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 875f5c0b7176422db7af775e96adfcb2 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d966b23fd6344790a811034e801341cd |
| service_name | glance                           |
| service_type | image                            |
| url          | http://192.168.40.61:9292        |
+--------------+----------------------------------+

```

Cài đặt package

`yum install openstack-glance -y`

Sao lưu cấu hình file config glance-api

`cp /etc/glance/glance-api.conf /etc/glance/glance-api.conf.origin`

Chỉnh sửa file config glance-api

``` sh
vi /etc/glance/glance-api.conf

[database]
connection = mysql+pymysql://glance:ducnm37@192.168.40.61/glance

[keystone_authtoken]

auth_url = http://192.168.40.61:5000  (do ở mục keystone_authtoken đoạn này không có các option này nên ta thêm trực tiếp vào )
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = ducnm37

...

auth_uri = http://192.168.40.61:5000

...
memcached_servers = 192.168.40.61:11211

...
auth_type = password


[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

Sao lưu file config glance-registry

`cp /etc/glance/glance-registry.conf /etc/glance/glance-registry.conf.origin`

Chỉnh sửa file config glance-registry

``` sh
[database]
connection = mysql+pymysql://glance:ducnm37@192.168.40.61/glance

[keystone_authtoken]


auth_url = http://192.168.40.61:5000  (do ở mục keystone_authtoken đoạn này không có các option này nên ta thêm trực tiếp vào )
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = ducnm37

...
auth_uri = http://192.168.40.61:5000

...
memcached_servers = 192.168.40.61:11211

...
auth_type = password

[paste_deploy]
flavor = keystone
```

Đồng bộ glance database

`su -s /bin/sh -c "glance-manage db_sync" glance`

Start dịch vụ glance-api và glance-registry, cho phép khởi động dịch vụ cùng hệ thống

``` sh
systemctl enable openstack-glance-api.service openstack-glance-registry.service
systemctl start openstack-glance-api.service openstack-glance-registry.service
```

Kiểm tra lại cài đặt. Tải image

``` sh
. admin-openrc
wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

Upload image

``` sh
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```

Kiểm tra lại image vừa upload

`openstack image list`

<a name="nova"></a>

## 5. Cài đặt compute service - nova

<a name="5.1"></a>
### 5.1 Cài đặt trên node controller

Tạo database cho nova

``` sh
mysql -u root -pducnm37
CREATE DATABASE nova_api; (sử dụng để request từ các compute với nhau, compute với database, giữa các project với nova, máy ảo với compute)
CREATE DATABASE nova; (sử dụng để thống kê xem các máy ảo thuộc compute nào)
CREATE DATABASE nova_cell0; (thống kê tài nguyên còn lại của các compute)
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'ducnm37';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'ducnm37';
exit
```

Tạo user nova

``` sh
. admin-openrc
openstack user create --domain default --password-prompt nova

User Password:ducnm37
Repeat User Password:ducnm37

```

Gán role admin cho user nova

`openstack role add --project service --user nova admin`

Tạo nova service

`openstack service create --name nova --description "OpenStack Compute" compute`

Tạo API endpoint cho Compute service

``` sh
openstack endpoint create --region RegionOne compute public http://192.168.40.61:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute internal http://192.168.40.61:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute admin http://192.168.40.61:8774/v2.1/%\(tenant_id\)s
```

Tạo người dụng Placement
```
openstack user create --domain default --password-prompt placement

User Password:ducnm37
Repeat User Password:ducnm37
```
Thêm placement user tới service project với admin role:
`openstack role add --project service --user placement admin`

Tạo đối tượng placement API trong service catalog:

`openstack service create --name placement --description "Placement API" placement`

Tạo Placement API service endpoints:

```
openstack endpoint create --region RegionOne placement public http://192.168.40.61:8778
openstack endpoint create --region RegionOne placement internal http://192.168.40.61:8778
openstack endpoint create --region RegionOne placement admin http://192.168.40.61:8778
```


Cài đặt các packages

``` 
yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api -y
```

Sao lưu file cấu hình

`cp /etc/nova/nova.conf /etc/nova/nova.conf.origin`

Chỉnh sửa file cấu hình `/etc/nova/nova.conf`

``` sh
[DEFAULT]
auth_strategy = keystone
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:ducnm37@192.168.40.61
my_ip = 192.168.40.61
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:ducnm37@192.168.40.61/nova_api

[api]
# ...
auth_strategy = keystone

[database]
connection = mysql+pymysql://nova:ducnm37@192.168.40.61/nova

[keystone_authtoken]
# ...
auth_url = http://192.168.40.61:5000/v3
memcached_servers = 192.168.40.61:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = ducnm37

[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
api_servers = http://192.168.40.61:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.40.61:5000/v3
username = placement
password = ducnm37
```
Thêm cấu hình /etc/httpd/conf.d/00-nova-placement-api.conf

```
vi /etc/httpd/conf.d/00-nova-placement-api.conf

<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```
Khởi động lại dịch vụ httpd

`systemctl restart httpd`

Đồng bộ database

`su -s /bin/sh -c "nova-manage api_db sync" nova`

**Lưu ý** bỏ qua log warnning
```
[root@controller ~]# su -s /bin/sh -c "nova-manage api_db sync" nova
/usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:332: NotSupportedWarning: Configuration option(s) ['use_tpool'] not supported
  exception.NotSupportedWarning
```

Đăng ký cơ sở dữ liệu cell0 

`su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova`

Tạo cell1 

`su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova`

Đồng bộ nova database

`su -s /bin/sh -c "nova-manage db sync" nova`

Kiểm tra nova cell0 và cell1 đã được đăng ký chính xác:

```
# nova-manage cell_v2 list_cells
+-------+--------------------------------------+
| Name  | UUID                                 |
+-------+--------------------------------------+
| cell1 | 109e1d4b-536a-40d0-83c6-5f121b82b650 |
| cell0 | 00000000-0000-0000-0000-000000000000 |
+-------+--------------------------------------+
```

Start các service nova và cho phép khởi động dịch vụ cùng hệ thống

``` sh
systemctl enable openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service

systemctl start openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

<a name="5.2"></a>
### 5.2 Cấu hình trên node Compute

**Node compute**

Cài đặt packages

`yum install openstack-nova-compute -y`

Sao lưu file cấu hình

`cp /etc/nova/nova.conf /etc/nova/nova.conf.origin`

Chỉnh sửa file cấu hình

``` sh
[DEFAULT]
auth_strategy = keystone
transport_url = rabbit://openstack:ducnm37@192.168.40.61
enabled_apis = osapi_compute,metadata
my_ip = 192.168.40.62
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[keystone_authtoken]
...
auth_uri = http://192.168.40.61:5000
auth_url = http://192.168.40.61:5000
memcached_servers = 192.168.40.61:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = ducnm37

[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.40.61:6080/vnc_auto.html

[glance]
api_servers = http://192.168.40.61:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.40.61:5000/v3
username = placement
password = ducnm37
```

Chạy câu lệnh sau để xem phần cứng của bạn có hỗ trợ ảo hóa hay không:

`egrep -c '(vmx|svm)' /proc/cpuinfo`

Nếu giá trị là 1 hoặc lớn hơn thì phần cứng của bạn đã hỗ trợ ảo hóa.
Nếu giá trị là 0 thì bạn buộc phải cấu hình libvirt sử dụng QEMU thay vì KVM.

Chỉnh sửa lại section `[libvirt]` trong file `vi /etc/nova/nova.conf`

``` sh
[libvirt]
virt_type = qemu
cpu_mode = none
```

Tiếp tục tiến hành start các service nova và cho phép khởi động dịch vụ cùng hệ thống.

``` sh
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

**Lưu ý:**

- Nếu nova-compute không thể khởi động, check `/var/log/nova/nova-compute.log` và thấy `AMQP server on controller:5672 is unreachable` thì firewall trên controller đang ngăn cản truy cập tới port 5672. Tiến hành mở port và khởi động lại dịch vụ.

- Node compute còn lại tiến hành cấu hình tương tự.

- Sau khi cấu hình xong quay trở lại node controller và kiểm tra các service đã lên hay chưa.

``` sh
. admin-openrc
openstack compute service list
```

Thêm compute node tới cell database
Thực hiện tại node controller

Khởi tạo biến môi trường Admin CLI
```
. admin-openrc
openstack compute service list --service nova-compute
```

VD:
```
[root@controller1 ~]# openstack compute service list --service nova-compute
+----+--------------+----------+------+---------+-------+----------------------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At                 |
+----+--------------+----------+------+---------+-------+----------------------------+
|  6 | nova-compute | compute1 | nova | enabled | up    | 2018-07-09T07:48:45.000000 |
+----+--------------+----------+------+---------+-------+----------------------------+
```

Discover các compute host:

`su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`

KQ:
```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': ddf7293f-05af-49ac-984a-25a71b3383c3
Checking host mapping for compute host 'compute': 39681a59-dbdc-4137-9bc9-52f71c0a684c
Creating host mapping for compute host 'compute': 39681a59-dbdc-4137-9bc9-52f71c0a684c
Found 1 unmapped computes in cell: ddf7293f-05af-49ac-984a-25a71b3383c3

```

**Node compute2**

Cài đặt packages

`yum install openstack-nova-compute -y`

Sao lưu file cấu hình

`cp /etc/nova/nova.conf /etc/nova/nova.conf.origin`

Chỉnh sửa file cấu hình

``` sh
[DEFAULT]
auth_strategy = keystone
transport_url = rabbit://openstack:ducnm37@192.168.40.61
enabled_apis = osapi_compute,metadata
my_ip = 192.168.40.63
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[keystone_authtoken]
...
auth_uri = http://192.168.40.61:5000
auth_url = http://192.168.40.61:5000
memcached_servers = 192.168.40.61:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = ducnm37

[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.40.61:6080/vnc_auto.html

[glance]
api_servers = http://192.168.40.61:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
# ...
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://192.168.40.61:5000/v3
username = placement
password = ducnm37
```

Chạy câu lệnh sau để xem phần cứng của bạn có hỗ trợ ảo hóa hay không:

`egrep -c '(vmx|svm)' /proc/cpuinfo`

Nếu giá trị là 1 hoặc lớn hơn thì phần cứng của bạn đã hỗ trợ ảo hóa.
Nếu giá trị là 0 thì bạn buộc phải cấu hình libvirt sử dụng QEMU thay vì KVM.

Chỉnh sửa lại section `[libvirt]` trong file `vi /etc/nova/nova.conf`

``` sh
[libvirt]
virt_type = qemu
cpu_mode = none
```

Tiếp tục tiến hành start các service nova và cho phép khởi động dịch vụ cùng hệ thống.

``` sh
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```

**Lưu ý:**

- Nếu nova-compute không thể khởi động, check `/var/log/nova/nova-compute.log` và thấy `AMQP server on controller:5672 is unreachable` thì firewall trên controller đang ngăn cản truy cập tới port 5672. Tiến hành mở port và khởi động lại dịch vụ.

- Node compute còn lại tiến hành cấu hình tương tự.

- Sau khi cấu hình xong quay trở lại node controller và kiểm tra các service đã lên hay chưa.

``` sh
. admin-openrc
openstack compute service list
```

Thêm compute node tới cell database
Thực hiện tại node controller

Khởi tạo biến môi trường Admin CLI
```
. admin-openrc
openstack compute service list --service nova-compute
```

VD:
```
[root@controller1 ~]# openstack compute service list --service nova-compute
+----+--------------+----------+------+---------+-------+----------------------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At                 |
+----+--------------+----------+------+---------+-------+----------------------------+
|  6 | nova-compute | compute1 | nova | enabled | up    | 2018-07-09T07:48:45.000000 |
+----+--------------+----------+------+---------+-------+----------------------------+
```

Discover các compute host:

`su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova`

KQ:
```
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': e0512d74-aff1-4734-94f5-0538305d5383
Checking host mapping for compute host 'compute1': dbd4d559-a783-4162-a932-aa2cfd74a083
Creating host mapping for compute host 'compute1': dbd4d559-a783-4162-a932-aa2cfd74a083
Found 1 unmapped computes in cell: e0512d74-aff1-4734-94f5-0538305d5383
```

