
Các phiên bản vmware: `Vmware Workstation`, `Vmware Server`, `Vmware Vsphere ESX/ESXi.`

**Vmware Workstation** và **Vmware Server** là những chương trình ảo hóa dành cho desktop. Cấu trúc của nó là 1 chương trình ứng dụng ảo hóa,chạy trên nền hệ điều hành Linux hay Windows. VMware Workstation và VMware Server cho phép tạo ra các máy ảo (VPS) một cách dễ dàng, nhanh chóng với mục đích thử nghiệm trên PC.

**Vmware Vsphere** là một nền tảng ảo hóa cho phép xây dựng hạ tầng điện toán đám mây. Nó bao gồm một bộ các ứng dụng ảo hóa dành cho doanh nghiệp, trong đó nền tảng là **ESX/ESXi**.
  
Khác với **VMware Workstation** hay **VMware Server**, **ESX/ESXi** là một “bare-metal” hypervisor, có thể hiểu **ESX/ESXi** như 1 lõi hệ điều hành được cài đặt trực tiếp trên server vật lý. Nó tương tác trực tiếp với các thiết bị phần cứng, quản lý và cho phép chia sẻ toàn bộ tài nguyên của server vật lý đó trong khi VMware Workstation và VMware Server thực hiện những công việc này thông qua hệ điều hành(Linux hay windows).Điều này làm cho **ESX/ESXi** hết sức uyển chuyển trong quản lý tài nguyên, từ đó tạo ra các tính năng đặc biệt thích hợp cho môi trường quản lý doanh nghiệp với các chuẩn công nghiệp.
 
**Vsphere ESX** có 2 phiên bản là `ESX` và `ESXi`. Mặc dù cả 2 đều là các hypervisor của Vmware nhưng chúng có những điểm khác biệt cốt lõi trong kiến trúc.

**ESX** gồm 1 lõi ảo hóa (vmkernel) được trang bị thêm 1 giao diện quản lý Console (Console OS hay COS), các agent quản lý của VMware được thực thi trong COS. Với kiến trúc này, **ESX** cho phép người sử dụng có thể cài đặt các ứng dụng của các bên thứ ba. Tuy nhiên, từ phiên bản VMware 5 trở đi, **ESX** đã bị loại bỏ và nhường chỗ cho **ESXi**.

Với **ESXi**, COS bị loại bỏ, các agent được chạy trực tiếp trên vmkernel. Chỉ có các module về driver và monitor thiết bị phần cứng của bên thứ ba được Vmware xác thực mới có thể hoạt động, điều này làm cho **ESXi** trở thành 1 kiến trúc chặt chẽ, ngăn chặn những đoạn mã độc được thực thi, nâng cao tính bảo mật toàn hệ thống. **ESXi** đòi hỏi ít các bản vá lỗi hơn, bên cạnh đó, với kiến trúc đã được tinh giản, **ESXi** chỉ mất khoảng 150Mb RAM để vận hành so với 2Gb RAM của **ESX**.Từ Vsphere 5, **ESX** đã được loại bỏ, thay vào đó, **ESXi** trở thành 1 hypervisor chính của Vmware

**- Các phiên bản của Vsphere:**
Được thiết kế đặc biệt cho môi trường doanh nghiệp vừa và nhỏ, Vsphere mang đến giải pháp ảo hóa với những đặc điểm về quản lý, tích hợp
và đảm bảo tính liên tục của hoạt động kinh doanh. Các phiên bản của VMware **Vsphere** bao gồm:

<ul>
 <li>Vsphere Standard Edition: giải pháp cơ bản với chi phí phần cứng tiết kiệm.</li>
<li>Vsphere Enterprise Edition: giải pháp mạnh mẽ cho phép người sử dụng có thể tùy biến để tối ưu hóa hạ tầng.</li>
<li>Vsphere Enterprise Plus Edition: giải pháp hoàn hảo, đầy đủ tính năng, cho phép ảo hóa toàn bộ datacenter thành hạ tầng điện toán 
  đám mây.</li>
</ul>

**-Các tính năng nổi bật của Vsphere:**

<ul>
<li>High availability: Đảm bảo tính liên tục của công việc khi một hệ thống có lỗi</li>
<li>Data recovery: BBảo vệ dữ liệu, backup và restore các máy ảo.</li>
<li>Vmotion: Di chuyển nhanh chóng các máy ảo sang một hệ thống khác mà không có downtime</li>
<li>Virtual serial port concentrator: Kết hợp các traffic từ nhiều cổng song song vào 1 giao diện quản lý</li>
<li>Hot add: Cho phép mở rộng tài nguyên CPU, RAM của host mà không cần có downtime</li>
<li>vShield Zones: Cho phép tạo, cấu hình và duy trì các vùng bảo mật riêng biệt</li>
<li>Fault tolerance: Đảm bảo tính liên tục của ứng dụng trong trường hợp server bị lỗi mà không bị mất dữ liệu</li>
<li>Storage APIs for array integration: Cho phép các bên thứ 3 sử dụng các APIs này cho các ứng dụng backup</li>
<li>Storage Vmotion: Di chuyển động các máy ảo sang các phân vùng storage khác mà không có downtime</li>
<li>Distributed Resource scheduler & distributed power manager: Quản lý tập trung tài nguyên các server (host) thành 1 khối và tự động cân bằng tải.</li>
<li>Distributed Switch: quản lý tập trung và theo dõi các kết nối mạng sử dụng cluster-level</li>
<li>I/O control (network and storage): Quản lý, theo dõi và tự động cân bằng tải trên các thiết bị storage và netwwork</li>
<li>Host profile: Tạo các máy ảo thông qua các template</li>
<li>Auto deploy: Tự động triển khai các host với ESXi.</li>
<li>Policy-driven Storage: Lựa chọn storage theo các policy đã định nghĩa sẵn</li>
<li>Storage DRS: Tự động cân bằng tải trên storage</li>
</ul>

**- Các chế độ card mạng trong vmware:**

**`Card bridge`**: Card này cho phép card máy ảo kết nối với card mạng thật của máy để kết nối ra ngoài internet(card ethernet hoặc wireless). Do đó khi sử dụng card mạng này IP của máy ảo sẽ cùng với dải IP của máy thật.

**`Host-only`**: Chỉ có thể giao tiếp với các `card Host-only` trên các máy ảo khác, và các `card host-only` không thể giao tiếp với mạng vật lý mà máy tính thật đang kết nối.

**`NAT`**: Chỉ có thể giao tiếp với các `card NAT` trên các máy ảo khác, card NAT không thể giao tiếp với mạng vật lý mà máy tính thật đang kết nối. Tuy nhiên nhờ cơ chế NAT được tích hợp trong VMWare, máy tính ảo có thể gián tiếp liên lạc với mạng vật lý bên ngoài (lúc này, các máy ảo sẽ kết nối với máy thật qua switch ảo , và máy thật sẽ đóng vai trò NAT server cho các máy ảo).

