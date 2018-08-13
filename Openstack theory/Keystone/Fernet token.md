Lệnh sử dụng để rotate key `keystone-manage fernet_rotate`

Khóa chính sẽ có số index cao nhất = 1

khóa phụ (staged key) có số index = 0 nó giống như 1 khóa thứ 2,nó có khả năng giải mã thông tin nhưng là khóa tiếp theo sẽ trở thành khóa chính (điều này rất quan trọng để xác thực token trong các triển khai  multi-node).

