# Các file cấu hình Glance

file image sẽ được lưu tại `/var/lib/glance/images/`





[1. File glance-api.conf](#api)

[2. File glance-registry.conf](#registry)

[3. File log của glance](#log)

--------

Glance có 2 daemon chạy ẩn đó là glance-api và glance-registry.

<a name="api"></a>
### 1. File glance-api.conf

- Nằm tại thư mục `/etc/glance/glance-api.conf`

- Tại section [database], cấu hình để kết nối tới database:

```
[database]
# ...
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance
```

`GLANCE_DBPASS` là password bạn chọn cho image service database.

- Section [keystone_authtoken] và [paste_deploy] chứa các cấu hình để kết nối tới identity service:

``` 
[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
# ...
flavor = keystone
```

- Section [glance_store] chứa cấu hình về kiểu lưu trữ image cũng như nơi chưa image file.

``` 
[glance_store]
# ...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

- Ở đây có 2 loại stores được sử dụng và mặc định sẽ sử dụng file system. Bạn có thể cấu hình để lưu trữ trên nhiều nơi khác nhau theo cấu trúc:

`filesystem_store_datadirs=PATH:PRIORITY`

Trong đó:

PATH là đường dẫn tới thư mục chứa image và PRIORITY là mức độ ưu tiên.

Ví dụ:

```
filesystem_store_datadirs = /var/glance/store
filesystem_store_datadirs = /var/glance/store1:100
filesystem_store_datadirs = /var/glance/store2:200
```

<a name="registry"></a>
### 2. File glance-registry.conf

- Tương tự như file `glance-api.conf`, file `glance-registry.conf` cũng được lưu tại thư mục `/etc/glance`. Đồng thời nó cũng chứa cấu hình để access database cũng như identity service:

``` 
[database]
# ...
connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance

[keystone_authtoken]
# ...
auth_uri = http://controller:5000
auth_url = http://controller:5000 (authen token)
memcached_servers = controller:11211 (bộ nhớ đệm đảm bảo tính nhất quán khi sử dụng các project)
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = GLANCE_PASS

[paste_deploy]
# ...
flavor = keystone  (bật đường dẫn pipeline:glance-api-keystone)
```

Trên đây chỉ là những cấu hình cơ bản tối thiểu mà người dùng cần thực hiện trong quá trình cài đặt glance

<a name="log"></a>
### 3. File log của glance

- Section [DEFAULT] chứa đường dẫn tới file và thư mục log của glance:

``` 
[DEFAULT]
log_file = /var/log/glance/api.log
log_dir = /var/log/glance
```

- File `glance-api.log` ghi lại lịch sử truy cập api server

- File `glance-registry.log` ghi lại lịch sử liên quan tới registry server.

- Bạn có thể thay đổi file lưu lại log bằng cách chỉnh sửa file cấu hình, sau đó khởi động lại glance-api server bằng câu lệnh:

`glance-control api start glance-api.conf`
