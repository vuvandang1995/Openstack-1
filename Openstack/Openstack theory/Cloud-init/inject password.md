# Phần này giới thiệu về inject pass sử dụng để tạo ra 1 random password cho user admin khi tạo máy ảo trên commandline

## Cấu hình trên controller:
- Cấu hình file dashboard:
```
vi /etc/openstack-dashboard/local_settings


OPENSTACK_HYPERVISOR_FEATURES = {
...
    'can_set_password': False,
}
```

- Cấu hình file nova.conf
```
vi /etc/nova/nova.conf

[libvirt]
inject_password=true
```

## Cấu hình trên compute:
```
vi /etc/nova/nova.conf


[libvirt]
virttype = qemu
inject_password=True
inject_key=True
inject_partition=-1

```
