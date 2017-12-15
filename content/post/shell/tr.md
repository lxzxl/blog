---
title: "tr笔记"
date: 2017-12-14T11:18:15+08:00
weight: 70
keywords: ["shell", "linux", "tr"]
description: "sed命令学习笔记"
tags: ["shell", "linux"]
categories: ["shell"]
author: "lxzxl"
---

> tr 用来从标准输入中通过替换或删除操作进行字符转换

tr -c -d -s ["string1_to_translate_from"]["string2_to_translate_to"] input_file

-c 用字符串 1 中字符集的补集替换此字符集，要求字符集为 ASCII。

-d 删除字符串 1 中所有输入字符。

-s 删除所有重复出现字符序列，只保留第一个；即将重复出现字符串压缩为一个字符串。

指定字符串 1 或字符串 2 的内容时，只能使用单字符或字符串范围或列表。

[a-z] a-z 内的字符组成的字符串。

[A-Z] A-Z 内的字符组成的字符串。

[0-9] 数字串。

/octal 一个三位的八进制数，对应有效的 ASCII 字符。

[O*n] 表示字符 O 重复出现指定次数 n。因此[O*2]匹配 OO 的字符串。

大部分 tr 变种支持字符类和速记控制字符。

字符类格式为[:class]，包含数字、希腊字母、空行、小写、大写、ctrl 键、空格、点记符、图形等等。

如果要去除重复字母或将其压缩在一起，使用-s 选项.

tr -s "[a-z]" < opps.txt
