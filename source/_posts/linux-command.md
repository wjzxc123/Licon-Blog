---
title: Linux 常用指令
excerpt: "Linux 常用指令"
date: "2025/4/11 13:59:11"
tag: [Linux]
categories: [Linux]
---

```bash
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
```