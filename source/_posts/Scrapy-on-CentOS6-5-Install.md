title: CentOS Scrapy - 安装
date: 2015-06-30 12:12:12
tags: [it, linux, python, scrapy]
---


> Demo 中安装版本为 `1.0.0`
 前提 [Python on Linux - 安装](http://kenneth-hao.github.io/2015/06/29/Python-on-Linux-Install-md/)
 
![scrapy 官方图片](http://7xjzby.com1.z0.glb.clouddn.com/blog_scrapy_offical屏幕快照 2015-07-01 18.25.24.png)

### 安装 Twisted

- 前往[Twisted 官方下载页](https://twistedmatrix.com/trac/), 在页面中找到 `Source` 下载地址, 并复制
- 登录 `Linux` 服务器的终端
- `cd /usr/local`
- `wget https://pypi.python.org/packages/source/T/Twisted/Twisted-15.2.1.tar.bz2`
- `tar xvf Twisted-15.2.1.tar.bz2`
- `cd Twisted-15.2.1`
- `python setup.py install`

---

### 安装 lxml

- `yum install libxml2`
- `yum install libxslt-devel`
- `pip install lxml`

---

### 安装 scrapy
- `yum install libffi-devel`
- `pip install scrapy`
- `scrapy version`    `# 是否正常输出`

---

*REF :* [官方文档](http://doc.scrapy.org/en/1.0/index.html)



