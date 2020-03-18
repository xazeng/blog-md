---
title: 高版本 gcc 解决方案：devtoolset
categories: linux
date: 2020-03-18 09:43:44
tags: [gcc, linux]
---

编译源代码时，经常会遇到gcc版本太低，而Centos源里又没有高版本gcc的情况。  
编译安装高版本gcc，固然是一种方法，但是偶尔遇到低配的机器就会非常麻烦。  
比较容易解决这种情况的办法就是 RedHat 退出的 *scl* (Software Collections)。
它可以根据 devtoolset 一起配合快速统一开发环境。
<!--more-->

安装

    # 1. Install a package with repository for your system:  
    # On CentOS, install package centos-release-scl available in CentOS repository:  
    $ sudo yum install centos-release-scl
    
    # On RHEL, enable RHSCL repository for you system:
    $ sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
    
    # 2. Install the collection:
    $ sudo yum install devtoolset-7
    
    # 3. Start using software collections:
    $ scl enable devtoolset-7 bash

如果不确定版本，可以先预览下

    $ sudo yum list devtoolset-7\*

其它详情请见：<https://www.softwarecollections.org/en/scls/rhscl/devtoolset-7/>