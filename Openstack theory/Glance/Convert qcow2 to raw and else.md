# chuyển đổi định dạng của image khi boot image vào volume


`cinder reset-state --state available 16c1e5a7-9956-419a-9a81-e13a908d6166`

`cinder upload-to-image --force True 16c1e5a7-9956-419a-9a81-e13a908d6166 newimage`

Với 16c1e5a7-9956-419a-9a81-e13a908d6166 là ID của volume kiểm tra bằng lệnh `cinder list`

```
Để chuyển đổi từ định dạng raw sang qcow2, ta dùng câu lệnh: qemu-img convert -f raw -O qcow2 /var/lib/libvirt/images/VM.img /var/lib/libvirt/images/VM.qcow2

Để chuyển đổi từ định dạng qcow2 sang raw, ta dùng câu lệnh: qemu-img convert -f qcow2 -O raw /var/lib/libvirt/images/VM.qcow2 /var/lib/libvirt/images/VM.raw
```

Tạo volume boot từ image với image định dạng qcow2 sẽ nhanh hơn định dạng raw

