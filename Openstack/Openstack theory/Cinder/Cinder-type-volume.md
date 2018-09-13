# Tạo type cho volume 

Việc tạo type cho volume giúp ta có thể lựa chọn việc lưu volume ở backend chỉ định muốn lưu, việc chọn type chỉ sử dụng được khi tạo volume trước khi tạo máy ảo. Nếu mặc định không chọn type thì cinder sẽ dựa vào `filter và `weight` để chọn ra backend thích hợp nhất để tạo volume trên đó.


**Ví dụ**: Bên dưới ta đã cấu hình backend lvm và nfs, ta sẽ cấu hình 2 type backend là LVN và nfs dùng để lưu volume.

```
cinder type-create nfs
+--------------------------------------+------+-------------+-----------+
| ID                                   | Name | Description | Is_Public |
+--------------------------------------+------+-------------+-----------+
| 4aa21639-6f47-4dc9-aeb8-82f040309943 | nfs  | -           | True      |
+--------------------------------------+------+-------------+-----------+
```
```
cinder type-key nfs set volume_backend_name=nfs
```
```
cinder extra-specs-list
+--------------------------------------+------+--------------------------------+
| ID                                   | Name | extra_specs                    |
+--------------------------------------+------+--------------------------------+
| 4aa21639-6f47-4dc9-aeb8-82f040309943 | nfs  | {'volume_backend_name': 'NFS'} |
+--------------------------------------+------+--------------------------------+
```

```
cinder type-create lvm
cinder type-key lvm set volume_backend_name=LVM
cinder extra-specs-list
```
