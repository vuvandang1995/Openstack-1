# chuyển đổi định dạng của image khi boot image vào volume

Khi tạo volume boot từ image thì image sẽ được lưu bên trong volume mới tạo ra lằm tại `/dev/cinder-volumes` trên controller

Tạo file image được lưu bên trong volume:

`cinder reset-state --state available 16c1e5a7-9956-419a-9a81-e13a908d6166`

`cinder upload-to-image --force True 16c1e5a7-9956-419a-9a81-e13a908d6166 newimage`

Với 16c1e5a7-9956-419a-9a81-e13a908d6166 là ID của volume kiểm tra bằng lệnh `cinder list`

```
Để chuyển đổi từ định dạng raw sang qcow2, ta dùng câu lệnh: qemu-img convert -f raw -O qcow2 /var/lib/libvirt/images/VM.img /var/lib/libvirt/images/VM.qcow2

Để chuyển đổi từ định dạng qcow2 sang raw, ta dùng câu lệnh: qemu-img convert -f qcow2 -O raw /var/lib/libvirt/images/VM.qcow2 /var/lib/libvirt/images/VM.raw
```

Tạo volume boot từ image với image định dạng qcow2 sẽ nhanh hơn định dạng raw do định dạng qcow2 chỉ chứa dung lượng đúng bằng dung lượng thật của dữ liệu, còn raw sẽ có dung lượng bằng dung lượng khi tạo ra file

