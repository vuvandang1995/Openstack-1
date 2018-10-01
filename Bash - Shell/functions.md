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

## 
