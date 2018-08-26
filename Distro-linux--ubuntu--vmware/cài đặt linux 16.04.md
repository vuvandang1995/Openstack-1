**Cài đặt Ubuntu 16.04**

**B1: Chọn  ngôn ngữ cho hệ điều hành Ubuntu 16.04**

<img src="https://i.imgur.com/u32ApG8.jpg">
 
**B2: Tiếp tục ta chọn cài đặt Ubuntu**

<img src="https://i.imgur.com/018xLcu.jpg">
 
**B3: Chọn 1 ngôn ngữ cho tiến trình cài đặt** 

<img src="https://i.imgur.com/eXEY8wV.jpg">
 
**B4: Chọn time zone** 

<img src="https://i.imgur.com/vqubIyu.jpg">
 
**B5: Chọn cấu hình thêm về bàn phím(yes)  hoặc  sử dụng bàn phím thủ công mặc định (no)**

<img src="https://i.imgur.com/4aBz2kx.jpg">
 
**B6: Chọn kiểu ngôn ngữ cho bàn phím**

<img src="https://i.imgur.com/Tbwkff5.jpg">
 
**B7: Chọn ngôn ngữ của bàn phím để phù hợp với máy chủ**

<img src="https://i.imgur.com/jGbPk9v.jpg">
 
**B8: Tùy thuộc vào việc bạn cấu hình card mạng hoặc có card mạng hay không sẽ có thông báo tương tự. ở đây hiển thị DHCP không hoạt động, chọn continue để bỏ qua và tiếp tục cài đặt**

<img src="https://i.imgur.com/g2GKVf2.jpg">
 
**B9: Ở đây có 2 tùy chọn đầu là  hệ thống hỏi bạn có muốn thử cài đặt tự động network hoặc cài đặt tự động network với  DHCP lại không, thì do DHCP không hoạt động nên ta sẽ chọn cấu hình network thủ công (tùy chọn thứ 3), còn tùy chọn thứ 4 là không cài đặt network và có thể cài network sau.**

<img src="https://i.imgur.com/raM8GrT.jpg">
 
**B10: Cài đặt network ta bắt đầu nhập địa chỉ ip, subnetmask,gateway,name server hoặc DNS(có thể bỏ chống)**

<img src="https://i.imgur.com/Id2mkxG.jpg">

<img src="https://i.imgur.com/crGPedc.jpg">

<img src="https://i.imgur.com/tj5YT9L.jpg">

<img src="https://i.imgur.com/QYc4ahX.jpg">

**B11: nhập hostname máy chủ và domain(có thể bỏ qua nếu không có)**

<img src="https://i.imgur.com/gKi5BbC.jpg">

<img src="https://i.imgur.com/YVYHfIE.jpg">

**B12: Do hệ điều hành Ubuntu không cho phép truy cập trực tiếp ngay lần đầu bằng tài khoản root( vấn đề bảo mật) nên ta phải tạo 1 tài khoản thường để đăng nhập sau đó mới tạo tài khoản root**

- *Nhập tên người sử dụng*

 <img src="https://i.imgur.com/mbGJlPP.jpg">
 
- *Nhập tên tài khoản*

 <img src="https://i.imgur.com/phkyK1q.jpg">
 
- *Nhập mật khẩu cần tạo*

 <img src="https://i.imgur.com/wAzdGNI.jpg">
 
- *Nhập lại mật khẩu*

<img src="https://i.imgur.com/K4qPODp.jpg">

**B13: Ở đây hệ thông hỏi bạn có muốn mã hóa thư mục home không(phòng trường hợp bạn bị kẻ xấu lấy cắp tài khoản họ sẽ không thể truy cập dữ liệu trong thư mục của bạn) chọn ‘yes’ để mã hóa, ‘no’ để bỏ qua và không mã hóa.**

<img src="https://i.imgur.com/7Y0HKGb.jpg">

**B14:Do time zone bước 4 chọn không có nên ta phải chọn lại time zone cho hệ thống của bạn** 

 <img src="https://i.imgur.com/Ec6gJIz.jpg">
 
**B15:Hệ thống yêu cầu phân vùng cho ổ đĩa của bạn, có 4 tùy chọn**

- *Guided - use entire disk: Phân vùng tự động, tự động fomat và chia ổ đĩa của bạn*

- *Guided - use entire disk and with set up LVM: Phân vùng tự động bằng LVM, có thể cho phép bạn tùy biến không gian ổ đĩa của bạn (thêm hoặc xóa bớt)*

- *Guided - use entire disk and with set encrypted up LVM: Phân vùng ổ cứng bằng LVM và cài đặt mã hóa ổ cứng, khi người dùng muốn thay đổi ổ cứng phải nhập mã được mã hóa*

- *Manual: Phân vùng thủ công. Người sử dụng sẽ phân vùng và định dạng ổ cứng bằng tay( nên chia làm 2 phân vùng:  Phân vùng đầu tiên có định dạng ext4, thư mục root dùng để chứa dữ liệu và hệ điều hành và phân vùng thứ 2 khoảng 1,1 GB có định dạng swap dùng làm Ram ảo cho hệ điều hành)*

<img src="https://i.imgur.com/W6T5k9Y.jpg">

*Thông báo format ổ đĩa của bạn*

 <img src="https://i.imgur.com/PxknxGT.jpg">
 
*Hệ thống hỏi bạn có muốn phân vùng ổ đĩa với các thông số như trên không (chọn yes nếu muốn, chọn no để quay lại)*

 <img src="https://i.imgur.com/sEklVFR.jpg">
 
*Hệ thống thông báo cho bạn dung lượng tối đa (21gb) và tối thiểu(1,9gb) để phân vùng cài đặt các gói*  

 <img src="https://i.imgur.com/urWUB5B.jpg">
 
*Hệ thông hỏi bạn có đồng ý với sự phân vùng này không chọn yes để tiếp tục no để quay lại*

 <img src="https://i.imgur.com/FIEmpOM.jpg">
 
**B16: Cấu hình proxy HTTP không có thì bỏ chống**

 <img src="https://i.imgur.com/dZIH441.jpg">
 
**B17: hệ thống hỏi bạn có muốn cập nhật tự động cho hệ thống của bạn hay không**

**- Có 3 tùy chọn:**

- *No automatic updates: không tự động cập nhật*

- *Install security updates automatically: Cho phép tự động cập nhật*

- *Mange system with Landscape: Quản lý từ xa*

<img src="https://i.imgur.com/a2GGhJ1.jpg">
 
**B18:Hệ thống hỏi bạn có muốn cài đặt ứng dụng hỗ trợ hay không (bấm space để chọn các ứng dụng muốn cài)**

 <img src="https://i.imgur.com/Yp6uLn7.jpg">
 
**B19:Hệ thống hỏi bạn có muốn cài GRUB boot loader không : là 1 ứng dụng để tải nhân hệ điều hành và khởi động hệ thống, ngoài ra nó cho phép người dùng có thể nạp vào nhiều hệ điều hành sử dụng nhiều hệ điều hành trên một ổ cứng.**

 <img src="https://i.imgur.com/DN3XL5n.jpg">
 
**B20: restart lại hệ thống để hoàn tất cài đặt**

 <img src="https://i.imgur.com/raANwO4.jpg">
 
**B21:Ssau khi hệ thống khởi động lên bạn nhập user và password đã tạo vào**

 <img src="https://i.imgur.com/mXtSCTc.jpg">
 
**B22: Bạn dùng lệnh sudo passwd root để đặt mật khẩu cho root và lệnh su root  để chuyển sang tài khoản root**

 <img src="https://i.imgur.com/4CvAbeG.jpg">








