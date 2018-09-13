# Cấu hình cho cinder scheduler


- Cấu hình sử dụng scheduler_driver_filter

```
vi /etc/cinder/cinder.conf


[default]
scheduler_default_filters = DriverFilter
enabled_backends = lvm-1, lvm-2

[lvm-1]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM01
filter_function = "volume.size < 10"

[lvm-2]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM02
filter_function = "volume.size >= 10"

```

- Cấu hình sử dụng GoodnessWeigher

```
vi /etc/cinder/cinder.conf



[default]
scheduler_default_weighers = GoodnessWeigher
enabled_backends = lvm-1, lvm-2

[lvm-1]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM01
goodness_function = "(volume.size < 5) ? 100 : 50"

[lvm-2]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM02
goodness_function = "(volume.size >= 5) ? 100 : 25"
```
Nếu volume size nhỏ hơn 5, lvm1 được đánh 100 điểm, lvm2 được đánh 25 điểm. Nếu volume size lơn hơn 5, lvm2 được đánh 100, lvm1 50.


- Ta có thể kết hợp cả 2

```
vi /etc/cinder/cinder.conf


[default]
scheduler_default_filters = DriverFilter
scheduler_default_weighers = GoodnessWeigher
enabled_backends = lvm-1, lvm-2

[lvm-1]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM01
filter_function = "stats.total_capacity_gb < 500"
goodness_function = "(volume.size < 25) ? 100 : 50"

[lvm-2]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = LVM02
filter_function = "stats.total_capacity_gb >= 500"
goodness_function = "(volume.size >= 25) ? 100 : 75"
```

