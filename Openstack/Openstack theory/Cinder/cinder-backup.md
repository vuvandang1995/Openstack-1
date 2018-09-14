# Hướng dẫn cấu hình cinder backup với backend là NFS

Cinder backup sử dụng để restore lại trạng thái của volume trước đó, vó 2 loại backup là `full backup` và `incremental backup`

- Full backup : Là tạo ra 1 bản sao đầy đủ của bản ban đầu

- Incremental backup : Là bản back up cho bản full backup khi backup cho bản chính, và khi tạo backup cho bản chính nó sẽ tạo ra bản backup cho bản backup trước đó chứ ko tạo riêng 1 bản backup cho bản chính. Điều kiện để có thể tạo Incremental backup là cần có 1 bản full backup.




1. Cấu hình NFS server

2. Cấu hình cinder backup

3. Hướng dẫn thực hiện tạo backup cho cinder và restore lại backup

--------------------

## 1. Cấu hình NFS server

Tham khảo hướng dẫn cấu hình NFS server làm backend cho cinder [tại đây](https://github.com/Ducnm37/All/blob/master/Openstack/Openstack%20theory/Cinder/Cinder%20multiple%20backends.md)

## 2. Cấu hình cinder backup

Chỉnh sửa file `/etc/cinder/cinder.conf`

Thêm vào 2 dòng sau trong section [DEFAULT]

```
backup_driver=cinder.backup.drivers.nfs
backup_share=HOST:EXPORT_PATH
```

Trong đó `HOST:EXPORT_PATH` là địa chỉ của đường dẫn đã được thêm vào file `/etc/exports` ở bên nfs server, lưu ý là tạo 1 lvm mới sau đó mount vào 1 mục mới bên nfs server sau đó thêm đường dẫn và phân quyền thư mục vào file `/etc/export`, bên cinder chỉ cần cấu hình `backup_driver` và `backup_share` sau đó restart lại dịch vụ cinder `systemctl restart openstack-cinder-*`

Ví dụ: `192.168.239.198:/mnh/nfs` (bên nfs server đã mount 1 ổ mới vào mục `/mnh`)

Khởi động dịch vụ cinder backup

```
systemctl enable openstack-cinder-backup.service
systemctl start openstack-cinder-backup.service
```

Kiểm tra tại node cinder xem đã được mount chưa bằng lệnh `df -h`

## 3. Hướng dẫn thực hiện tạo backup cho cinder và restore lại backup

Lưu ý: Để tạo được backup, volume phải ở trạng thái `available`. Do vậy nếu volume đang được attach vào máy ảo, cần phải gỡ nó ra.

Xem danh sách volume

`cinder list`

Show trạng thái volume `ducnm`

`cinder show ducnm`

Xem danh sách các backup

`cinder backup-list`

Tạo backup cho volume `ducnm`

`cinder backup-create --display-name thao_backup ducnm`

Kiểm tra lại bằng lệnh `cinder backup-list`

Sang phía nfs server để kiểm tra lại metadata file của backup được tạo ra.

Để restore lại volume về trạng thái lúc tạo backup, sử dụng lệnh sau

`cinder backup-restore --volume-id <volume-id> <backup-id>`

Để xóa một backup

`cinder backup-delete <backup-id>`
