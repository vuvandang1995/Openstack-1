# cấu hình cold migrate (= resize) trong bản openstack manual setup

Hướng dẫn cấu hình cold migrate trong OpenStack

- Sử dụng với SSH tunneling để migrate máy ảo hoặc resize máy ảo ở node mới.

**Các bước cấu hình SSH Tunneling giữa các Nodes compute**

- Cho phép user nova có thể login (thực hiện trên tất cả các node compute).
Ví dụ ở đây ta muốn migrate vm từ node compute1 (192.168.40.62) tới node compute2 (192.168.40.63).

`usermod -s /bin/bash nova`

- Thực hiện tạo key pair trên các node compute nguồn cho user nova

```
su nova
ssh-keygen
echo 'StrictHostKeyChecking no' >> /var/lib/nova/.ssh/config
exit
```

- Thực hiện với quyền root, scp key pair tới compute node. Nhập mật khẩu khi được yêu cầu.

``` 
scp /var/lib/nova/.ssh/id_rsa.pub root@compute2:/root/
```

- Trên node đích, thay đổi quyền của key pair cho user nova và add key pair đó vào SSH.

``` 
mkdir -p /var/lib/nova/.ssh
cat /root/id_rsa.pub >> /var/lib/nova/.ssh/authorized_keys
echo 'StrictHostKeyChecking no' >> /var/lib/nova/.ssh/config
chown -R nova:nova /var/lib/nova/.ssh
```

- Từ node compute1 kiểm tra để chắc chắn rằng user `nova` có thể login được vào node compute2 còn lại mà không cần sử dụng password
``` 
su nova
ssh 192.168.40.63
exit
```
- Từ node compute2 kiểm tra để chắc chắn rằng user `nova` có thể login được vào node compute1 còn lại mà không cần sử dụng password

``` 
su nova
ssh 192.168.40.62
exit
```
------------------------

# Cấu hình live migration
Yêu cầu chung:
- Cả hai node nguồn và đích đều phải được đặt trên cùng subnet và có cùng loại CPU.
- Cả controller và compute đều phải phân giải được tên miền của nhau.
- Compute node buộc phải sử dụng KVM với libvirt.

Trên 2 compute
```
sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
```

- Restart lại dịch vụ:

``` sh
systemctl restart libvirtd
systemctl restart openstack-nova-compute.service
```

- Nếu sử dụng block live migration cho các VMs boot từ local thì sửa file `nova.conf` rồi restart lại dịch vụ nova-compute :

``` sh

[libvirt]
.......
block_migration_flag=VIR_MIGRATE_UNDEFINE_SOURCE, VIR_MIGRATE_PEER2PEER, VIR_MIGRATE_LIVE, VIR_MIGRATE_NON_SHARED_INC
......
```

`systemctl restart openstack-nova-compute.service`
