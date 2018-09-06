# Neutron workflow khi tạo máy ảo

<img src="https://i.imgur.com/dQO9kf2.png">

- Đầu tiên khi máy ảo nhận được API call để tạo máy ảo, nó sẽ kết nối với network
- Tiếp theo sẽ allocate_network sẽ gọi tới neutron-server
- Tại Neutron-server sẽ tạo port và lưu thời gian tạo vào database
- Tại L2 agent sẽ kiểm tra xem có port nào mới tạo, sau đó vif plugs sẽ kết nối tới br-int và scan liên tục trên hosts để kiểm tra sự thay đổi của host
- Nova tìm kiếm vif_driver (`vif_driver` *là 1 trình điều khiển của nova sử dụng để kết nối hoặc ngắt kết nối các interfaces ảo vào bên trong*) integration bridge và add nó vào TAP interface
- L2 agent không biết gì trên port nên nó sẽ lấy thông tin chi tiết get_device bằng cách gọi tới neutron-server với ID host đang chạy
- Neutron server phát hiện bind_port và sẽ lưu vào database liên kết giữa port ID và host ID
