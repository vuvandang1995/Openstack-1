# Phần này giới thiệu về quota trong openstack

**Quota**: Được sử dụng để giới hạn tài nguyên của các service trong openstack

Kiểm tra các quota default hiện có  `openstack quota show`

Kiểm tra quota default của neutron `neutron quota-default-show`

- Set quota cho project: 
```
openstack quota set --class --'tên trường muốn set quota' 'số lượng quota muốn set' default
```

Ví dụ: `openstack quota set --class --instances 5 default`  (*set cho trường instance với số lượng quota là 5 trong project default*)

