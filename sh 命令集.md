1. if
```sh
if [ ] ;then
  echo "if"
else
  echo "else"
fi
```
2. 表达式语句赋值
  - `$()`：`str=$(grep "substr" ./test.txt)` 得到包含 `substr` 的行
  - 反引号
3. 字符串拼接
```
str=a
str="$str,bb"
echo "$str"  # a,bb
 ```
4. 判断字符串是否为 null
  -  ```if [ "$str" == '' ] ;then```
  -  ```if [ -n "$str" ] ;then```
5. 字符串是否包含子字符串
  ```sh
 ## grep
  str=abc
  echo "$str" | grep "abcd"
  ```
5.  source: 在当前bash环境下读取并执行FileName中的命令。该filename文件可以无"执行权限"
- `mkdir -p`：创建目录及此目录下的目录
- `sed -i`: 直接对文本文件进行操作的
> sed -i 's/原字符串/新字符串/' /home/1.txt
