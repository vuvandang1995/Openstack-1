# Live migrate & cold migrate
### Các bước thực hiện.

SSH vào compute2 và thực hiện lệnh để khai báo file /etc/hosts trên controlller1

cat << EOF > /etc/hosts
127.0.0.1   localhost controller1
172.16.68.201 controller1
172.16.68.202 compute1
172.16.68.203 compute2
EOF

SSH vào compute2 và thực hiện lệnh để khai báo file /etc/hosts trên compute1

cat << EOF > /etc/hosts
127.0.0.1   localhost compute1
172.16.68.201 controller1
172.16.68.202 compute1
172.16.68.203 compute2
EOF

SSH vào compute2 và thực hiện lệnh để khai báo file /etc/hosts trên compute2

cat << EOF > /etc/hosts
127.0.0.1   localhost compute2
172.16.68.201 controller1
172.16.68.202 compute1
172.16.68.203 compute2
EOF

### Thực hiện đứng từ các node compute và ssh với tài khoản root sang nhau.


	* Mặc định khi sử dụng packstack để cài OpenStack Queens ta sẽ sử dụng giao thức ssh (có 2 giao thức là ssh hoặc TCP) để thực hiện live migrate (không cần reboot VM khi migrate) và cold migrate (khi migrate thì VMsẽ bị reboot lại). Do vậy ta cần thực hiện một số thao tác phụ để đảm bảo cold migrate và live migrate được. Các bước dưới sẽ hướng dẫn.


#### Đối với live migrate


	* Đối với live migrate trong khi cài OpenStack Queens bằng packstack, ta chỉ cần đứng từ node compute này ssh sang note compute còn lại (ở đây có compute1 và compute2), thực hiện ssh = tài khoản root để quá trình xác thực lần đầu được thực hiện. Sau đó việc live migrate sẽ thực hiện tự động hoàn toàn.


Đứng trên compute1 và thực hiện ssh sang compute2 với tài khoản root của compute2

ssh root@compute2

Màn hình sẽ hỏi yes/no, ta nhập yes và khai báo mật khẩu cho tài khoản root của máy compute2. Tham khảo http://prntscr.com/k0h6h9. ư
Đăng nhập thành công nhớ exit nhé, vì lúc này đang ở trong compute2

Đứng trên compute2 và thực hiện ssh sang compute1 với tài khoản root của compute1

ssh root@compute1

Màn hình sẽ hỏi yes/no, ta nhập yes và khai báo mật khẩu cho tài khoản root của máy compute1. Tham khảo http://prntscr.com/k0h6uc. 
Đăng nhập thành công nhớ exit nhé, vì lúc này đang ở trong compute1

Sau đó có thể tạo máy ảo và live migrate  


#### Đối với cold migrate

Đối với cold migrate cần thực hiện các bước đặt mật khẩu chon tài khoản nova (tài khoản dùng để ssh khi cold migrate và thực thi các lệnh).

- Nguyên lý khi migrate là các flavor sẽ được chuyển từ compute này sang flavor của compute khác do vậy cần ssh từ nova của compute này sang compute khác để thực hiện quá trình migrate thành công. Mặc định trong bản packstack sẽ không cho phép các compute truy cập bằng user nova mà bắt truy cập bằng user nova_migration. Khi đó sẽ có file `/var/lib/nova/.ssh/config` chỉ cho phép truy cập vào nova bằng user **nova_migration**  ta cần xóa nội dung file đó đi, hoặc xem file `/etc/ssh/sshd_config` hoặc `/etc/ssh/ssh_config` ở đoạn cuối ta có thể thấy user sử dụng là **nova_migration** ta có thể xóa đoạn đó đi để bỏ qua xác thực bằng user **nova_migration**

<img src="https://i.imgur.com/A5fvzYa.png">


##### Thực hiện đặt mật khẩu cho tài khoản nova trên compute1

Đặt mật khẩu cho user nova, lưu ý mật khẩu này để sử dụng

usermod -s /bin/bash nova

passwd nova


##### Thực hiện đặt mật khẩu cho tài khoản nova trên compute2

Đặt mật khẩu cho user nova, lưu ý mật khẩu này để sử dụng

usermod -s /bin/bash nova

passwd nova

Tham khảo:  http://prntscr.com/k0h8cr


##### Thực hiện tạo keypair trên compute1

Tạo keypair trên compute1 để sử dụng cho user nova, public key sẽ được copy sang compute2, thực hiện lệnh dưới và ấn mặc định enter để tạo key

ssh-keygen

Kết quả: http://prntscr.com/k0h9ma

##### Thực hiện tạo keypair trên compute2

Tạo keypair trên compute2 để sử dụng cho user nova, public key sẽ được copy sang compute1, thực hiện lệnh dưới và ấn mặc định enter để tạo key

ssh-keygen

Kết quả: http://prntscr.com/k0h9ma



#### Copy public key từ compute1 sang compute2


	* Copy key và nhập mật khẩu của compute2 khi được hỏi.


ssh-copy-id nova@compute2


	* Thử ssh sang compute2 bằng tài khoản nova


ssh nova@compute2

Kết quả như sau là ok: http://prntscr.com/k0hcqw

#### Copy public key từ compute2 sang compute1


	* Copy key và nhập mật khẩu của compute1 khi được hỏi.


ssh-copy-id nova@compute1


	* Thử ssh sang compute1 bằng tài khoản nova


ssh nova@compute1

Kết quả như sau là ok: http://prntscr.com/k0hdav


#### Phân quyền cho file /etc/nova/migration/identity  trên cả compute1 và compute2

chown nova:nova /etc/nova/migration/identity
