# Các hàm sử dụng trong shell

## 1. In nội dung vào 1 file

Cú pháp:
```
cat <<EOF>> 'file'
('nội dung muốn chèn')
EOF
```

vú dụ: cat <<EOF>> 123.txt
i love you
EOF

sẽ chèn chữ `i love you` vào file 123.txt

**Lưu ý**: Theo quy ước thì EOF không được viết thụt dòng nên bắt buộc phải đặt ở đầu dòng

## 2. Vòng lặp for, do

Cú pháp: `for (( i = x;i <= y; i++ )); do ` 

- Sau `for` muốn thực hiện lệnh thì phải có `do` ở sau

- Kết thúc hàm `do` là `done`
ví dụ:
```
for (( i = 1;i <= 20;i++ )); do
if [[ $i -gt 10 ]];
then
...
done
```


## 3. If, else, then

Cú pháp: 
```
if [[ 'nội dung' ]];
then
'nội dung'
```
- Kết thúc hàm `if` phải là `fi`

Ví dụ: 
```
if [[ $i -gt 10 ]];
then
cat <<EOF>> 123.txt
$i
EOF
fi
```

## 4. Echo

- Sử dụng in ra màn hình

```
echo [options] [string, variables...]
```
Các `option` trong câu lệnh :

|Option|Ý nghĩa|
|------|-------|
|-i|in giá trị vào file|
|-n|Không xuống dòng|
|-e|Cho phép thực hiện các option với dấu \|
|\a|Chuông cảnh báo|
|\b|Xóa đi ký tự bên tay trái|
|\c|Không xuống dòng đồng thời chặn toàn bộ các ký tự bên phải|
|\n|Tạo 1 dòng mới|
|\t|Tạo 1 khoảng cách|

Ví dụ: `echo "1234"`  in ra màn hình 1234

Ví dụ: 

echo -e "If i die \a\t\tyou die\n"
Giá trị trả về :
```
If i die                you die
```

# 5. Command

- Dùng để thực hiện lệnh nào đó

Ví dụ: `command touch abc.txt`  tạo ra file abc.txt

# 6. Read

- Dùng để lấy dữ liệu nhập từ bàn phím và lưu vào biến, lệnh read có thể dùng để đọc file. Nếu file chứa nhiều hơn 1 dòng thì chỉ có dòng thứ nhất được gán cho biến

ví dụ: `read var=$test`  nhập giá trị cho biến `test`

# 7. Sed

- Dùng để tìm và thay thế văn bản trong file:

Cú pháp: `Sed -i -e "nội dung biểu thức" 'file muốn thay thế nội dung'` 
<ul>
  <ul>
    <li>Tùy chọn -e dùng để thay thế trực tiếp trên file.</li>
    <li>Tùy chọn -i dùng để thay đổi nội dung file nhưng lưu ra 1 file mới.</li>
</ul>
  </ul>

Ví dụ 2 :  Ta có file 123.txt với nội dung sau
```
foo 1
fOo 2
```
`sed "s/foo/bar/g 123.txt` 

kết quả sẽ được
```
bar 1
fOo 2
```
- ở đây với tùy chọn /g sẽ phân biệt chữ hoa và chữ thường ta cũng có thể bỏ qua tùy chọn /g tác dụng vẫn như nhau (ở đây là chữ o giữ foo và fOo)
- Nếu thay bằng tùy chọn /i thì sẽ không phân biệt chữ hoa và chữ thường

Ví dụ 2: `sed -i ".xxx" -e "/s/foo/bar/g 123.txt` kết quả thay đổi file 123.txt sẽ lưu vào 1 file mới tạo ra là 123.txt.xxx

## 8. Lệnh đếm thời gian

`stat $abc.txt -c %Y` in ra thời gian sửa đổi của file abc.txt lần cuối cùng tùy chọn %Y là đổi ra giây

`date +%s` in ra thời gian hiện tại tính ra giây
