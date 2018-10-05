# các tham số trong option global

### Process management and security:

- ca-base <dir> : Gán thư mục mặc định để tìm nạp chứng chỉ CA SSL và CRLs khi được sử dụng với đường dẫn `ca-file` hoặc `crl-file` 

- chroot <jail dir> : Thay đổi quyền root, tăng mức độ bảo mật khi hệ thống bị xâm nhập, và quyền thao tác hệ thống chỉ có superuser 

- cpu-map [auto:]<process-set>[/<thread-set>] <cpu-set>... : Gán tiến trình với ràng buộc về luồng cpu
```
cpu-map 1-4 0-3   # bind processes 1 to 4 on the first 4 CPUs

cpu-map 1/all 0-3 # bind all threads of the first process on the
                  # first 4 CPUs

cpu-map 1- 0-     # will be replaced by "cpu-map 1-64 0-63"
                  # or "cpu-map 1-32 0-31" depending on the machine's
                  # word size.
```


- crt-base <dir>
