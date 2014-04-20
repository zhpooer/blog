title: linux-grep手记
date: 2014-04-18 08:01:19
tags:
- grep
- linux
---
# 常用方法 #
~~~~~~
ls -l | grep '^a' # 通过管道过滤ls -l输出的内容，只显示以a开头的行

grep 'test' d* # 显示所有以d开头的文件中包含test的行

grep 'test' aa bb cc # 显示在aa，bb，cc文件中匹配test的行

grep -i pattern files #不区分大小写地搜索。默认情况区分大小写

grep -l pattern files # 只列出匹配的文件名

grep -L pattern files # 列出不匹配的文件名

grep -w pattern files # 只匹配整个单词，而不是字符串的一部分(如匹配‘magic’，而不是‘magical’)

grep -C number pattern files # 匹配的上下文分别显示[number]行
grep -A number pattern files # 匹配的上下文显示后面的[number]行
grep -B number pattern files # 匹配的上下文显示前面的[number]行

grep pattern1 \| pattern2 files # 显示匹配 pattern1 或 pattern2 的行

grep pattern1 files | grep pattern2 # 显示既匹配 pattern1 又匹配 pattern2 的行
~~~~~~

# 正则表达式与grep #
详细正则表达式介绍,[请点击我](/2014/04/15/java正则学习/).
不过在使用grep时,有些符号需要转义如 `{}<>`
~~~~~~
grep "a\{3, 5\}"   # a出现3到5次
grep "\<"          # 查找左尖括号
~~~~~~

# 常用的选项 #

-?  同时显示匹配行上下的？行，如：grep -2 pattern filename同时显示匹配行的上下2行。

-b，--byte-offset  打印匹配行前面打印该行所在的块号码。

-c,--count   只打印匹配的行数，不显示匹配的内容。

-f File，--file=File   从文件中提取模板。空文件中包含0个模板，所以什么都不匹配。

-h，--no-filename   当搜索多个文件时，不显示匹配文件名前缀。

-i，--ignore-case   忽略大小写差别。

-q，--quiet   取消显示，只返回退出状态。0则表示找到了匹配的行。

-n，--line-number   在匹配的行前面打印行号。

-s，--silent  不显示关于不存在或者无法读取文件的错误信息。

-v，--revert-match  反检索，只显示不匹配的行。
