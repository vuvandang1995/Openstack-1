# Quá trình live migrate trên compute

- User sẽ gửi yêu cầu migrate 1 instance đến API

- API sẽ các thực instance, user, admin policies...

- Sau khi API xác thực thành công sẽ truy cập vào database để kiếm tra instance đó chắc chắn tồn tại và lấy thông tin của instance

- Sau khi lấy thông tin của instance API sẽ chuyển yếu cầu live migrate instance đến Scheduler

- Scheduler sẽ kiểm tra các compute target và tìm ra 1 compute phù hợp đáp ứng các tiêu chí để migrate ( Nếu không có host nào đáp ứng được yêu cầu thì Scheduler sẽ hủy tiến trình migrate)

- Sau khi tìm được host phù hợp Scheduler sẽ gửi yêu cầu migrate đến compute source

- Compute source sẽ bắt đầu thực hiện migrate đến compute target đã được chọn thông qua libvirt URI

- Compute source chấm dứt kết nối tới instance volumes

- Compute source giải phóng security groups khỏi instance

- Compute source chuyển thông tin network cho compute target

- Sau khi Compute target nhận được thông tin về network của instance, Compute target sẽ gửi yêu cầu setup network cho instance tới neutron và neutron gửi thông tin bản ghi về thông tin network cho compute target (kết nối network tới compute target được thiết lập để compute target và compute source chuyển dữ liệu các thông số còn lại từ compute source sang compute target)

- Compute target setup volumes cho instance

- Compute target cập nhật hoàn thành bản ghi network cho instance

- Compute target define các thông số đã cập nhật vào instance

- Compute target yêu cầu neutron setup thông số network của máy ảo như trong bản ghi đã hoàn thành

- Sau đó Compute target update các thông tin trạng thái (active) cho instance và hủy kết nối tới compute source

- Compute Source hủy bỏ source instance đã migrate

- Compute Source gửi yêu cầu tới neutron hủy bỏ tất cả các network trên instance đã migrate, neutron sẽ hủy bỏ các network liên quan đến instance đã migrate của Compute source

- Quá trình migrate hoàn thành
