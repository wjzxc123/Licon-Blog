---
title: Linux 常用指令
excerpt: "Linux 常用指令"
date: "2025/4/11 13:59:11"
tag: [Linux]
categories: [Linux]
---

```sh
#列出所有文件及详情
ls -al 
 
#每次删除前确认
rm -i [文件名]
 
#显示删除过程
rm -v [文件名]

#删除当前目录下除了01和02的文件
rm -rf -v !(01|02)

#删除当前目录下所有txt文件
find . -name "*.txt" -exec rm -rf {} \

#判断是否有警告信息-q表示静默通知
grep -q "warning" /var/log/syslog && echo "发现警告信息" 

#创建文件或更新文件时间戳
touch /tmp/test.txt

# 合并 file1 和 file2 到新文件
cat file1.txt file2.txt > combined.txt  

# 合并所有 .log 文件
cat *.log > all_logs.txt 

# 输入内容后，按 Ctrl+D 保存退出
cat > newfile.txt

# 显示带行号的文件内容
cat -n file.txt

# 过滤包含 "error" 的行
cat file.txt | grep "error"   

# 分页查看内容    
cat file.txt | less

# 统计文件行数             
cat file.txt | wc -l

# 错误！会清空 file1 内容
cat file1 > file1

# 显示文件前 10 行
head -n 10 file.txt

#解压到指令的目录
tar -zxvf archive.tar.gz -C /target/directory

# 压缩目录
tar -zcvf archive.tar.gz /path/to/directory

# 添加文件到 tar 压缩包
tar -rvf archive.tar newfile.txt

#用户类型：u（所有者）、g（组）、o（其他）、a（所有用户）
#操作符：+（添加权限）、-（移除权限）、=（直接设置权限）
#权限：r(4)、w(2)、x(1)
#示例：
chmod u+x file.txt
# 所有者：rwx，组和其他：r-x
chmod 755 script.sh

# 将文件所有者改为 alice
sudo chown alice file.txt 

# 同时修改所有者为 alice，组为 dev
sudo chown alice:dev file.txt

# 创建新用户并设置密码
sudo useradd 新用户名 && sudo passwd 新用户名

# 修改用户名
sudo usermod -l new_username old_username

# 赋予用户管理员权限
sudo usermod -aG sudo username

# 锁定（密码字段前加 `!`）
sudo usermod -L username

# 解锁
sudo usermod -U username

# 查看磁盘使用情况
df -h

# 查看目录大小
du -sh /home

# 查看内存使用情况 MB
free -m

# 查看系统信息
uname -a

# 查看网络连接和端口占用
netstat -tulnp

# 复制文件到远程主机
scp /local/path/file.txt user@remote-host:/remote/path/
# 示例：
scp ~/data.zip alice@192.168.1.100:/home/alice/backups/

# 指定端口复制
scp -P 2222 /local/file user@remote-host:/remote/path/

# 指定密钥复制
scp -i ~/.ssh/id_rsa /local/file user@remote-host:/remote/path/

# 限制为 800 Kbps
scp -l 800 /large-file.iso user@remote-host:/downloads/ 

# 本地目录同步,源路径末尾的 / 表示同步目录内容（不包含目录自身），无 / 会同步整个目录。
rsync -av /source/dir/ /backup/dir/

# 压缩传输加速,-e "ssh -p 2222"：指定 SSH 端口
rsync -avz -e "ssh -p 2222" /local/dir/ user@remote-host:/remote/dir/

#断点续传，-P：显示进度，中断后可重新执行命令继续传输
rsync -avP /source/large-file.iso user@remote-host:/target/

# 打印第1列和第3列（默认空格分隔）
awk '{print $1, $3}' file.txt

# 从第二行开始打印第一列
awk 'NR>1 {print $1}' file.txt

# 指定冒号分隔符（如 /etc/passwd）
awk -F':' '{print $1, $6}' /etc/passwd


# 将 stdin 数据作为参数传递给指定命令
command1 | xargs [选项] command2
# 直接处理输入参数
xargs [选项] command2 < input.txt
# 每次传递 2 个参数给命令（例如批量创建目录）
echo "dir1 dir2 dir3 dir4" | xargs -n 2 mkdir
# 等效执行：mkdir dir1 dir2 && mkdir dir3 dir4


# 压缩文件
zip archive.zip file.txt

# systemctl 管理系统服务 ,启动 Nginx 服务
sudo systemctl start nginx

#查看系统日志
journalctl -u nginx

# >：重定向输出（覆盖文件）
echo "hello" > output.txt

# >>：重定向输出（追加到文件）
echo "world" >> output.txt

# |：管道（传递前一个命令的输出）
cat file.txt | grep "error"

# &：后台运行程序
python script.py &

# &&：前一个命令成功后再执行下一个
make && make install

# 查看命令的帮助文档
man [命令]

# 查看用户是否存在
id [用户名]

# 隐藏标准输出和标准错误
#/dev/null：Linux 的“黑洞”设备，写入它的内容会被丢弃
[指令] &>/dev/null

#$? 保存上一条命令的退出码
id root
echo $?

#搜索文件内容
grep [选项] "搜索模式" 文件名

# -i 忽略大小写
grep -i "error" log.txt  # 匹配 Error、ERROR、error 等

# -r 递归搜索
grep -r "error" /path/to/dir

# -E：使用扩展的正则表达式
grep -E "error[0-9]+" log.txt
```