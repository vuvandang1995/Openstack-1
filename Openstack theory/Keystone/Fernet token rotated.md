Lệnh sử dụng để rotate key `keystone-manage fernet_rotate`

Khóa chính sẽ có số index cao nhất = 1

khóa tổ chức (staged key) là khóa phục có số index = 0 có chức năng tạo ra khóa mới để trở thành primary key tiếp theo khi roted, và  primary key cũ sẽ chuyển về secondary key, còn secondary key cũ nhất sẽ bị xóa.

**Ví dụ bên dưới sẽ chỉ ra rằng, trước khi rotated sẽ có 2 key 0 và 1. Sau khi rotated sẽ tạo ra key mới chính ra key số 0, và key số 0 cũ sẽ trở thành key 2 và là primary key còn key số 1 ban đầu vẫn dữ nguyên trở thành secondary key. Rotated lần 2 sẽ tạo ra key 0 1 2 3, nhưng cấu hình chỉ cho phép 3 key, nên key số 1 sẽ bị xóa, key số 0 cũ sẽ trở thành số 3 và là primary key, key số 2 cũ chuyển thành primary key. Cách tính số key tối thiểu: `số key tối thiểu = (thời gian hết hạn của token)/(thời gian rotated key) + 2`**

Sau khi chạy lệnh `keystone-manage fernet_setup`

```
$ keystone-manage fernet_setup
2507 INFO keystone.token.providers.fernet.utils [-] [fernet_tokens] key_repository does not appear to exist; attempting to create it
2507 INFO keystone.token.providers.fernet.utils [-] Created a new key: /etc/keystone/fernet-keys/0
2507 INFO keystone.token.providers.fernet.utils [-] Starting key rotation with 1 key files: ['/etc/keystone/fernet-keys/0']
2507 INFO keystone.token.providers.fernet.utils [-] Current primary key is: 0
2507 INFO keystone.token.providers.fernet.utils [-] Next primary key will be: 1
2507 INFO keystone.token.providers.fernet.utils [-] Promoted key 0 to be the primary: 1
2507 INFO keystone.token.providers.fernet.utils [-] Created a new key: /etc/keystone/fernet-keys/0
2507 INFO keystone.token.providers.fernet.utils [-] Excess keys to purge: []
$ ls /etc/keystone/fernet-keys/
0  1
```

Sau khi chạy lệnh `keystone-manage fernet_rotate`

```
$ keystone-manage fernet_rotate
2528 INFO keystone.token.providers.fernet.utils [-] Starting key rotation with 2 key files: ['/etc/keystone/fernet-keys/0', '/etc/keystone/fernet-keys/1']
2528 INFO keystone.token.providers.fernet.utils [-] Current primary key is: 1
2528 INFO keystone.token.providers.fernet.utils [-] Next primary key will be: 2
2528 INFO keystone.token.providers.fernet.utils [-] Promoted key 0 to be the primary: 2
2528 INFO keystone.token.providers.fernet.utils [-] Created a new key: /etc/keystone/fernet-keys/0
2528 INFO keystone.token.providers.fernet.utils [-] Excess keys to purge: []
$ ls /etc/keystone/fernet-keys/
0  1  2
```


Nếu tiếp tục chạy lệnh `keystone-manage fernet_rotate`
```
$ keystone-manage fernet_rotate
2698 INFO keystone.token.providers.fernet.utils [-] Starting key rotation with 3 key files: ['/etc/keystone/fernet-keys/0', '/etc/keystone/fernet-keys/1', '/etc/keystone/fernet-keys/2']
2698 INFO keystone.token.providers.fernet.utils [-] Current primary key is: 2
2698 INFO keystone.token.providers.fernet.utils [-] Next primary key will be: 3
2698 INFO keystone.token.providers.fernet.utils [-] Promoted key 0 to be the primary: 3
2698 INFO keystone.token.providers.fernet.utils [-] Created a new key: /etc/keystone/fernet-keys/0
2698 INFO keystone.token.providers.fernet.utils [-] Excess keys to purge: [1]
$ ls /etc/keystone/fernet-keys/
0  2  3
```

- key 1 bị xóa, mọi thứ được mã hóa bằng key 1 sẽ không được xác minh vì khóa được sử dụng để mã hóa thông tin đó đã bị xóa, nguyên nhân key 1 bị xóa là do cấu hình hiện tại cho phép mặc định là 3 khóa hoạt động ta có thể cấu hình nhiều khóa hoạt động bằng việc sửa `max_active_keys` ở mục `fernet_tokens` trong file `/etc/keystone/keystone.conf` nhiều lên 

```
[fernet_tokens]

#
# From keystone
#

# Directory containing Fernet token keys. (string value)
#key_repository = /etc/keystone/fernet-keys/

# This controls how many keys are held in rotation by keystone-manage
# fernet_rotate before they are discarded. The default value of 3 means that
# keystone will maintain one staged key, one primary key, and one secondary
# key. Increasing this value means that additional secondary keys will be kept
# in the rotation. (integer value)
# max_active_keys = 3
```

Ví dụ: nếu định xoay vòng khóa chính sau mỗi 30 phút và mỗi mã thông báo Keystone có tuổi thọ là sáu giờ, thì max_active_keys phải được đặt thành 12 (30x12=360) hoặc nhiều hơn.

<img src="http://i.imgur.com/kXZGhUZ.png">
