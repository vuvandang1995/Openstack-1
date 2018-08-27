# cấu hình live migrate sử dụng ssh (Tunneling versus direct transport used a encrypted SSH)

Cấu hình live migrate file nova.conf trên 2 compute

- Tại compute và compute2:
 
 Chỉnh sửa file `vi /etc/nova/nova.conf`
 
 `live_migration_uri: "qemu+ssh://nova@%s/system?keyfile=/var/lib/nova/.ssh/id_rsa&no_verify=1"`
 
 - Cấu hình ssh không cần xác thực cho nova không cần mật khẩu trên 2 compute
``` 
usermod -s /bin/bash nova

passwd nova

su nova

ssh-keygen (enter để tạo key)

ssh-copy-id nova@compute2 (đứng tại compute) và ssh-copy-id nova@compute (đứng tại compute2)

```


