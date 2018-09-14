# Hướng dẫn cấu hình cinder backup với backend là NFS

Cinder backup sử dụng để restore lại trạng thái của volume trước đó, khi restore bản backup nào thì máy ảo sẽ trở về trạng thái và dữ liệu của bản backup đó.

### Có 2 loại backup là `full backup` và `incremental backup`

- **Full backup** : Là tạo ra 1 bản backup đầy đủ ban đầu cộng thêm thêm phần dữ liệu mới thay đổi, cứ như vậy sẽ gây ra hiện tượng kích thước bản backup sẽ ngày càng tăng.

<img src="https://i.imgur.com/87MXhvJ.png">

- **Incremental backup** : là bản sao lưu trong đó các bản sao liên tiếp của dữ liệu chỉ chứa phần đã thay đổi kể từ khi bản sao lưu trước đó được thực hiện. Khi cần full backup, quá trình khôi phục sẽ cần bản full backup cuối cùng cộng với tất cả các bản incremental backup cho đến thời điểm phục hồi, ưu điểm là giảm bớt kích thước của bản backup mỗi khi update, nhược điểm là phải phụ thuộc vào bản full backup.

<img src="https://i.imgur.com/WcYhiW5.png">

------------------------------------------

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

`openstack volume backup create [--incremental] [--force] VOLUME`
