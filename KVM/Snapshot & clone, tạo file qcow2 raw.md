# Snapshot và clone máy ảo

## Có 2 loại snapshot là internal và external
**File snapshot trong thư mục** /var/lib/libvirt/qemu/snapshot/b/Snapshot 3.xml (file snapshot là Snapshot 3.xml của máy ảo `b`)

**- 1 chú ý là chỉnh sửa file snapshot xml nhưng khi revirt file snapshot sẽ vẫn quay trở lại trạng thái lúc chưa chỉnh sửa.**

### 1. Internal

- virsh snapshot-create-as VMname --name "Snapshot 1" --description "First snapshot" --atomic  (*tạo snapshot*)

- virsh snapshot-list VMname (*kiểm tra snapshot đã tạo*)

- virsh snapshot-revert VMname --snapshotname "Snapshot 1"  (*sử dụng snapshot*)

- virsh snapshot-delete VMname Snapshotname  (*xóa snapshot*)

Internal snapshot chỉ hỗ trợ định dạng `qcow2` vì thế hãy xem rằng ổ đĩa của máy ảo thuộc định dạng nào bằng câu lệnh `qemu-img info /` . Nếu định dạng ổ đĩa không phải là `qcow2`, hãy chuyển nó sang định dạng này bằng câu lệnh `qemu-img convert`.

- qemu-img convert -f qcow2 <file1> -O raw <file1.raw>  (*chuyển từ qcow2 sang raw, hoặc đổi vị trí qcow2 và raw nếu muốn chuyển đổi ngược lại*)

### 2. Hướng dẫn tạo và quản lí External Snapshot 

**Tạo snapshot**

- Tiến hành kiểm tra ổ đĩa mà máy ảo muốn tạo snapshot đang sử dụng bằng câu lệnh `virsh domblklist VMname --details`

<img src="http://i.imgur.com/6imSrzk.png">

- Tiến hành tạo snapshot bằng câu lệnh `virsh snapshot-create-as VMname snapshot1 "My First Snapshot" --disk-only --atomic`

Trong đó `--disk-only` dùng để tạo snapshot cho riêng ổ đĩa.

- Check lại danh sách bằng câu lệnh `virsh snapshot-list VMname`

<img src="http://i.imgur.com/S6klysK.png">

- Snapshot đã được tạo tuy nhiên nó chỉ lưu trữ duy nhất trạng thái ổ đĩa:

<img src="http://i.imgur.com/A9Wyiwr.png">

- Kiểm tra lại ổ đĩa mà máy ảo đang sử dụng:

<img src="http://i.imgur.com/WBeHKg4.png">

Lúc này ổ đĩa cũ đã biến thành trạng thái `read-only`, VM dùng ổ đĩa mới để lưu dữ liệu và `backingfile` sẽ là ổ đĩa ban đầu. Hãy xem thông tin của ổ đĩa này:

<img src="http://i.imgur.com/BuqBwSR.png">

**Revert lại trạng thái snapshot**

- Để revert lại trạng thái của external snapshot, bạn phải cấu hình file XML bằng tay bởi libvirt vẫn chưa hỗ trợ cho việc này. Giả sử VM đang ở snapshot2 và bạn muốn quay trở lại snapshot1:
  - Lấy đường dẫn tới ổ đĩa được tạo ra khi snapshot:

     <img src="http://i.imgur.com/8QkfP0Q.png">

  - Kiểm tra để đảm bảo nó còn nguyên vẹn và được kết nối với backing file

     <img src="http://i.imgur.com/3GTE5yy.png">

  - Chỉnh sửa bằng tay file XML, bỏ ổ đĩa hiện tại và thay thế bằng ổ đĩa ở trạng thái snapshot1:

     <img src="http://i.imgur.com/wF6FA7k.png">

  - Kiểm tra lại xem máy ảo đã sử dụng đúng ổ chưa:

     <img src="http://i.imgur.com/TDdUY5q.png'">

  - Khởi động máy ảo và kiểm tra

**Xóa external snapshot**

- Quy trình xóa một external snapshot khá phức tạp. Để có thể xóa, trước tiên bạn phải tiến hành hợp nhất nó với ổ đĩa cũ. Có hai kiểu hợp nhất đó là:

  <ul>
  <li>blockcommit: Hợp nhất dữ liệu với ổ đĩa cũ.</li>
  <li>blockpull : Hợp nhất dữ liệu với ổ đĩa được tạo ra khi snapshot. Ổ đĩa sau khi hợp nhất sẽ luôn có định dạng qcow2.</li>
  </ul>

- Hợp nhất sử dụng `blockcommit` : 
  <ul>
  <li>Kiểm tra ổ đĩa hiện tại mà máy ảo sử dụng bằng câu lệnh "virsh domblklist VM1"</li>
  <li>Xem thông tin backing file của ổ đĩa đang được sử dụng bằng câu lệnh "qemu-img info --backing-chain /vmstore1/vm1.snap4 | grep backing"</li>
  <li>Hợp nhất snapshot bằng câu lệnh "virsh blockcommit VM1 hda --verbose --pivot --active" . Lưu ý đối với ubuntu, chỉ bản 16.04 mới hỗ trợ câu lệnh này</li>
  <li>Check lại ổ đĩa đang sử dụng bằng câu lệnh "virsh domblklist VM1"</li>
  <li>Kiểm tra lại danh sách các snapshot bằng câu lệnh "virsh snapshot-list VM1"</li>
  <li>Xóa snapshot bằng câu lệnh "virsh snapshot-delete VM1 snap1 --children --metadata"</li>
  </ul>

- Hợp nhất sử dụng `blockpull`: 

 - Xem ổ đĩa hiện tại máy ảo đang sử dụng:

 <img src="http://i.imgur.com/Zk7Yh8t.png">

 - Xem backing file của VM:

 <img src="http://i.imgur.com/jvqgfRF.png">

 - Hợp nhất ổ đĩa cũ với ổ đĩa snapshot:

 <img src="http://i.imgur.com/9HtNPZs.png">

 - Xóa bỏ base image và snapshot metadata bằng câu lệnh `virsh snapshot-delete VMname snapshotname --metadata`
 
-----

#Tạo file với định dạng qcow2 và raw
qemu-img create -f raw /var/lib/libvirt/images/myfile.raw 10G (*tạo file raw với dung lượng 10G)
qemu-img create -f qcow2 /var/lib/libvirt/images/thao.qcow2 10G (tạo file qcow2)

## Hướng dẫn triển khai máy ảo từ template sử dụng phương thức "Clone" trên virt-manager

- Mở virt-manager, chọn máy ảo đã được chuyển đổi thành template, click chuột phải và chọn `Clone`

<img src="http://i.imgur.com/N81Pgvh.png">

- Điền tên và chọn các thông tin liên quan như network, storage... Sau đó click vào `Clone` để tiến hành tạo máy ảo mới.
- Sau khi hoàn tất, máy ảo đã sẵn sàng để sử dụng

<img src="http://i.imgur.com/8yXMAne.png">

- Lưu ý rằng máy ảo khi tạo ra với phương thức "Clone" sẽ hoàn toàn độc lập với template, nó vẫn có thể chạy khi ta bỏ đi template.

## Sau đây là các bước cụ thể để tạo template với máy ảo Ubuntu 14.04 64bit trên KVM

- Cài đặt Ubuntu 14.04 trên KVM bằng công cụ mà bạn ưa thích (virt-manager, virt-install...). Cài đặt lên những dịch vụ cần thiết.
- Shutdown máy ảo bằng câu lệnh `virsh shutdown VMname`
- Sử dụng `virt-sysprep` để "niêm phong" máy ảo:
  <ul>
  <li>"virt-sysprep" là một tiện ích nằm trong gói "libguestfs-tools-c" được sử dụng để loại bỏ những thông tin cụ thể của hệ thống đồng thời niêm phong và biến máy ảo trở thành template</li>
  <li>Có 2 options để dùng  "virt-sysprep" đó là "-a" và "-d". Tùy chọn "-d" được sử dụng với tên hoặc UUID của máy ảo, tùy chọn "-a" được sử dụng với đường dẫn tới ổ đĩa máy ảo.</li>
  </ul>

- Người dùng cũng có thể liệt kê các options cụ thể khi sử dụng với `virt-sysprep`. Ví dụ: `virt-sysprep --operations ssh-hostkeys,udev-persistent-net -d`.
- Giờ đây ta đã có thể đánh dấu máy ảo trở thành template. Người dùng cũng có thể backup file XML và tiến hành "undefine" máy ảo trong libvirt.
- Sử dụng "virt-manager" để thay đổi tên máy ảo, đối với việc backup file XML, hãy chạy câu lệnh: `virsh dumpxml Template_VMname > /root/Template_VMname.xml`
- Để undefine máy ảo, chạy câu lệnh `virsh undefine VMname`

## Hướng dẫn triển khai máy ảo từ template sử dụng phương thức "Clone" trên virt-manager

- Mở virt-manager, chọn máy ảo đã được chuyển đổi thành template, click chuột phải và chọn `Clone`

<img src="http://i.imgur.com/N81Pgvh.png">

- Điền tên và chọn các thông tin liên quan như network, storage... Sau đó click vào `Clone` để tiến hành tạo máy ảo mới.
- Sau khi hoàn tất, máy ảo đã sẵn sàng để sử dụng

<img src="http://i.imgur.com/8yXMAne.png">

- Lưu ý rằng máy ảo khi tạo ra với phương thức "Clone" sẽ hoàn toàn độc lập với template, nó vẫn có thể chạy khi ta bỏ đi template.
