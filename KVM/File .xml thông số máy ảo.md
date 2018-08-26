# File xml chứa các thông số cấu hình máy ảo
Thư mục `/etc/libvirt/qemu/`

<img src="https://i.imgur.com/L8PHwEi.png">

- name : tên máy ảo

- uuid: định danh id cho từng máy ảo

- memory: thông số ram tính bằng kb

- vcpu : số lõi cpu (ở đây là 2)

- type arch: loại hệ điều hành x86-64 bit

- CPU mode: loại cpu ở đây là SandBridge

<img src="https://i.imgur.com/pV2LJmP.png">

- driver name: qemu

- device=disk: thiết bị ổ đĩa cứng

- type: định dạng qcow2 của file chứa hệ điều hành máy ảo

- source file: đường dẫn đến file máy ảo

- bus: ở đây là chuẩn giao tiếp ổ cứng ide

- device=cdroom: thiết bị bộ nhớ chỉ cho phép đọc dữ liệu, ở đây là đọc dữ liệu từ hệ điều hành

- type: định dang raw

- bus: chuẩn giao tiếp ide

- interface type: chế độ card mạng ở đây là bridge

<img src="https://i.imgur.com/7kytnx2.png">

- mac address: địa chỉ mac

- source bridge: ở đây card bridge là card br0

- type: chuẩn card mạng rtl8139

## Chỉnh sửa số CPU và core

- Số cpu và core mặc định là 1 core 1 cpu
<img src="https://i.imgur.com/kSlKHjv.png">

- Ta có thể chỉnh số cpu và core như sau trong file .xml
<img src="https://i.imgur.com/rx0lBWz.png">

- Hoặc chỉnh trên virt-manager
<img src="https://i.imgur.com/BcMhJ78.png">

## Add thêm card mạng

- Thêm các dòng sau và chỉnh sửa lại các thông số (ở ví dụ này là thêm mới card mạng NAT, bằng cách sửa file .xml copy thông số card bridge phía trên và thêm/sửa đổi 1 số thông tin để tạo ra card NAT)

<img src="https://i.imgur.com/UKpMYMV.png">

với phần địa chỉ mac sẽ dữ nguyên 6 bit đầu thay đổi 3 bit sau

## Add thêm ổ đĩa 

- copy và thêm các dòng trong đoạn đánh dấu và chỉnh sửa thông số như đường dẫn, định dạng ổ đĩa ảo dev=sda,hda..., bus,...

<img src="https://i.imgur.com/zl1zZeD.png">
