```bash
# 解压
# -x 使用解压操作
# -z 使用 gzip
# -f 指定需要被操作的文件
tar -xzf nginx-1.16.1.tar.gz

# 复制
# -r 若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件。
cp -r contrib/vim/* ~/.vim/

# 查看当前进程状态
# -e 显示全部进程
# -f 显示程序间的关系
ps -ef | grep nginx

# 查找字符串
grep nginx
```