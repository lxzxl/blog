---
title: "面试知识点总结 (2)"
date: 2018-03-31T15:00:00+08:00
keywords: ["面试", "javascript", "原型", "原型链", "继承"]
description: "总结面试中常问的 JavaScript 知识点：关于原型和继承"
tags: ["javascript"]
categories: ["javascript"]
author: "lxzxl"
---

总结面试中常问的 JavaScript 知识点之 - 原型和继承

# 一切皆对象

原始类型: string, number, boolean, undefined, null

js类型可分为两类：值类型和引用类型

值类型: string, number, boolean, undefined

引用类型: null, 对象, 数组, 函数, 以及通过构造函数生成的string, number, boolean

# 原型

每个函数function都有一个prototype, 即原型

每个对象都有一个`__proto__`, 指向创建该对象的函数(构造函数)的prototype

构造函数原型的\__proto__指向Object.prototype

Object.prototype.\__proto__指向null

构造函数的\__proto__指向Function.prototype,因为都是new Function(…args)生成的

Function.prototype也是对象，其\__proto__指向Object.prototype