# Phần này giới thiệu về openstack



OpenStack là một phần mềm mã nguồn mở, một hệ điều hành đám mây điều khiển các kho lưu trữ lớn, tính toán việc lưu trữ và tài nguyên mạng trong toàn bộ trung tâm dữ liệu, tất cả được quản lý thông qua bảng điều khiển cho phép quản trị viên kiểm soát đồng thời trao quyền cho người dùng của họ với tài nguyên cung cấp thông qua giao diện web dùng để triển khai Cloud Computing (bao gồm private cloud và public cloud) 

<img src="https://i.imgur.com/or7Ws0w.png">

- Phía dưới là phần cứng của, đã được ảo hóa để chia sẻ cho ứng dụng, người dùng

- Trên cùng là các ứng dụng của bạn, tức là các phần mềm mà bạn sử dụng

- Và OpenStack là phần ở giữa 2 phần trên, trong OpenStack có các thành phần, module khác nhau nhưng trong hình minh họa các thành phần cơ bản: Dashboard, Compute, Networking, API, Storage …

- Cần hiểu thoáng và nhanh gọn hình này, ở đây OpenStack đã bao trùm cả lên phần ảo hóa, nó chính là trong khối Compute. Do vậy bạn đừng hiểu nhầm sang khía cạnh Ảo hóa

### Cloud Computing có 5 đặc tính, 4 mô hình dịch vụ và 3 mô hình triển khai

**5 đặc tính:**

- Rapid elasticity: Khả năng thu hồi và cấp phát tài nguyên

- Broad network access: Truy nhập qua các chuẩn mạng

- Measured service: Dịch vụ sử dụng đo đếm được, hay là chi trả theo mức độ sử dụng pay as you go

- On-demand self-service: Khả năng tự phục vụ

- Resource pooling: Chia sẻ tài nguyên

**4 mô hình dịch vụ:**

- Public Cloud: Đám mây công cộng (là các dịch vụ trên nền tảng Cloud Computing để cho các cá nhân và tổ chức thuê, họ dùng chung tài nguyên).

- Private Cloud: Đám mây riêng (dùng trong một doanh nghiệp và không chia sẻ với người dùng ngoài doanh nghiệp đó)

- Community Cloud: Đám mây cộng đồng (là các dịch vụ trên nền tảng Cloud computing do các công ty cùng hợp tác xây dựng và cung cấp các dịch vụ cho cộng đồng

- Hybrid Cloud : Là mô hình kết hợp (lai) giữa các mô hình Public Cloud và Private Cloud

**3 mô hình triển khai:**

- Hạ tầng như một dịch vụ (Infrastructure as a Service)

- Nền tảng như một dịch vụ (Platform as a Service)

- Nền tảng như một dịch vụ (Platform as a Service)

### Một vài thông tin tóm tắt về OpenStack:

*Ban đầu, OpenStack được phát triển bởi NASA và Rackspace, và ra phiên bản đầu tiên vào năm 2010. Định hướng của họ từ khi mới bắt đầu là tạo ra một dự án nguồn mở mà mọi người có thể sử dụng hoặc đóng góp. OpenStack dưới chuẩn Apache License 2.0, và vì thế phiên bản đầu tiên đã phát triển rộng rãi trong cộng đồng được hỗ trợ bởi hơn 9000 cộng tác viên trên gần 90 quốc gia, và hơn 150 công ty bao gồm Redhat, Canonical, IBM, AT&T, Cisco, Intel, PayPal, Comcast và một nhiều cái tên khác. Đến nay, OpenStack đã cho ra mắt 15 phiên bản, các bạn có thể xem thêm tại đây*

*OpenStack hoạt động theo hướng mở: (Open) Công khai lộ trình phát triển, (Open) công khai mã nguồn … Stack mang nghĩa là tầng, lớp (ở đây được hiểu là OpenStack được phát triển theo nguyên lí các tầng lớp chồng lên nhau)*

*Tháng 10/2010 Racksapce và NASA công bố phiên bản đầu tiên của OpenStack, có tên là OpenStack Austin, với 2 thành phần chính ( project con) : Compute (tên mã là Nova) và Object Storage (tên mã là Swift)*

*Tên các phiên bản được bắt đầu theo thứ tự A, B, C, D …trong bảng chữ cái*

## 2. Thành phần

OpenStack không phải là một dự án đơn lẻ mà là một nhóm các dự án nguồn mở nhằm mục đích cung cấp các dịch vụ cloud hoàn chỉnh

OpenStack chứa nhiều thành phần, dưới đây là một số thành phần chính:

- OpenStack compute (code-name Nova): là module quản lý và cung cấp máy ảo. Tên phát triển của nó Nova. Nó hỗ trợ nhiều hypervisors gồm KVM, QEMU, LXC, XenServer... Compute là một công cụ mạnh mẽ mà có thể điều khiển toàn bộ các công việc: networking, CPU, storage, memory, tạo, điều khiển và xóa bỏ máy ảo, security, access control. Bạn có thể điều khiển tất cả bằng lệnh hoặc từ giao diện dashboard trên web.

- OpenStack Glance (code-name Glance): Là OpenStack Image Service, quản lý các disk image ảo. Glance hỗ trợ các ảnh Raw, Hyper-V (VHD), VirtualBox (VDI), Qemu (qcow2) và VMWare (VMDK, OVF). Bạn có thể thực hiện: cập nhật thêm các virtual disk images, cấu hình các public và private image và điều khiển việc truy cập vào chúng, và tất nhiên là có thể tạo và xóa chúng.

- Block Storage Service (code-name Cinder): Cinder là một dịch vụ lưu trữ (Block Storage service) cho OpenStack. Nó được thiết kế để người dùng có thể thực hiện việc lưu trữ bởi Nova, việc này được thực hiện bởi `LVM` hoặc các plugin driver cho các nền tảng lưu trữ khác. Cinder ảo hóa việc quản lý các thiết bị block storage và cung cấp cho người dùng cuối một API đáp ứng được nhu cầu tự phục vụ cũng như tiêu thụ các tài nguyên. Self service API được sử dụng để giao tiếp với các dịch vụ Cinder

- Identity Server(code-name Keystone): quản lý xác thực cho user và projects.

- OpenStack Netwok (code-name Neutron): là thành phần quản lý network cho các máy ảo. Cung cấp chức năng network as a service. Đây là hệ thống có các tính chất pluggable, scalable và API-driven.

- OpenStack dashboard (code-name Horizon): cung cấp cho người quản trị cũng như người dùng giao diện đồ họa để truy cập, cung cấp và tự động tài nguyên cloud. Việc thiết kế có thể mở rộng giúp dễ dàng thêm vào các sản phẩm cũng như dịch vụ ngoài như billing, monitoring và các công cụ giám sát khác.
