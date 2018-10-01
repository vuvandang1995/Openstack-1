# Sử dụng để chèÒEn vào cuối của 1 file
```
cat <<EOF>> 'file'
(nội dung muốn chèn)
EOF
```
vú dụ: cat <<EOF>> 123.txt
i love you
EOF

sẽ chèn chữ `i love you` vào file 123.txt

**Lưu ý**: Theo quy ước thì EOF không được viết thụt dòng nên bắt buộc phải đặt ở đầu dòng
