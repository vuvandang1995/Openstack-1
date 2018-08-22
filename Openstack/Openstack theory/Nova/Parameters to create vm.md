# Phần này sẽ giới thiệu về các tham số cần để tạo một máy ảo



qemu-system-x86_64  

- **enable-kvm**: Bật ảo hóa hỗ trợ phần cứng

- **name instance-00000024**: Tên máy ảo

- **machine pc-i440fx-trusty, accel=kvm, usb=off**: Qemu sẽ mô phỏng một loạt các kiến trúc như 1 máy tính thông thường hoặc kiến trúc x86, 32-bit hoặc 64-bit, địa chỉ MAC, kiến trúc bộ tập lệnh RISC, kiến trúc hệ thống. Nếu KVM hỗ trợ phần cứng được sử dụng thì VD-T đã được bật trên BIOS , tham số accel=kvm. Nếu không sử dụng hỗ trợ phần cứng chỉ sử dụng mô phỏng thuần túy thì thông số accel = tcg, -no-kvm

- **cpu SandyBridge,+erms, +smep, +fsgsbase, +pdpe1gb, +rdrand, +f16c, +osxsave, +dca, +pcid, +pdcm, +xtpr, +tm2, +est, +smx, +vmx, +ds_cpl, +monitor, +dtes64, +pbe, +tm, +ht, +ss, +acpi, +ds, +vme**: Đặt CPU, SandyBridge là bộ vi xử lý intel, cộng với việc thông số của cpu được thêm vào và hiển thị tại /proc/cpuinfo

- **m 2048 -realtime mlock=off**: Thông số RAM, kích thước ram chạy trên máy ảo sẽ được điều chỉnh với MemoryBallooning -device virtio-balloon-pci, id=balloon0, bus=pci.0, addr=0x5

- **smp 1, sockets=1, cores=1, threads=1**: SMP là Symmetrical Multi-Processing (bộ đa xử lý đối xứng), nó đề cập đến một bộ vi xử lý nhiều CPUs trên 1 máy tính và nhiều CPU chia sẻ bộ nhớ thông qua bus hệ thống
<ul>
   <ul>
      
<img src="https://i.imgur.com/dfxsm3o.jpg">

<li>Tuy nhiên phương pháp này cũng gây bất lợi cho các CPUs khi truy cập vào RAM thông qua các bus, lúc này các bus sẽ trở thành nút cổ chai khiến CPU truy cập vào RAM rất chậm. Để giải quyết vấn đề này kiến trúc NUMA được sử dụng, nó gọi là kiến trúc truy cập bộ nhớ không đồng nhất</li>

<img src="https://i.imgur.com/EnPcFfo.jpg">

<li>Trong kiến trúc NUMA mỗi CPU có một bộ nhớ local trực tiếp, việc truy cập vào bộ nhớ local rất nhanh và nó không cần sử dụng bus hệ thống. Điều đó có thể làm tăng hiệu suất khi có nhiều CPU truy cập vào bộ nhớ. Kiến trúc NUMA có thể xem bằng lệnh numact</li>

<li>Các thông số smp1, sockets=1(*số lượng khe cắm ở bo mạch chủ*), cores=1, threads=1. Mô phỏng bộ xử lý với 1 vcpu, 1 socket, 1 core và 1 threads (*số luồng xử lý trên mỗi core*)</li>

Ví dụ: một máy ảo có 2 CPU, mỗi CPU có 4 core và 2 hyper-threading vậy số CPU sẽ là 2x4x2=16 CPU

- **uuid 1f8e6f7e-5a70-4780-89c1-464dc0e7f308**: UUID máy ảo

- **smbios type=1, manufacturer=OpenStack Foundation, product=OpenStack Nova, version=2014.1, serial=80590690-87d2-e311-b1b0-a0481cabdfb4, uuid=1f8e6f7e-5a70-4780-89c1-464dc0e7f308**: smbios là System Management BIOS được sử dụng để biểu diễn thông tin kiến trúc x86
<ul>
   <ul>
<li>Trên một hệ thống Unix ta có thể dùng lệnh dmidecode để xem thông tin SMBIOS, thông tin sẽ được chia thành nhiều bảng và gọi là type (ở đây chọn type 1):</li>

      
<li>Type0 là thông tin BIOS:</li>
<img src="https://i.imgur.com/qUJaAfp.jpg">

<li>Type 1 là thông tin hệ thống:</li>
<img src="https://i.imgur.com/3Yo2xDJ.jpg">

<li>Type 2 là thông tin bo mạch chủ:</li>
<img src="https://i.imgur.com/c3Op5VB.jpg">

</ul>
</ul>

- **my chardev = charmonitor, id = monitor, mode = control**: Quản lý máy ảo chạy qemu bằng monitor, có 2 cách truy cập vào qemu monitor console:
<ul>
  <ul>
    <li>Kết nối tới máy ảo với VNC, Ctrl+Alt+2 để enter, Ctrl+Alt+1 để trở về normal console</li>

<li>Chỉ định tham số -monitor khi qemu khởi động (*ví dụ -monitor stdio sẽ cho  phép chuẩn đầu vào giống như 1 lệnh monitor*)</li>
</ul>
</ul>

- **no-hpet**: Thời gian hệ thống xác định bởi -rtc -rtc[base = utc | localtime |date] [, clock = host | vm] [, driftfix =none | slew]
<ul>
  <ul>
     <li>HighPrecision Event Timer (HPET) là 1 cài đặt thời gian chính xác hơn</li>
  </ul>
</ul>

- **device usb-tablet, id=input0**: Mô phỏng bộ điều khiển PCI UHCI USB thông qua tablet, chuột có thể di chuyển tự do giữa các host và guest

- **device cirrus-vga, id=video0, bus=pci.0, addr=0x2**: Mô phỏng VGA card

- **netdev tap, fd = 32, id = hostnet0, vhost = on, vhostfd = 37**: Network card HOST -netdev tap, fd=32, id= hostnet0 , vhost=on, vhostfd=37

- **device virtio-net-pci,netdev=hostnet0,id=net0,mac=fa:16:3e:d1:2d:99,bus=pci.0,addr=0x3**: Network card GUEST  -device virtio-net-pci, netdev= hostnet0 , id=net0, mac=fa:16:3e:d1:2d:99,bus=pci.0,addr=0x3
<ul>
  <ul>
     <li>Các virtio trên card mạng là 1 paravirtualized</li>
</ul>
  </ul>
     
- **drive file=/var/lib/nova/instances/1f8e6f7e-5a70-4780-89c1-464dc0e7f308/disk, if=none, id=drive-virtio-disk0, format=qcow2, cache=none**: thông số hard disk của host

- **device virtio-blk-pci, scsi=off, bus=pci.0, addr=0x4, drive=drive-virtio-disk0, id=virtio-disk0, bootindex=1**:thông số hard disk của guest
<ul>
  <ul>
     <li>Các virtio về lưu trữ là 1 paravirtualized driver</li>
</ul>
  </ul>
     
- **chardev file, id=charserial0, path=/var/lib/nova/instances/1f8e6f7e-5a70-4780-89c1-464dc0e7f308/console.log**: Log của host

- **device isa-serial, chardev=charserial0, id=serial0**: Log của guest
<ul>
  <ul>
     <li>Ta cần cấu hình trong  image để ghi log vào console: /boot/grub/grub.cfg has linux/boot/vmlinuz-3.2.0-49-virtual root=UUID=6d2231e4-  0975-4f35-a94f-56738c1a8150 ro console=ttyS0</li>
</ul>
  </ul>
     
- **chardev pty, id=charserial1**:

- **device isa-serial, chardev=charserial1, id=serial1**:

- **vnc 0.0.0.0:12**: Địac hỉ VNC và port tương ứng

- **chardev socket,id=charmonitor,path=/var/lib/libvirt/qemu/instance-00000024.monitor,server,nowait**

- **rtc base = utc, drift fix = slew**

- **device piix3-usb-uhci,id=usb,bus=pci.0,addr=0x1.0x2**

## Ngoài 24 thông số cơ bản trên còn một số các thông số như:

- no-user-config

- nodefaults

- global kvm-pit.lost_tick_policy=discard

- no-shutdown

- boot strict=on

- k en-us

- device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x5

