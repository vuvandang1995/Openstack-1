# Tìm hiểu SSH

## 1. Giới thiệu SSH.

SSH (secure shell) là một giao thức mã hóa dùng để quản trị và giao tiếp với servers. Khi làm việc với Linux server, bạn sẽ dành phần lớn thời gian của mình vào terminal được kết nối qua SSH.

Có một vài cách để SSH tới SSH server như dùng password hoặc keypair. Trong đó phương thức dùng keypair được cho là có tính bảo mật cao hơn bởi nếu trong quá trình sử dụng mà các gói tin của bạn bị bắt lại, các phiên trao đổi khóa giữa SSH server và client sẽ bị lộ và attacker có thể dùng nó để giải mã dữ liệu. Hơn nữa, việc này cũng tạo điều kiện cho các cuộc tấn công Brute Force mật khẩu.

SSH có ỗ trợ sử dụng cặp khóa Private Key và Public Key được chia sẻ với nhau từ trước. Nghĩa là bạn đã có sẵn Private Key để trao đổi với server mà không cần đến quá trình trao đổi khóa, điều này sẽ hạn chế khả năng bị bắt gói. Hơn nữa cặp khóa này còn có một mật khẩu riêng của nó, gọi là passphrase (hay keyphrase). Mật khẩu này được dùng để mở khóa Private Key (được hỏi khi bạn SSH vào server) và tạo lớp xác thực thứ 2 cho bộ khóa. Nghĩa là:
+ Nếu attacker không có Private Key, việc truy cập vào server gần như không thể, chỉ cần bạn giữ kĩ Private Key.
+ Tuy nhiên trong trường hợp Private Key bị lộ, bạn vẫn khá an toàn vì đối phương không có passphrase thì vẫn chưa thể làm được gì, tuy nhiên đó chỉ là tạm thời. Bạn cần truy cập server thông qua cách trực tiếp hoặc qua VNC của nhà cung cấp nếu đó là một VPS để thay đổi lại bộ khóa.

## 2. Cách thức hoạt động của SSH keys

SSH key pairs là một cặp khóa được mã hóa có thể được dùng để xác thực giữa client và server. Mỗi một cặp khóa sẽ có public key và private key. Private key được giữ ở phía client và phải được bảo mật tuyệt đối. Nếu có được private key, attackers hoàn toàn có thể truy cập vào server. Cũng vì thế nó được mã hóa với passphrase.

Public key có thể được chia sẻ và phân tán rộng rãi bởi nó được dùng để mã hóa các tin nhắn mà chỉ private key mới giải mã được. Public key sẽ được upload lên phía server và được lưu tại thư mục người dùng (~/.ssh/authorized_keys).

Khi có client muốn xác thực bằng SSH keys, server có thể test xem client đó có giữ private key hay không. Nếu client chứng minh được nó có private key thì kết nối có thể được thiết lập.

Hình dưới đây mô tả quá trình xác thực giữa server và client:

<img src="http://i.imgur.com/5scoQCI.png">

## 3. Thực hành lab SSH

Mô hình lab: client-server. Phía server chạy linux, phía client sử dụng windows hoặc linux.

### 3.1 Tạo khóa trên server

**Phía server**

Sử dụng câu lệnh sau để tạo 1 cặp ssh keys.

`ssh-keygen -t rsa`

Keys được tạo ra sẽ được lưu tại thư mục của user tạo keys, ví dụ bạn tạo keys bằng tài khoản root thì keys sẽ được lưu tại `/root/tên-file`

Sau khi gõ lệnh trên, bạn sẽ được yêu cầu nhập vào tên file chứa key và cả passphrase để mã hóa private key. Ví dụ ở đây mình tạo ra 1 cặp key `ducnm37.pub` và `ducnm37` bằng tài khoản root. Keys của mình sẽ được lưu tại `/root/ducnm37` và `/root/ducnm37.pub`.

``` sh
root@meditech:~# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): ducnm37
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ducnm37.
Your public key has been saved in ducnm37.pub.
The key fingerprint is:
46:a6:db:ff:e7:cb:15:7c:a0:f7:ea:a8:9d:9d:74:dc root@meditech
```

<img src="https://i.imgur.com/sKyn0p6.png">

Sau đó tiến hành phân quyền cho cặp key.
Lưu ý: nếu bạn không chỉnh sửa cấu hình ssh thì bạn phải chuyển public keys tới thư mục mặc định (~/.ssh/authorized_keys) thì server mới có thể xác nhận.

```sh
root@meditech:~# mkdir .ssh
root@meditech:~# mv /root/ducnm37.pub /root/.ssh/authorized_keys
root@meditech:~# chmod 600 /root/.ssh/authorized_keys
root@meditech:~# chmod 700 .ssh
```

Tiếp đến, hãy tiến hành cấu hình trong file `/etc/ssh/sshd_config` để khai báo thư mục đặt key cũng như cho phép user root login. Sau khi chỉnh sửa, tiến hành restart lại dịch vụ ssh.

<img src="http://i.imgur.com/X5DCr55.png">

**Phía client**

- Nếu bạn sử dụng Windows để SSH đến, tiến hành copy file private key ra máy và load bằng PuTTY hoặc MobaXterm. Ở đây mình dùng MobaXterm để load private key.

<img src="http://i.imgur.com/OaZyWCE.png">

<img src="http://i.imgur.com/mIZKUzq.png">

Sau khi load key, bạn hãy lưu nó dưới dạng `ppk`

<img src="http://i.imgur.com/30ygCPg.png">

Để ssh vào server. Bạn chỉ cần import private key vào session.

<img src="https://i.imgur.com/IIdP09U.png">

Nếu có passphrase thì bạn sẽ phải nhập passphrase, và thế là xong !


- Nếu bạn sử dụng hệ điều hành Linux, tiến hành copy private key vào và phân quyền cho private key (phải phân quyền là `600` vì mặc định là `644` thì key sẽ không được phép sử dụng)



Sau đó dùng câu lệnh sau để ssh tới server dùng private key đã phân quyền:

`ssh -i ducnm37 root@192.168.100.51`


### 3.1 Tạo khóa trên phía client.

- Nếu bạn dùng Windows, có thể dùng PuTTY hoặc MobaXterm để gen ssh keys. Ở đây mình dùng MobaXterm.

Chọn loại key là RSA và click generate

<img src="http://i.imgur.com/1Ya6uPx.png">

Di chuyển chuột liên tục vào vùng trống để tạo khóa mới

<img src="http://i.imgur.com/LdJdAo3.png">

Copy toàn bộ nội dung trong ô “Public key for pasting into OpenSSH authorized_keys file:” và lưu lại dưới tên authorized_keys rồi gửi lên Server. Đây là Public Key dành riêng cho OpenSSH. Nút “Save public key” sẽ cho một Public Key dạng khác, bạn không cần quan tâm đến nút này.

<img src="http://i.imgur.com/ed9Qmeg.png">

Nhập passphrase và chọn “Save Private key“. Việc tạo bộ khóa hoàn tất.

<img src="http://i.imgur.com/T5QmxEf.png">

Để đăng nhập, bạn mở session mới, nhập địa chỉ của ssh server, chọn phần `Advanced SSH settings` -> `Use private key` rồi chọn tới private key đã save.


- Nếu bạn dùng Linux, tiến hành gen key như bình thường bằng câu lệnh `ssh-keygen -t rsa`.

Sau đó tiến hành đẩy public key (file_name.pub) lên server vào file `~/.ssh/authorized_keys`.

Để tiến hành đăng nhập, bạn sử dụng câu lệnh sau trên phía client `ssh -i ducnm37 root@192.168.100.51` . Trong đó `ducnm37` chính là private key được tạo cùng với public key trước đó.



## 4. Cấu hình ssh chỉ sử dụng key pairs

Để cấu hình ssh chỉ sử dụng key pairs, tiến hành sửa đổi file `/etc/ssh/sshd_config`

``` sh
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```

Bên cạnh đó, bạn cũng có thể cấu hình sửa đổi file lưu public key:

`AuthorizedKeysFile      %h/.ssh/authorized_keys`

## 5. Sử dụng cloud-init để chèn key pair cho máy ảo trên OpenStack

**Giới thiệu tổng quan về cloud-init**

Ngày nay với metadata service, người dùng có thể bắt đầu cấu hình cho server của mình trước khi log in. metadata service là một địa chỉ HTTP nơi mà máy chủ có thể access tới trong quá trình boot. Nó chứa các thông tin cơ bản như địa chỉ ip, host name...Trong quá trình cài đặt ban đầu, các giá trị này được "nhồi" vào trong bởi một chương trình có tên là `cloud-init` giúp cấu hình các dịch vụ cần thiết.

Cloud-config script chính là phương pháp phổ biến nhất được dùng để chèn vào trong quá trình tạo máy ảo. Nó có định dạng `YAML` đơn giản, dễ cấu hình. Một số các chức năng mà script này mang lại:

- Thay đổi password của user root
- Tạo mới user
- Tạo mới password cho user
- Cho phép user có quyền của user root
- Thay đổi port của dịch vụ SSH
- Cho phép SSH bằng user root
- ...

Tìm hiểu thêm về cloud-init [tại đây](http://cloudinit.readthedocs.io/en/latest/topics/examples.html)

**Hướng dẫn sử dụng cloud-init để chèn SSH key giúp user root đăng nhập vào máy ảo chạy Ubuntu 14.04 trên OpenStack.**

- Tạo key pairs bất kì bằng câu lệnh hoặc sử dụng công cụ như MobaXterm, PuTTY
- Đăng nhập và tạo máy ảo trên dashboard
- Sau khi khai báo thông tin cấu hình cho máy ảo, chèn đoạn script sau vào phần `Customization Script`

``` sh
#cloud-config
users:
  - name: root
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCv60WjxoM39LgPDbiW7ne3gu18q0NIVv0RE6rDLNal1quXZ3nqAlANpl5qmhDQ+GS/sOtygSG4/9aiOA4vXO54k1mHWL2irjuB9XbXr00+44vSd2q/vtXdGXhdSMTf4/XK17fjKSG/9y3yD6nml6q9XgQxx9Vf/IkaKdlK0hbC1ds0+8h83PTb9dF3L7hf3Ch/ghvj5++tWJFdFeG+VI7EDuKNA4zL8C5FdYYWFA88YAmM8ndjA5qCjZXIIeZvZ/z9Kpy6DL0QZ8T3NsxRKapEU3nyiIuEAmn8fbnosWcsovw0IS1Hz6HsjYo4bu/gA82LWt3sdRUBZ/7ZsVD3ELip user@example.com
runcmd:
  - sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin no/' /etc/ssh/sshd_config
```

**Lưu ý** : Thay đoạn khóa trên bằng public key của bạn.

- Sau khi tạo xong máy ảo, tiến hành đăng nhập bằng private key đã tạo.

