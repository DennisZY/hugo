---
title: "Libevent初步"
date: 2020-07-09T11:14:54+08:00
draft: false
tags: 
- cpp
- 网络
categories: 
- 学习笔记
---

# 环境配置

OS: Ubuntu 18.04.4 LTS x86_64
Kernel: 4.15.0-88-generic

本来是用manjaro做开发server的，环境几乎不用配置，但是感觉和现实不太契合，所以还是在云上找个机来做server了，centos的仓库太老了，g++是4.8.5的，不想自己配cpp环境，然后改成ubuntu了。

## openssl

``` sh
sudo apt-get install openssl
sudo apt-get install libssl-dev
```

## libenvent

``` sh
wget -c https://github.com/libevent/libevent/releases/download/release-2.1.12-stable/libevent-2.1.12-stable.tar.gz
./configure --prefix=/usr/local/libevent\
make
sudo make install
```

