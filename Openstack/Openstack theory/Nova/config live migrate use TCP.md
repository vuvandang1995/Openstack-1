# Cấu hình live migrate sử dụng tcp

- Cấu hình trên cả  2 compute

```
sed -i 's/#listen_tls = 0/listen_tls = 0/g' /etc/libvirt/libvirtd.conf
sed -i 's/#listen_tcp = 1/listen_tcp = 1/g' /etc/libvirt/libvirtd.conf
sed -i 's/#auth_tcp = "sasl"/auth_tcp = "none"/g' /etc/libvirt/libvirtd.conf
sed -i 's/#LIBVIRTD_ARGS="--listen"/LIBVIRTD_ARGS="--listen"/g' /etc/sysconfig/libvirtd
```

- Restart lại dịch vụ:
```
systemctl restart libvirtd
systemctl restart openstack-nova-compute.service
```

 - Cấu hình ssh không cần xác thực cho nova không cần mật khẩu trên 2 compute
``` 
usermod -s /bin/bash nova

passwd nova

su nova

ssh-keygen (enter để tạo key)

ssh-copy-id nova@compute2 (đứng tại compute) và ssh-copy-id nova@compute (đứng tại compute2)

```


