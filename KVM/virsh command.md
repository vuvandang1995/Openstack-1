## Các lệnh quản lý VM bằng Virsh phổ biến

### Các tùy chọn cơ bản

- Show danh sách tất cả các VM

`virsh list`

- Tạo một VM từ file XML

`virsh create file`

- Khởi động một VM

`virsh start VM`

- Tắt một VM

`virsh destroy VM`

- Xem VM nào được định nghĩa từ một file XML chỉ định

`virsh define filename.xml`

- Xem ID của một VM là gì tương ứng với tên hoặc UUID của nó

`virsh domid hostname`

hoặc: `virsh domid UUID`

- Xem thông tin chi tiết của một VM

`virsh dominfo hostname`

hoặc: `virsh dominfo ID`

- Khởi động lại một VM

`virsh reboot VM`

- Khôi phục một VM từ file snapshot

`virsh restore file`

- Khi một VM đang bị tạm dừng, dùng lệnh sau để tiếp tục

`virsh resume VM`

- Lưu trạng thái của một VM vào file

`virsh save VM filename`

- Tắt một VM

`virsh shutdown VM`

- Tạm dừng một VM

`virsh suspend VM`

### Các tùy chọn quản lý tài nguyên

- Thay đổi bộ nhớ của VM

`virsh setmem [VM] [size]`

- Thay đổi giới hạn tối đa bộ nhớ

`virsh setmaxmem [VM] [size]`

- Thay đổi số CPU ảo

`virsh setvcpus [VM] [count]`

- Xem thông tin của CPU của VM

`virsh vcpuinfo [VM]`

### Các tùy chọn sửa lỗi và giám sát

- Show phiên bản các công nghệ đang sử dụng

`virsh version`

- Xem thông tin VM trong XML

`virsh dumpxml [VM]`

- Xem thông tin KVM server

`virsh nodeinfo`
