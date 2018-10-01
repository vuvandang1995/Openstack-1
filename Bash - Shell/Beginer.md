### Tìm hiểu Lập trình shell linux

### 1. Shell script 

Shell là chương trình giao tiếp với người dùng. Có nghĩa là shell chấp nhận các lệnh từ bạn (keyboard) và thực thi nó. Nhưng nếu bạn muốn sử dụng nhiều lệnh chỉ bằng một lệnh, thì bạn có thể lưu chuỗi lệnh vào text file và bảo shell thực thi text file này thay vì nhập vào các lệnh. Điều này gọi là shell script.

Định nghĩa: Shell script là một chuỗi các lệnh được viết trong plain text file. Shell script thì giống như batch file trong MS-DOS nhưng mạnh hơn.

Tại sao phải viết shell script:

•	Shell script có thể nhận input từ user, file hoặc output từ màn hình.

•	Tiện lợi để tạo nhóm lệnh riêng.

•	Tiết kiệm thời gian.

•	Tự động làm một vài công việc thường xuyên.

Hướng dẫn tạo và thực thi chương trình shell:

```
#!/bin/bash
echo "hello world"
```
Dòng đầu tiên chúng ta luôn đặt #!/bin/bash, đây là cú pháp bắt buộc. Sau # được hiểu là comment, chú thích của các đoạn mã.

Sau đó, để script có thể thực thi ta phải cấp quyền cho nó.

chmod +x  test.sh

### Biến trong shell

Trong Linux shell có 2 loại biến:

* Trong Linux shell có 2 loại biến:

+ Tạo ra và quản lý bởi Linux.

+ Tên biến là CHỮ HOA

Biến do người dùng định nghĩa

- Tạo ra và quản lý bởi người dùng - Tên biến là chữ thường

Lệnh điều kiện:

Một số điều kiện cho if, else khác các bạn có thể tham khảo bảng sau đây:

Lệnh so sánh với số:

<table>
<thead>
<tr>
<th>Cú pháp</th>
<th>Ý nghĩa</th>
</tr>
</thead>
<tbody>
<tr>
<td>n1 -eq n2</td>
<td>Kiểm tra n1 = n2</td>
</tr>
<tr>
<td>n1 -ne n2</td>
<td>Kiểm tra n1 khác n2</td>
</tr>
<tr>
<td>n1 -lt n2</td>
<td>Kiểm tra n1 &lt; n2</td>
</tr>
<tr>
<td>n1 -le n2</td>
<td>Kiểm tra n1 &lt;= n2</td>
</tr>
<tr>
<td>n1 -gt n2</td>
<td>Kiểm tra n1 &gt; n2</td>
</tr>
<tr>
<td>n1 -ge n2</td>
<td>Kiểm tra n1 &gt;= n2</td>
</tr>
</tbody>
</table>

Lệnh so sánh với chuỗi:

<table>
<thead>
<tr>
<th>Cú pháp</th>
<th>Ý nghĩa</th>
</tr>
</thead>
<tbody>
<tr>
<td>s1 = s2</td>
<td>Kiểm tra s1 = s2</td>
</tr>
<tr>
<td>s1 != s2</td>
<td>Kiểm tra s1 khác s2</td>
</tr>
<tr>
<td>-z s1</td>
<td>Kiểm tra s1 có kích thước bằng 0</td>
</tr>
<tr>
<td>-n s1</td>
<td>Kiểm tra s1 có kích thước khác 0</td>
</tr>
<tr>
<td>s1</td>
<td>Kiểm tra s1 khác rỗng</td>
</tr>
</tbody>
</table>

Toán tử kết hợp:

<table>
<thead>
<tr>
<th>Column 1</th>
<th>Column 2</th>
</tr>
</thead>
<tbody>
<tr>
<td>!</td>
<td>Phủ định (not)</td>
</tr>
<tr>
<td>-a</td>
<td>Và (and)</td>
</tr>
<tr>
<td>-o</td>
<td>Hoặc (or)</td>
</tr>
</tbody>
</table>

Lệnh kiểm tra file (nói chung cho cả tệp và thư mục):

<table>
<thead>
<tr>
<th>Cú pháp</th>
<th>Ý nghĩa</th>
</tr>
</thead>
<tbody>
<tr>
<td>-f file</td>
<td>Kiểm tra xem file có phải là tệp hay không</td>
</tr>
<tr>
<td>-d file</td>
<td>Kiểm tra xem file có phải là thư mục hay không</td>
</tr>
<tr>
<td>-r file</td>
<td>Kiểm tra file có đọc (read) được hay không</td>
</tr>
<tr>
<td>-w file</td>
<td>Kiểm tra file có ghi (write) được hay không</td>
</tr>
<tr>
<td>-x file</td>
<td>Kiểm tra file có thực thi (execute) được hay không</td>
</tr>
<tr>
<td>-s file</td>
<td>Kiểm tra file có kích thước lớn hơn 0 hay không</td>
</tr>
<tr>
<td>-e file</td>
<td>Kiểm tra xem file có tồn tại hay không</td>
</tr>
</tbody>
</table>



















