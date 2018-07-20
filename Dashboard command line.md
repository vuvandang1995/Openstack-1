# Phần này nói về việc quản lý dashboard bằng dòng lệnh

- Chạy câu lệnh xác thực biến môi trường: `source /root/keystonerc_admin`

- Hiển thị danh sách image: `openstack image list`

- Xem thông tin chi tiết một image: `openstack image show <tên/ID image>`

- Tạo mới image từ file tải về:  
                               `openstack image create --file /tmp/cirros-0.3.4-x86_64-disk.img
                                --disk-format qcow2 \
                                --container-format bare --public cirros-0.3.4-x86_64`

