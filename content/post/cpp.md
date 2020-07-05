---
title: "C++基础"
date: 2020-06-11T10:50:08+08:00
draft: false
tags: 
- cpp
categories: 
- 学习笔记
---

# const

const是用于修饰各种类型为常类型，常类型的变量或对象是不能被更新的。

## 使用方法

```cpp
const int a = 100;
```

## 跨文件访问 

非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

## 指针

```cpp
const char * a; //指向const对象的指针或者说指向常量的指针。
char const * a; //同上
char * const a; //指向类型对象的const指针。或者说常指针、const指针。
const char * const a; //指向const对象的const指针。 
```

## 类内const

- const成员函数不能修改数据成员。
- 
- const对象只能访问const成员函数,而非const对象可以访问任意的成员函数,包括const成员函数.