title: Linux-find手记
date: 2014-04-16 21:40:07
tags:
- linux
- find
---

# 常用方法 #
~~~~~~
find ./ -name "*.log" -type f  # 找到文件后缀为.log的文件, 注意:一定要有引号
find . -perm 755 -print       # 找到权限为755的文件
find ./ -user poe –print # 找到用户为poe的文件
find /apps -group gem –print
find / -mtime -5 –print #在系统根目录下查找更改时间在5日以内的文件
find /var/adm -mtime +3 –print # 在/var/adm目录下查找更改时间在3日以前的文件
find /usr/sam -path "/usr/sam/dir1" -prune -o –print # 在/usr/sam目录下查找不在dir1子目录之内的所有文件
find . -maxdepth 1 -name "*.zip" -print # 只在的文件
find . -perm 755 -exec ls {} \; ##执行命令, 注意;和{}之间的空格
~~~~~~

#`-type`#

查找某一类型的文件，诸如：

b - 块设备文件

d - 目录

c - 字符设备文件

p - 管道文件

l - 符号链接文件

f - 普通文件
~~~~~~
find . ! -type d –print # 在当前目录下查找除目录以外的所有类型的文件 
~~~~~~

# `-ok` #
和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
~~~~~~
#在当前目录中查找所有文件名以.LOG结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示
find . -name "*.conf"  -mtime +5 -ok rm {  } \;

~~~~~~

# 其他有用指令 #
`-follow`：如果find命令遇到符号链接文件，就跟踪至链接所指向的文件

`-depth`：在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找 

`-xdev` 只查找当前目录下的文件 

`-nogroup`: 查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在

# find 与 xargs #

类似于 `-exec` 指令, 但是效率高
~~~~~~
# 查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件
find . -type f -print | xargs file
find ./ -size 0 | xargs rm -f & 删除文件大小为零的文件
~~~~~~
