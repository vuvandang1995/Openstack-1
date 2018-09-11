### 5. Cinder Driver.

- Cinder driver maps các Cinder requests yêu cầu từ dòng lệnh lên external storage platform.
- Có trên 50 loại driver tuy nhiên chúng ta thường sử dụng nhất là LVM (Logical Volume Managerment).
- Để set một volume driver chúng ta dùng tham số `volume_driver` trong file `cinder.conf`

```sh
volume_driver = cinder.volume.drivers.lvm.LVMISCSIDriver
```

- LVM sẽ maps các thiết bị Block storage vật lý trên các thiết bị Block storage ảo cấp cao hơn.
- Cinder Volume được khởi tạo như Logical Volumes bởi LVM.
- Sử dụng giao thức iSCSI để kết nối các volumes tới các compute nodes.
- Không có nhà cung cấp cụ thể.

`Cinder Attach Flow` 

![cinder-attach-flow](/images/cinder-attach-flow.png)

- Cinder gọi Cinder qua APi của cinder, truyền thông tin kết nối.
  - Ví vụ :  Host name, iSCSI initiator name, FC WWPNs
- Cinder-API chuyển thông điệp đến Cinder-volume.
- Sau đó trình kiểm tra lỗi đầu vào sẽ làm việc và gọi đến volume driver.
- Volume Driver sẽ chuẩn bị các yếu tố cần thiết để cho phép kết nối.
  - Ví dụ : Cho phép máy chủ NOVA có thể truy cập vào Volume.
- Volume driver trả về thông tin kết nối, được truyền cho NOVA.
  - Ví dụ : iSCSI iqn and portal, FC WWPN.
- NOVA tạo kết nối đến storage sử dụng thông tin được trả về.
- NOVA chuyển volume device/file tới hypervisor.


### 6. Cinder Status.

|Status|Mô tả|
|------|-----|
|Creating|Volume được tạo ra|
|Available|Volume ở trạng thái sẵn sàng để attach vào một instane|
|Attaching|Volume đang được gắn vào một instane|
|In-use|Volume đã được gắn thành công vào instane|
|Deleting|Volume đã được xóa thành công|
|Error|Đã xảy ra lỗi khi tạo Volume|
|Error deleting|Xảy ra lỗi khi xóa Volume|
|Backing-up|Volume đang được back up|
|Restore_backup|Trạng thái đang restore lại trạng thái khi back up|
|Error_restoring|Có lỗi xảy ra trong quá trình restore|
|Error_extending|Có lỗi xảy ra khi mở rộng Volume|


### Cinder back up.
- Cinder có thể back up một volume.
- Một bản back up là một bản sao lưu được lưu trữ vào ổ đĩa. Bản backup này được lưu trữ vào object storage.
- Backups có thể cho phép phục hồi từ :
 <ul>
  <li>Dữ liệu volume bị hư.</li>
  <li>Storage failure.</li>
  <li>Site failure (Cung cấp giải pháp backup an toàn).</li>
 </ul>

### 7. Advanced Features.

- Snapshot :
  - Một snapshot là một bản sao của một thời điểm của dữ liệu chứa một volume.
  - Một snapshot sẽ cùng tồn tại trên storage backend như một volume đang hoạt động.
- Quota :
  - Admin sẽ đặt giới hạn cho volume, khả năng backup và snapshot tùy thuộc vào chính sách cài đặt.
- Volume transfer :
  - Chuyển một volume từ user này đến một user khác.
- Encryption :
  - Mã hóa được thực hiện bởi NOVA sử dụng dm-crypt , là một hệ thống con minh bạch mã hóa đĩa trong Linux kernel.
- Migration (Admin only):
  - Chuyển dữ liệu từ back-end hiện tại của volume đến một nơi mới.
  - Hai luồng chính phụ thuộc vào việc volume có được gắn vào hay không.
