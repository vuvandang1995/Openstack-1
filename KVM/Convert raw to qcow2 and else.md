# Chuyển đổi định sạng raw sang qcow2 và ngược lại

Để chuyển đổi từ định dạng raw sang qcow2, ta dùng câu lệnh: `qemu-img convert -f raw -O qcow2 /var/lib/libvirt/images/VM.img /var/lib/libvirt/images/VM.qcow2`

Để chuyển đổi từ định dạng qcow2 sang raw, ta dùng câu lệnh: `qemu-img convert -f qcow2 -O raw /var/lib/libvirt/images/VM.qcow2 /var/lib/libvirt/images/VM.raw`

Sau khi chuyển đổi, tiến hành shutdown máy ảo. Đồng thời, sửa file xml (driver name và source file)  của VM bằng câu lệnh virsh edit VMname , chuyển type từ raw sàng qcow2 hoặc ngược lại
