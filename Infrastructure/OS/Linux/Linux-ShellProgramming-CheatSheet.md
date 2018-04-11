[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Linux Shell 编程速览手册

```sh
# 遍历并且判断文件是否存在
for file in Data/*.txt; do
    [ -e "$file" ] || continue
    # ... rest of the loop body
done

# 追加内容
for f in *; do
  echo "whatever" > tmpfile
  cat $f >> tmpfile
  mv tmpfile $f
done
```
