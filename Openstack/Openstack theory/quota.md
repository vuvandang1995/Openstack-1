# Phần này giới thiệu về quota trong openstack

**Quota**: Được sử dụng để giới hạn tài nguyên của các service trong openstack

- Kiểm tra các quota default hiện có: `openstack quota show --default`

- Kiểm tra quota default của neutron: `neutron quota-default-show`

- Kiểm tra quota default của nova: `nova  quota-defaults`

- Kiểm tra quota default của cinder: `cinder quota-defaults show`

- Set quota cho project:** 
```
openstack quota set --class --'tên trường muốn set quota' 'số lượng quota muốn set' default
```

Ví dụ: `openstack quota set --class --instances 5 default`  (*set cho trường instance với số lượng quota là 5 trong project default*)

- Update quota : 
```
nova quota-class-update --'tên trường muốn update' 'số quota cần update' default
```
Ví dụ: `nova quota-class-update --instances 3 default` 

- 
