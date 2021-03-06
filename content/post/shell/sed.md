---
title: "sed笔记"
date: 2015-12-14T11:18:15+08:00
keywords: ["shell", "linux", "sed"]
description: "sed命令学习笔记"
tags: ["shell", "linux"]
categories: ["shell"]
author: "lxzxl"
---

```bash
sed [选项] [sed命令] 输入文件
sed [选项] [-f sed脚本文件] 输入文件
```

<!--more-->

## sed 选项：

| 选项 | 描述                                                                       |
| ---- | -------------------------------------------------------------------------- |
| -i   | 直接作用源文件，源文件将被修改。                                           |
| -f   | 指定 sed 脚本文件                                                          |
| -n   | 没有的话会每行都打印出来,遇到匹配行会再打印一边.加上该选项,会只打印匹配行. |

## sed 命令:

| 命令                          | 描述                                               |
| ----------------------------- | -------------------------------------------------- |
| a\|在当前行后添加一行或多行   |
| c\|用新文本替换当前行中的文本 |
| d                             | 删除行                                             |
| i\|在当前行之前插入文本       |
| h                             | 把模式空间的内容复制到暂存缓冲区                   |
| H                             | 把模式空间的内容添加到缓冲区                       |
| g                             | 取出暂存缓冲区的内容，将其复制到模式缓冲区         |
| G                             | 取出暂存缓冲区的内容，将其追加到模式缓冲区         |
| l                             | 列出非打印字符                                     |
| p                             | 打印行                                             |
| n                             | 读入下一行输入，并从下一条而不是第一条命令对其处理 |
| q                             | 结束或退出 sed                                     |
| r                             | 从文件中读取输入行                                 |
| !                             | 对所选行以外的行应用所有命令                       |
| s                             | 用一个字符串替换另外一个字符串                     |

## 替换标志：

| 标志 | 描述                           |
| ---- | ------------------------------ |
| g    | 在行内进行全局替换             |
| p    | 打印行                         |
| w    | 将行写入文件                   |
| x    | 交换暂存缓冲区和模式空间的内容 |
| y    | 将字符转换成另外一个字符       |
| $    | 表示最后一行                   |

## 打印：p 命令

`sed '/abc/p' file`：打印 file 中包含 abc 的行。默认情况 sed 把所有行都打印到屏幕，如果某行匹配到模式，则把该行另外再打印一遍
`sed -n '/abc/p' file`：和上面一样，只是去掉了 sed 的默认行为，只会打印匹配的行

## 删除：d 命令

`sed '3,$d' file`：删除从第 3 行到最后一行的内容。
`sed '$d' file`：删除最后一行的内容
`sed '/abc/d'`：删除包含 abc 的行。
`sed '3d' file`：删除第三行的内容

## 替换：s 命令

`sed 's/abc/def/g' file`:把行内的所有 abc 替换成 def，如果没有 g,则只替换行内的第一个 abc
`sed -n 's/abc/def/p' file`:只打印发生替换的那些行
`sed 's/abc/&def/' file`:在所有的 abc 后面添加 def（&表示匹配的内容）
`sed -n 's/abc/def/gp' file`:把所有的 abc 替换成 def，并打印发生替换的那些行
`sed 's#abc#def#g' file 把所有的 abc 替换成 def，跟在替换 s 后面的字符就是查找串和替换串之间的分割字符，本例中试是#

## 指定行的范围：逗号

`sed -n '/abc/,/def/p' file`:打印模式 abc 到 def 的行
`sed -n '5/,/def/p' file`:打印从第五行到包含 def 行之间的行。
`sed /abd/,/def/s/aaa/bbb/g file`:修改从模式 abc 到模式 def 之间的行，把 aaa 替换成 def

## 多重编辑：-e

`sed -n -e '1,3d' -e 's/abc/def/g' file`:删除 1-3 行，然后把其余行的 abc 替换成 def

## 读文件：r 命令

`sed '/abc/r newfile' file`:在包含 abc 的行后读入 newfile 的内容

## 写文件：w 命令

`sed '/abc/w newfile' file`:在包含 abc 的行写入(保存到)newfile

## 追加：a 命令

`sed '/abc/a\def' file`:在包含 abc 的行后新起一行，写入'def'

## 插入：i 命令

`sed '/abc/i\def' file`:在包含 abc 的行前新起一行，写入'def'

## 修改：c 命令

`sed '/abc/c\def' file`:在包含 abc 的行替换成'def'，旧文本被覆盖

## 读取下一行：n 命令

`sed '/abc/{n ; s/aaa/bbb/g;}' file`:读取包含 abc 的行的下一行，替换 aaa 为 bbb

## 花括号的作用：{}

花括号{}中可以放入多个命令，每个命令后面要用分号";"

## 转换：y 命令

`sed 'y/abc/ABC/' file`:将 a 替换成 A，b 替换成 B，c 替换成 C（正则表达式元字符不起作用）

## 退出：q 命令

`sed '/abc/{ s/aaa/bbb/ ;q; }' file`:在某行包含了 abc，把 aaa 替换成 bbb，然后退出 sed，只处理该行。

## 暂存和取用：h H g G

sed 有两个缓冲区，所有命令都工作在 Pattern 区上，其中保留着 sed 刚读入的行。另外一个 Hold 区就是用来临时存放 Pattern 区数据的。在把 Pattern 区上的数据放入 Hold 区之前，Hold 区为空行。
**G** 将一个换行和 Hold 区中的数据追加到 Pattern 区中数据之后
**g** 将 Hold 区的数据覆盖到 Pattern 区中，Pattern 区中源数据全丢弃
**H** 将一个换行符和 Pattern 区中的数据追加到 Hold 区数据之后
**h** 将 Pattern 区中的数据覆盖到 Hold 区，Hold 区中的源数据全丢弃
**x** 交换 Pattern 区和 Hold 区中的数据
`sed -e '/abc/h' -e '$G' file`:包含 abc 的行通过 h 命令保存到暂存缓冲区，在第二条命令汇中，sed 读到最后一行$时，G 命令从暂存缓冲区中读取一行，追加到模式缓冲区的后面。即所有包含 abc 的行的最后一行被复制到文件末尾。
