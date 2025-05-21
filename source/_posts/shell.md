---
title: shell
excerpt: "shell"
date: 2025-04-30 10:26:14
tag: [shell,Linux]
categories: [Linux]
---
# Shell脚本语法手册（快速上手指南）

## 一、基础概念

### 1. 脚本格式
```bash
#!/bin/bash      # 指定解释器（shebang）
# 注释内容       # 使用#号添加注释
```


### 2. 执行方式
```bash
chmod +x script.sh   # 添加可执行权限
./script.sh          # 执行脚本
```


---

## 二、变量操作

### 1. 变量定义与使用
```bash
name="value"     # 定义变量（无空格）
echo $name       # 使用变量
echo ${name}     # 推荐写法（边界更清晰）
```


### 2. 特殊变量
| 变量 | 说明 |
|------|------|
| `$0` | 脚本名称 |
| `$1~$9` | 第1~9个参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数 |
| `$?` | 上条命令返回值 |
| `$$` | 当前进程ID |

---

## 三、基本语法

### 1. 条件判断
```bash
if [ condition ]; then
    commands
fi

# 示例：
if [ -f "file.txt" ]; then
    echo "文件存在"
fi


#以下为if条件的扩展内容：========================================================
#[[ ... ]]是 增强型条件测试结构
#  支持 && || 操作符
#  支持模式匹配    (* ? [])  通配符
#  支持正则表达式  =~ 操作符
#  自动处理空变量
[[ "$name" == "Alice" ]]    # 精确匹配
[[ "$str" != "error" ]]     # 不等于判断

# 自动处理空变量（无需担心未定义变量）
[[ $undefined_var == "" ]]  # 安全返回 true


filename="image.jpg"
[[ "$filename" == *.jpg ]]  # 通配符匹配（返回 true）

# 匹配复杂模式
[[ "$hostname" == web?? ]]  # web后跟两个字符（如 web01）

email="user@example.com"
[[ "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]] && echo "合法邮箱"

# 捕获匹配组
[[ "2024-07-30" =~ ([0-9]{4})-([0-9]{2})-([0-9]{2}) ]] && echo "年: ${BASH_REMATCH[1]}"

[[ $num -gt 10 ]]    # 大于（需注意变量为数字）
[[ $age -ge 18 ]]    # 大于等于


# 组合条件（无需转义符号）
[[ $count -gt 0 && ($mode == "dev" || $force == "yes") ]]

# 否定条件
[[ ! -e "/lockfile" ]]  # 文件不存在
```




### 2. 分支选择
```bash
case $var in
    pattern1) commands ;;
    pattern2) commands ;;
    *) default commands ;;
esac
```


### 3. 循环结构
#### for循环
```bash
for var in list; do
    commands
done

# C风格
for ((i=0; i<5; i++)); do
    echo $i
done
```


#### while循环
```bash
while [ condition ]; do
    commands
done
```


#### until循环
```bash
until [ condition ]; do
    commands
done
```


---

## 四、常用命令

### 1. 文件测试运算符
| 运算符 | 描述 |
|--------|------|
| `-f file` | 是否为普通文件 |
| `-d dir` | 是否为目录 |
| `-r file` | 是否可读 |
| `-w file` | 是否可写 |
| `-x file` | 是否可执行 |
| `-s file` | 文件大小是否非0 |

### 2. 字符串比较
| 运算符 | 描述 |
|--------|------|
| `=` | 等于 |
| `!=` | 不等于 |
| `-z` | 是否为空字符串 |
| `-n` | 是否非空 |

### 3. 数字比较
| 运算符 | 描述 |
|--------|------|
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-lt` | 小于 |
| `-ge` | 大于等于 |
| `-le` | 小于等于 |

---

## 五、输入输出

### 1. 输出重定向
```bash
> file      # 覆盖写入
>> file     # 追加写入
2> error.log # 错误输出重定向
```


### 2. 输入重定向
```bash
read var    # 读取用户输入
<<EOF       # 多行输入
```


### 3. 命令替换
```bash
result=$(command) # 推荐写法
result=`command`  # 旧式写法
```


---

## 六、函数
```bash
function name() {
    commands
    return value
}

# 调用
name arg1 arg2
```


---

## 七、示例脚本

### 1. 文件备份脚本
```bash
#!/bin/bash

filename=$1
backup_dir="/backup"

if [ -f "$filename" ]; then
    cp "$filename" "$backup_dir/$(basename $filename)_$(date +%Y%m%d)"
    echo "Backup completed!"
else
    echo "File not found: $filename"
    exit 1
fi
```


### 2. 日志分析脚本
```bash
#!/bin/bash

log_file="/var/log/syslog"

grep "ERROR" $log_file | while read line; do
    echo "[Error] $line"
done
```


---

## 八、调试技巧

### 1. 开启调试模式
```bash
bash -x script.sh
```


### 2. 设置set选项
```bash
set -e   # 遇错终止
set -u   # 未定义变量报错
set -o pipefail  # 管道错误处理
```


---

## 九、实用技巧

1. 使用`which`查找命令路径
2. 使用`man command`查看帮助文档
3. 使用`||`和`&&`进行条件执行
4. 使用`trap`捕获信号
5. 使用`source`或`.`加载环境变量

---