---
title: "awk笔记"
date: 2015-12-14T11:18:15+08:00
keywords: ["shell", "linux", "awk"]
description: "awk命令学习笔记"
tags: ["shell", "linux"]
categories: ["shell"]
author: "lxzxl"
---

记录awk的一些常用命令及用法

<!--more-->

## 命令格式:

`awk [-F fild-separator] 'commands' input-file(s)`
**[ -F 域分隔符]**是可选的，因为 awk 使用**空格**作为缺省的域分隔符例如：

```bash
awk -F: '{ print $1 }' /etc/passwd
awk -f awk-script-file input-files(s)
```

**-f** 选项指明在文件 awk_script_file 中的 awk 脚本

## awk 读文件记录的方式:

| 域 1   | 分隔符  | 域 2 | 分隔符 | 域 3 | 分隔符 | 域 4 |
| ------ | ------- | ---- | ------ | ---- | ------ | ---- |
| 记录 1 | P.Bunny | #    | 02/13  | #    | 123    | #    | Yellow\n |
| 记录 2 | J.Troll | #    | 07/13  | #    | 124    | #    | Brown\n |

awk 执行时，其浏览域标记为`$1，$2 ... $n`。这种方法称为域标识。使用这些域标识将更容易对域进行进一步处理。
`$0`，意即**所有域**。

* 打印报告头:
  打印信息头放置在 BEGIN 模式部分，因为打印信息头被界定为一个动作，必须用大括号括起来。
  `awk 'BEGIN {print "Name Belt\n---------------------------"}{print $1"\t",$4}' grade.txt`
* 打印信息尾
  END 语句在所有文本处理动作执行完之后才被执行。END 语句在脚本中的位置放置在主要动作之后。
  `awk 'BEGIN {print "Name\n--------"}{print $1} END {print "end-of-report"}' grade.txt`

## 条件操作符:

`<, >`：小于，大于
`<=, >=`：小于等于，大于等于
`==, !=`：等于，不等于
`~, !~`：匹配正则表达式， 不匹配正则表达式

* 匹配

```bash
awk '{if($4~/Brown/) print $0}' grade.txt
awk '$4 ~ /Brown/' grade.txt
```

匹配记录找到时，如果不特别声明， awk 缺省打印整条记录。**默认是输出$0**

```bash
awk '{if($3=="48") print$0}' grade.txt
awk '$3=="48" {print$0}' grade.txt
awk '$3=="48"' grade.txt
awk '$6 < $7 {print $1}' grade.txt
awk '/[Gg]reen/' grade.txt
awk '$1~/^...a/' grade.txt
```

* `&& AND`:语句两边必须同时匹配为真。
* `|| OR`:语句两边同时或其中一边匹配为真。
* `!`: 非求逆。

```bash
awk '{if ($1=="P.Bunny" && $4=="Yellow") print $0}' grade.txt
awk '$1=="P.Bunny" && $4=="Yellow"' grade.txt
awk '$4=="Yellow" || $4~/Brown/' grade.txt
```

## awk 内置变量:

`ARGC`:命令行参数个数
`ARGV`:命令行参数排列
`ENVIRON`:支持队列中系统环境变量的使用
`FILENAME`:awk 浏览的文件名
`FNR`:浏览文件的记录数
`FS`:设置输入域分隔符，等价于命令行-F 选项
`NF`:浏览记录的域个数
`NR`:已读的记录数
`OFS`:输出域分隔符
`ORS`:输出记录分隔符
`RS`:控制记录分隔符

```bash
awk 'END{print NR}' grade.txt
awk '{print NF,NR,$0} END{print FILENAME}' grade.txt
```

在从文件中抽取信息时，最好首先检查文件中是否有记录。下面的例子只有在文件中至少有一个记录时才查询 Brown 级别记录。

```bash
awk '{if (NR>0 && $4~/Brown/)print $0}' grade.txt
```

## awk 操作符:

`= += *= / = %= ^ =` 赋值操作符
`?` 条件表达操作符
`|| && !` 并、与、非（上一节已讲到）
`~ !~` 匹配操作符，包括匹配和不匹配
`< <= == != >>` 关系操作符
`+ - * / % ^` 算术操作符
`+ + --` 前缀和后缀

```bash
awk '{name=$1;belts=$4;if(belts ~/Yellow/) print name" is belt "belts}' grade.txt
awk 'BEGIN{BASELINE="27"} {if ($6<BASELINE) print $0}' grade.txt
awk 'tot+=$6; END{print "Club student total points :" tot}'
```

如果文件很大，你只想打印结果部分而不是所有记录，在语句的外 面加上括号{}即可。

```bash
awk 'tot+=$6 {}; END{print "Club student total points :" tot}'
awk '{(tot+=$6)}; END{print "Club student total points :" tot}' grade.txt
```

## awk 内置字符串函数：

`r`:正则表达式
`gsub(r,s)`:在整个`$0`中用 s 替代 r
`gsub(r,s,t)`:在整个 t 中用 s 替代 r
`index(s,t)`:返回 s 中字符串 t 的第一位置
`length(s)`:返回 s 长度
`match(s,r)`:测试 s 是否包含匹配 r 的字符串
`split(s,a,fs)`:在 fs 上将 s 分成序列 a
`sprint(fmt,exp)`:返回经 fmt 格式化后的 e x p
`sub(r,s)`:用$0 中最左边最长的子串代替 s
`substr(s,p)`:返回字符串 s 中从 p 开始的后缀部分
`substr(s,p,n)`: 返回字符串 s 中从 p 开始长度为 n 的后缀部分

```bash
awk 'gsub(/4842/,4899){print $0}' grade.txt  #改变学生序号4842到4899
awk '$1=="J.Troll" {print length($1)" "$1}' grade.txt
```

## 特殊键：

`\b`:退格键
`\t`:tab 键
`\f`:走纸换页
`\ddd`:八进制值
`\n`:新行
`\c`:任意其他特殊字符，例如\\为反斜线符号
`\r`:回车键

## awk 输出函数 printf:

`printf([format_control_flg],arg)`

| printf 修饰符 | 作用                               |
| :-----------: | ---------------------------------- |
|    **\-**     | 左对齐                             |
|   **Width**   | 域的步长，用 0 表示 0 步长         |
|   **.prec**   | 最大字符串长度，或小数点右边的位数 |

| printf 格式 | 作用                                |
| :---------: | ----------------------------------- |
|     %c      | ASCII 字符                          |
|     %d      | 整数                                |
|     %e      | 浮点数，科学记数法                  |
|     %f      | 浮点数，例如（123.44）              |
|     %g      | awk 决定使用哪种浮点数转换 e 或者 f |
|     %o      | 八进制数                            |
|     %s      | 字符串                              |
|     %x      | 十六进制数                          |

`next;`：忽略本行，到下一行

## 脚本格式：

```bash
#!/bin/awk -f
#to call:passwd.awk /etc/passwd
#print out the first and seventh fields
BEGIN{
    FS=":"
}
{print $1,"\t",$7}
```
