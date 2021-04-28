---
layout:     post
title:      Pod升级到1.5.2
subtitle:   
date:       2018-09-26
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Git
---
1. 安装rvm

$ curl -L get.rvm.io | bash -s stable

2. 列出出可用的ruby版本

$ rvm list known

3. 更新最新版ruby,此处安装最新版为2.4.1

$ rvm install 2.4.1

3. 找出目前版本的pod路径

$ which pod

4. 移除现有pod

$ rm -rf /usr/local/bin/pod

5. 安装1.5.2 pod

5.1. 移除旧源:

如果之前是https://rubygems.org/源

$ gem sources --remove https://rubygems.org/

如果之前是https://ruby.taobao.org/源

$ gem sources --remove https://ruby.taobao.org/

5.2 可以先执行一次系统更新操作

$ sudo gem update --system

5.3 切换gem源

$ gem source -a https://gems.ruby-china.com

确认切换成功

$ gem sources -l

5.4 安装指定版本 1.5.2

$ sudo gem install -n /usr/local/bin cocoapods -v 1.5.2

确认安装成功

$ pod --version