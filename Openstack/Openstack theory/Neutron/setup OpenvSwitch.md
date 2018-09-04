# cài đặt openvswitch cho kvm trên centos7

**Bước 1** Cài các gói sau
```
yum install wget openssl-devel  python-sphinx gcc make python-devel openssl-devel kernel-devel graphviz kernel-debug-devel autoconf automake rpm-build redhat-rpm-config libtool python-twisted-core python-zope-interface PyQt4 desktop-file-utils libcap-ng-devel groff checkpolicy selinux-policy-devel -y
```

**Bước 2** Tạo ovs user and download Open vSwitch 2.9

- Tạo user với tên ovs
```
[root@compute02 ~]# useradd ovs
[root@compute02 ~]# su - ovs
[ovs@compute02 ~]$
```

- Cài Open vSwitch rpm
```
[root@compute02 ~]#wget http://mirror.centos.org/centos/7/os/x86_64/Packages/openvswitch-2.0.0-7.el7.x86_64.rpm
[root@compute02 ~]# yum localinstall /home/ovs/openvswitch-2.0.0-7.el7.x86_64.rpm
```

**Bước 3** Start và enable Open vSwitch Service
```
[root@compute02 ~]# systemctl start openvswitch.service
[root@compute02 ~]# systemctl enable openvswitch.service

[root@compute02 ~]# systemctl status openvswitch.service
● openvswitch.service - Open vSwitch
Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; enabled; vendor preset: disabled)
Active: active (exited) since Sun 2018-08-05 10:16:12 EDT; 17s ago
Main PID: 73958 (code=exited, status=0/SUCCESS)
Aug 05 10:16:12 compute02 systemd[1]: Starting Open vSwitch...
Aug 05 10:16:12 compute02 systemd[1]: Started Open vSwitch.

```
- Upgrade lại hệ thống
```
yum install upgrade -y
```
-Kiểm tra OVS version
```
[root@compute02 ~]# ovs-vsctl -V
ovs-vsctl (Open vSwitch) 2.0.0
Complied Apr 19 2018
[root@compute02 ~]#
```

**Bước 4** Tạo OVS bridge và thêm interface vào nó
- Tạo ovs bridge
```
[root@compute02 ~]# ovs-vsctl add-br ovs-br0
```
- Gán i cho interface
```
[root@compute02 ~]# ip addr flush dev eth2
```
- Đăt địa chỉ IP
```
[root@compute02 ~]# ip addr add 192.168.1.4/24 dev ovs-br0
```
- Gán interface vào port của ovs-br0
```
[root@compute02 ~]# ovs-vsctl add-port ovs-br0 eth2
```
- Cho phép link kết nối tới ovs-br0 hoạt động
```
[root@compute02 ~]# ip link set dev ovs-br0 up
```

- Setup trên card mạng eth2
```
[root@compute02 ~]# cd /etc/sysconfig/network-scripts/
[root@compute02 network-scripts]# cp eth2 ifcfg-ovs-br0
[root@compute02 network-scripts]#
[root@compute02 network-scripts]# vi ifcfg-eth2
DEVICE=eth2
HWADDR="00:0c:29:c1:c3:4e"
TYPE=OVSPort
DEVICETYPE=ovs
OVS_BRIDGE=ovs-br0
ONBOOT=yes
```
- Setup trên card mạng ovs-br0
```
[root@compute02 network-scripts]# vi ifcfg-ovs-br0
DEVICE=ovs-br0
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=static
IPADDR=192.168.1.4
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
ONBOOT=yes
```
- Khởi động lại network
```
[root@compute02 ~]#Service network restart
```
- Kiểm tra lại 
```
[root@compute02 ~]# ovs-vsctl show
8dc5f8e7-0e54-4d9d-ba7a-cd6b9b94f470
    Bridge "ovs-br0"
        Port "ovs-br0"
            Interface "ovs-br0"
                type: internal
        Port "eth2"
            Interface "eth2"
    ovs_version: "2.9.2"
[root@compute02 ~]#
```
**Bước 5** Define ovs network
- Tạo file template
```
[root@compute02 ~]# vi /tmp/ovs-network.xml
<network>
<name>ovs-network</name>
<forward mode='bridge'/>
<bridge name='ovs-br0'/>
<virtualport type='openvswitch'/>
</network>
```
- Define network bằng libvirt
```
[root@compute02 ~]# virsh net-define /tmp/ovs-network.xml
Network ovs-network defined from /tmp/ovs-network.xml
[root@compute02 ~]# virsh net-start ovs-network
Network ovs-network started
[root@compute02 ~]# virsh net-autostart ovs-network
Network ovs-network marked as autostarted
[root@compute02 ~]#
```
- Kiểm tra lại
```
[root@compute02 ~]# virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
 ovs-network          active     yes           yes
[root@compute02 ~]#
```
**Bước 6** Tạo máy ảo và kiểm tra lại
- Tạo máy ảo
```
virt-install \
 --name vm1 \
 --memory 512 \
 --disk path=/var/lib/libvirt/images/pac2.qcow2,size=8,format=qcow2,bus=ide,cache=none \
 --vcpus 1 \
 --network network:ovs-network \
 --os-type=Linux \
 --os-variant=Generic \
 --hvm --virt-type kvm \
 --vnc --noautoconsole \
 --import
```
- Kiểm tra 
```
[root@localhost images]# ovs-vsctl show
8dc5f8e7-0e54-4d9d-ba7a-cd6b9b94f470
    Bridge "ovs-br0"
        Port "ovs-br0"
            Interface "ovs-br0"
                type: internal
        Port "eth2"
            Interface "eth2"
    ovs_version: "2.0.0"
```
- Hiển thị các port đang kết nối
```
[root@compute02 ~]# ovs-vsctl list-ports ovs-br0
eth2
vnet0
```
