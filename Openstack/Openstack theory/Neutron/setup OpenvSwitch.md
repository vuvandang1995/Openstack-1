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
Created symlink from /etc/systemd/system/multi-user.target.wants/openvswitch.service to /usr/lib/systemd/system/openvswitch.service.
[root@compute02 ~]# systemctl status openvswitch.service
● openvswitch.service - Open vSwitch
Loaded: loaded (/usr/lib/systemd/system/openvswitch.service; enabled; vendor preset: disabled)
Active: active (exited) since Sun 2018-08-05 10:16:12 EDT; 17s ago
Main PID: 73958 (code=exited, status=0/SUCCESS)
Aug 05 10:16:12 compute02 systemd[1]: Starting Open vSwitch...
Aug 05 10:16:12 compute02 systemd[1]: Started Open vSwitch.
[root@compute02 ~]#
```
-Kiểm tra OVS version
```
[root@compute02 ~]# ovs-vsctl -V
ovs-vsctl (Open vSwitch) 2.0.0
Complied Apr 19 2018
[root@compute02 ~]#
```



