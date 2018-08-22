# Các bước cài đặt máy ảo trên KVM


- egrep -c '(svm|vmx)' /proc/cpuinfo (CPU của máy phải hỗ trợ ảo hóa phần cứng Intel VT-x hoặc AMD-V. Để xác định CPU có những tính năng này không. Giá trị 0 chỉ thị rằng CPU không hỗ trợ ảo hóa phần cứng trong khi giá trị khác 0 chỉ thị có hỗ trợ.)
<img src="https://i.imgur.com/utBCcL6.png">

- sudo apt-get install qemu-kvm libvirt-bin bridge-utils (cài đặt `bridge`:  chứa một tiện ích cần thiết để tạo và quản lý các thiết bị bridge, `libvirt-bin`:cung cấp libvirt mà bạn cần quản lý qemu và kvm bằng libvirt, `qemu-kvm`: Phần phụ trợ cho KVM)

- sudo adduser 'Username' libvirtd (thêm user vào nhóm libvirtd)

- virsh --connect qemu:///system list (Kiểm tra danh sách máy ảo)

- sudo brctl addbr br0 (tạo card mạng ảo br0)

- sudo brctl addif br0 Ten_card_mang (add card mạng vật lý vào card mạng ảo br0)

- sudo ip addr show 

- vi /etc/network/interfaces  (Chỉnh sửa file cấu hình mạng)
    #The primary network interface
    iface eth0 inet manual
    #Bridge
    auto br0
    iface br0 inet static
    address 192.168.0.12
    netmask 255.255.255.0
    gateway 192.168.0.1
    dns-nameserver 8.8.8.8
    broadcast 192.168.0.255
    bridge_ports eth0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0

- ifdown -a && ifup -a  (restart lại card mạng)
- sudo brctl show (kiểm tra tất cả các card mạng)
- apt-get install virt-manager  (phần mềm quản lý máy ảo)
- virt-manager (Khởi động virt-manager)

- wget https://sourceforge.mirrorservice.org/g/gn/gns-3/Qemu%20Appliances/linux-microcore-6.4.img  (file test)

# Tạo máy ảo bằng cli virt-install
<ul>
<li>virt-install \  (truy cập vào virt-install)</li>
<li>--name pac \        (đặt tên máy ảo)</li>
<li>-memoryr 512 \    (dung lượng ram)</li>
<li>--vcpus 1 \ (số lõi cpu)</li>
<li>--os-type=Linux \ (loại hệ điều hành)</li>
<li>--os-variant=generic \  (loại hệ điều hành)</li>
<li>--disk path=/root/mini.iso,format=raw,bus=ide,cache=none,size=1 \    (đường dẫn đến file cài đặt, Loại bus, định dạng file chứa hệ điều hành của má ảo,  thông số dung lượng ổ đĩa)</li>
<li>--hvm --virt-type kvm \ (hvm là loại ảo hóa toàn bộ)</li>
<li>--vnc --noautoconsole \ (vnc là loại đồ họa, và noautoconsole là không sử dụng console giữa máy ảo và máy chủ)</li>
<li>--cdrom /var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1804.iso \ (chọn đường dẫn để boot hệ điều hành lần đầu từ cdrom, chỉ sử dụng để boot hệ điều hành từ file iso,cd/dvd)</li>

<li>--import (Dùng khi boot hệ điều hành trực tiếp từ disk path, còn trường hợp boot từ cdrom sẽ không sử dụng được)</li>
</ul>
