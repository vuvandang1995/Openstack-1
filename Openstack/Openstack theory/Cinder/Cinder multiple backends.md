 Mô hình
 
<img src="https://github.com/anhict/images/blob/master/Screenshot_53.png">

Trên mô hình phải được cài đặt sẵn OpenStack và Cinder, tham khảo tại https://github.com/Ducnm37/All/blob/master/Openstack/Setup%20-%20Administator/4.%20Openstack%20queens%20manual%20-%20OpenvSwitch.md#cinder 

## 1. Trên node Cinder

- Cài đặt các gói release trước và cài nfs-common, glusterfs-client : 
```
yum -y install centos-release-gluster
yum -y install nfs-common glusterfs-client
```

- Chỉnh sửa vaf thêm mới vào file:
```
vi /etc/cinder/cinder.conf

[Default]
...
enabled_backends = lvm,nfs,glusterfs

[nfs]
volume_driver = cinder.volume.drivers.nfs.NfsDriver
volume_backend_name = NFS
nfs_shares_config = /etc/cinder/nfs_shares
nfs_mount_point_base = (đường dẫn muốn mount vào, nếu không chọn đường dẫn thì xóa dòng này đi thì system mới mount vào nơi khác được)

[glusterfs]
volume_driver = cinder.volume.drivers.glusterfs.GlusterfsDriver
volume_backend_name = GlusterFS
glusterfs_shares_config = /etc/cinder/glusterfs_shares
glusterfs_mount_point_base = (đường dẫn muốn mount vào, nếu không chọn đường dẫn thì xóa dòng này đi thì system mới mount vào nơi khác được)
```

- Tạo file: 
```
vi /etc/cinder/glusterfs_shares

# create new : specify GlusterFS volumes
192.168.239.196:/testvol2


# testvol2 là trên của volume mà chúng ta sẽ tạo GFS server.
```

- Tạo file:
```
vi /etc/cinder/nfs_shares

192.168.239.198:/mnt/nfs/
# /mnt/nfs/ là thư mục mà chúng ta sẽ share khi cấu hình NFS.
```
- Thực hiện phân quyền và restart lại dịch vụ :
```
chmod 640 /etc/cinder/nfs_shares /etc/cinder/glusterfs_shares 
chgrp cinder /etc/cinder/nfs_shares /etc/cinder/glusterfs_shares 
systemctl restart openstack-cinder-volume.service target.service
```

## 2. Trên node compute

- Cài đặt các gói release, nfs-common và glusterfs-client :
```
yum install centos-release-gluster -y

yum -y install nfs-common glusterfs-client 

```

- Sửa cấu hình nova
```
vi /etc/nova/nova.conf

[Default]
...
osapi_volume_listen = 0.0.0.0
volume_api_class = nova.volume.cinder.API
```
- restart lại nova compute: `systemctl restart openstack-nova-compute.service`

## 3. Trên Node gfs và gfs1


```
vi /etc/hosts

192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs
```

- Reboot lại máy chủ để lấy cấu hình mới nhất :`init 6`

- Phần vùng ổ cứng:
```
fdisk /dev/vdb

root@lvm:~# fdisk /dev/sdb
Command (m for help): n
Partition type:
p   primary (0 primary, 0 extended, 4 free)
e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-62914559, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): +10G


Command (m for help): w
```

- Tải gói định dạng xfs :
`yum install xfsprogs -y`

- Format phân vùng vừa tạo với định dạng xfs :
`mkfs.xfs /dev/vdb1`

- Mount partition vào thư mục /mnt và tạo thư mục /mnt/brick1
`mount /dev/vdb1 /mnt && mkdir -p /mnt/brick1`


- Cài đặt gói glusterfs-server, trước tiên cần cài centos-release-gluster sau đó mới cài được gluster-server
```
yum install centos-release-gluster -y
yum -y install glusterfs-server
```

- Start dịch vụ
```
systemctl start glusterd
systemctl enable glusterd
```

- Tắt selinux
```
vi /etc/sysconfig/selinux 

SELINUX=disabled
```

Kiểm tra lại `sestatus`

### Tiếp tục thực hiện trên gfs1

- Để có thể tạo pool trên storage với server gfs ta cần mở port cho glustered
```
firewall-cmd --add-service=glusterfs --permanent 
firewall-cmd --reload
```
hoặc tắt firewalld `systemctl stop firewalld`


- Tạo 1 pool storage với Server gfs:
`gluster peer probe gfs`

- Kiểm tra trạng thái gluster pool
```
[root@gfs1 glusterd]# gluster peer status
Number of Peers: 1

Hostname: gfs
Uuid: ef498eae-f12b-4e30-9705-871b4f934147
State: Peer in Cluster (Connected)
```
- Khởi tạo volume : Lưu ý sẽ không khởi tạo được testvol2 nếu mục /mnt/brick1 lằm trên root system, mà bắt buộc phải nằm trên phân vùng mới vdb

```gluster  volume create testvol2 replica 2 transport tcp  gfs:/mnt/brick1  gfs1:/mnt/brick1```

- Start volume
```
gluster volume start testvol2
```

## 4. Cài đặt NFS



- Chỉnh sửa file hosts:

```
vi /etc/hosts

192.168.239.190 controller
192.168.239.191 compute1
192.168.239.192 compute2
192.168.239.193 compute3
192.168.239.194 cinder
192.168.239.196	gfs
192.168.239.197	gfs1
192.168.239.198	nfs 
```

Thực hiện phân vùng ổ cứng :
```
fdisk /dev/vdb

root@lvm:~# fdisk /dev/sdb
Command (m for help): n
Partition type:
p   primary (0 primary, 0 extended, 4 free)
e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-62914559, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): +10G

Command (m for help): w
```
- Tải gói định dạng xfs :
```
yum install xfsprogs -y
```
- Format phân vùng vừa tạo với định dạng xfs :
```
mkfs.xfs /dev/vdb1
```
- Mount partition vào thư mục /mnt :
```
mount /dev/vdb1 /mnt
```
- Tạo thư mục /mnt/nfs :
```
mkdir -p /mnt/nfs
```
- Cài đặt gói nfs-kernel-server :
```
yum -y install nfs-utils
```

- Mở port cho nfs hoặc tắt firewalld
```
firewall-cmd --permanent --zone=public --add-service=nfs
firewall-cmd --reload
```

- Bật nfs-server:
```
systemctl enable nfs-server.service
systemctl start nfs-server.service
```

- Thực hiện lệnh cho phép mount thư mục /mnt/nfs đến dải 192.168.239.0/24
```
echo "/mnt/nfs 192.168.239.0/24(rw,no_root_squash)" >> /etc/exports
```
- Thực hiện lệnh để hệ thống tự động mount vào thư mục khi restart hệ thống. :
```
echo "/dev/vdb1 /mnt xfs defaults 0 0" >> /etc/fstab
```
- Restart dịch vụ:
```
service nfs-server restart
```

## 5. Cài đặt trên node Cinder

Thực hiện lệnh để hệ thống tự động mount vào thư mục khi restart hệ thống. :
```
echo "/dev/vdb1 /mnt xfs defaults 0 0" >> /etc/fstab  (cấu hình cho file /etc/fstab để mout cứng, lưu ý là cinder phải tách khỏi controller nếu không khi restart system sẽ được load từ file mout chứ ko phải file root của hệ điều hành)
```
## 6. Kiểm tra

```
root@controller:~# . admin-openrc
root@controller:~# cinder service-list

[root@controller ~]# cinder service-list
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
| Binary           | Host           | Zone | Status  | State | Updated_at                 | Disabled Reason |
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
| cinder-scheduler | controller     | nova | enabled | up    | 2018-09-12T10:24:51.000000 | -               |
| cinder-volume    | controller@lvm | nova | enabled | up    | 2018-09-12T10:24:50.000000 | -               |
| cinder-volume    | controller@nfs | nova | enabled | up    | 2018-09-12T10:24:53.000000 | -               |
+------------------+----------------+------+---------+-------+----------------------------+-----------------+
```
Hoặc xem đã được mout đến nfs-server chưa bằng lệnh `df -h`
```
[root@controller ~]# df -h
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/centos-root   50G  4.2G   46G   9% /
devtmpfs                 2.9G     0  2.9G   0% /dev
tmpfs                    2.9G     0  2.9G   0% /dev/shm
tmpfs                    2.9G  8.8M  2.9G   1% /run
tmpfs                    2.9G     0  2.9G   0% /sys/fs/cgroup
/dev/mapper/centos-home   44G   33M   44G   1% /home
/dev/vda1               1014M  180M  835M  18% /boot
tmpfs                    581M     0  581M   0% /run/user/0
192.168.239/198:/mnt/nfs    60G   33M   60G   1% /var/lib/cinder/mnt/88bcb50137425c193680158748900ff0
```











