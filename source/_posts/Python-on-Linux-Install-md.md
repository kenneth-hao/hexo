title: Python on Linux - 安装
date: 2015-06-29 17:36:30
tags: [it, linux, python] 
---

> Demo 中安装版本为 `2.7.10`
 日期: `20150629.0530`

### 安装 sqlite
- `yum install sqlite-devel` 安装新版本的 python 时, python(CentOS)需要重新编译sqlite3 的类库

---

### 安装 Python

- 前往[Python 官方下载页](https://www.python.org/downloads/)
- 在页面中选择一个版本 如: 2.7.10. 在打开后的页面中, 将 `Gzipped source tarball` de 超链地址复制.
- 登录`Linux` 服务器的终端
- `cd /usr/local`
- `wget https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz`
- `tar xvf Python-2.7.10.tgz`
- `cd Python-2.7.10`
- `./configure`
- `make & make install`
- `python --version` `是否显示正常版本号`

---

### 安装 pip

- `wget https://bootstrap.pypa.io/get-pip.py`
- `python get-pip.py`
- `pip -V` `是否显示正常版本号`

