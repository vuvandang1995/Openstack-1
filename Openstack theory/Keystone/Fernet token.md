Lệnh sử dụng để rotate key `keystone-manage fernet_rotate`

Khóa chính sẽ có số index cao nhất = 1

khóa phụ (staged key) có số index = 0 nó giống như 1 khóa thứ 2,nó có khả năng giải mã thông tin, secondary key là khóa tiếp theo sẽ trở thành khóa chính

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
- sẽ sinh ra khóa số "2" và với khóa có số index cao nhất là primary key là file "1", staged key là file "0", sau khi hết 1 khoảng thời gian khóa chính "1" sẽ xuống và khóa secondary "2" sẽ trở thành primary key, và được sử dụng để mã hóa thông tin còn khóa "1" dùng để giải mã. khóa staged key "0" sử dụng sau khi chạy lệnh `keystone-manage fernet_rotate` để tạo ra khóa số "2"

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

- Sẽ sinh ra khóa số "3" và khóa số 1 sẽ bị xóa, mọi thứ được mã hóa bằng khóa 1 sẽ không được xác minh vì khóa được sử dụng để mã hóa thông tin đó đã bị xóa, nguyên nhân khóa 1 bị xóa là do cấu hình hiện tại cho phép mặc định là 3 khóa hoạt động (ta có thể cấu hình nhiều khóa hoạt động bằng việc sửa `max_active_keys` ở mục `fernet_tokens` trong file `/etc/keystone/keystone.conf` nhiều lên 

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
#max_active_keys = 3
```

Ví dụ: nếu định xoay vòng khóa chính sau mỗi 30 phút và mỗi mã thông báo Keystone có tuổi thọ là sáu giờ, thì max_active_keys phải được đặt thành 12 (30x12=360) hoặc nhiều hơn.

