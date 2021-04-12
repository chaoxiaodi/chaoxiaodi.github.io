---
layout: post
title: "SHELL三剑客系列 - grep"
subtitle: 'linux shell grep 学习使用记录'
author: "chaoxiaodi"
header-style: text
tags:
  - linux
  - shell
  - 三剑客
  - grep
---

### 前言

`grep` 运维日常工作中使用的命令最频繁的命令之一了
`grep` 命令主要用于搜索文件或者标准输出的结果中符合条件的内容

此文主要记录下日常运维工作中常用的 `grep` 命令方便以后使用

### 命令参数说明

    -a   --text   #不要忽略二进制的数据。
    -A<显示行数>   --after-context=<显示行数>   #除了显示符合范本样式的那一列之外，并显示该行之后的内容。
    -b   --byte-offset   #在显示符合样式的那一行之前，标示出该行第一个字符的编号。
    -B<显示行数>   --before-context=<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前的内容。
    -c    --count   #计算符合样式的列数。
    -C<显示行数>    --context=<显示行数>或-<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前后的内容。
    -d <动作>      --directories=<动作>   #当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
    -e<范本样式>  --regexp=<范本样式>   #指定字符串做为查找文件内容的样式。
    -E      --extended-regexp   #将样式为延伸的普通表示法来使用。
    -f<规则文件>  --file=<规则文件>   #指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
    -F   --fixed-regexp   #将样式视为固定字符串的列表。
    -G   --basic-regexp   #将样式视为普通的表示法来使用。
    -h   --no-filename   #在显示符合样式的那一行之前，不标示该行所属的文件名称。
    -H   --with-filename   #在显示符合样式的那一行之前，表示该行所属的文件名称。
    -i    --ignore-case   #忽略字符大小写的差别。
    -l    --file-with-matches   #列出文件内容符合指定的样式的文件名称。
    -L   --files-without-match   #列出文件内容不符合指定的样式的文件名称。
    -n   --line-number   #在显示符合样式的那一行之前，标示出该行的列数编号。
    -q   --quiet或--silent   #不显示任何信息。
    -r   --recursive   #此参数的效果和指定“-d recurse”参数相同。
    -s   --no-messages   #不显示错误信息。
    -v   --revert-match   #显示不包含匹配文本的所有行。
    -V   --version   #显示版本信息。
    -w   --word-regexp   #只显示全字符合的列。
    -x    --line-regexp   #只显示全列符合的列。
    -y   #此参数的效果和指定“-i”参数相同。

#### 日常运维一些常用举例

    ps aux |grep ssh
    查看进程并搜索是否存在ssh
    
    netstat -lntup |grep 80      
    netstat -lntup |grep nginx
    查看是否监听了80 端口 
    查看是否有 nginx 监听了端口
    
    grep -A4 '^bin' passwd
    使用 -A 参数后面跟一个数字 表示匹配到结果后,在输出后面的4行
    常用于搜索一些错误日志，一般错误日志的关键字后面会跟几行报错
    
    grep -B4 '^bin' passwd
    使用 -B 参数后面跟一个数字 表示匹配到结果后,在输出前面4行
    
    grep -B4 '^bin' passwd
    使用 -C 参数后面跟一个数字 表示匹配到结果后,在输出前面4行以及后面4行
    
    grep -c 'nologin' passwd
    显示一个行数，passwd中有多少行包含 nologin,而不会打印出行。
    
    ps aux |grep -q 'ssh'
    q 参数代表不输出任何信息，如果匹配到了内容返回结果为0 否则为非0 
    常用于脚本中进行判断,如值判断文件是否包含某内容不需要结果，或进程是否存在等
    
    grep -s djfakdsjf.txt
    使用 -s 参数可以取消错误的输出
    比如指定的文件不存在又不想显示错误信息可以使用 -s 参数
    
    grep -i playernum t.log
    使用 -i 忽略大小写
    
    grep -n playernum t.log
    使用 -n #在显示符合样式的那一行之前，标示出该行的列数编号。
    
    grep -w test t.log
    使用 -w #只显示全匹配的单词
    比如
    test
    test12
    test23
    test test
    那
    test
    test test 
    会被匹配出来
    
    
    grep -e "abc" -e "xyz" t.log
    使用 -e #可以一次性过滤多个字段
    
    grep "^$" test.txt
    匹配空行
    
    上面一般通过 -v 参数取反，取消显示空行
    grep -v "^$" test.txt
    再进一步过滤注释行 #开头的行
    grep -Ev '^#|^$' test.txt
    
    grep -r "test" .
    在当前目录及所有子目录下的文件中过滤包含 test 的行
    
    

### 后记
此文记录下日常学习以及工作中用到的grep命令
    
如果有任何问题或建议欢迎提出、讨论~




---我是超小弟·一名不误专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏
