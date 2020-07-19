---
layout:     post
title:      "利用GitHub Actions实现自动签到"
subtitle:   "GitHub Actions初学笔记"
date:       2020-07-19
author:     "Poplar"
header-img: "img/post-bg-2015.jpg"
tags:
    - GitHub Actions
    - Hao4K
---

> 这篇文章还未完善，随着学习的进行可能会发生变化。

## GitHub Actions介绍

[Github Actions](https://github.com/features/actions)是Github的持续集成服务，简单讲就是利用仓库中的代码实现自动化流程，功能强大。由于水平有限，需要深入了解可以看[阮一峰的文章](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)以及[GitHub Actions官方文档](https://docs.github.com/cn/actions)。

## 实例：利用GitHub Actions实现Hao4K网站的自动签到

据说[hao4k](https://www.hao4k.cn/)影音论坛下载时需要积分，而积分可以由签到获取。在尝试了一段时间的手动签到后，萌生了自动实现的想法。正好学习了一点python基础，而且利用GitHub Actions可以实现自动运行。经过2个小时的折腾，初步实现了这个功能，过程记录如下。

### Python爬虫实现自动签到

由于目标网站比较友好，几乎没有任何反爬虫设置，登陆过程也只需要用户名和密码，那么Python爬虫的实现就比较简单了，要点是利用requests.Session()实例实现登陆状态的保持，代码举例如下：

	```python
	import requests
	
	login_url = 'hao4k.cn'
	signin_url = 'hao4k.cn/signin'
	form_data = {
		'username': username
		'passwors': password
		}
	def run(form_data):
		s = requests.Session()
		login_resp = s.post(login_url)
		if login_resp == 200:
			signin_resp = s.get(signin_url)
			print(signin_resp.status_code)
	```

### GitHub Actions设置

点击仓库上面Actions标签，按照指引生成一个workflow，位于`.github/workflows`目录下。GitHub扫描到该文件夹下yml后缀的文件便会当作一个workflow自动运行。

	name: CI
	on:
	  push:
	    branches: 
	      - master
	  schedule:
	    - cron: '0 23 * * *'
		
	jobs:
	  sign_in:
	    runs-on: ubuntu-latest
	    steps:
	    - uses: actions/checkout@v2
	    - name: Set python
	      uses: actions/setup-python@v2
	      with:
	        python-version: '3.x'
	    - name: Install dependencies
	      run: python -m pip install --upgrade requests
	    - name: Hao4k auto sign in
	      env:
	        HAO4K_USERNAME: ${{ secrets.HAO4K_USERNAME }}
	        HAO4K_PASSWORD: ${{ secrets.HAO4K_PASSWORD }}
	      run: python autosignin.py

（1）name  

`name`字段是workflow的名称，随意定义。如果省略该字段，默认为该workflow的文件名。

（2）on

`on`字段指触发workflow的条件，通常是在某些GitHub常见的事件下，如`push`或`pull_request`事件，也可以是外部事件或者定时触发，详见[官方文档](https://docs.github.com/cn/actions/reference/events-that-trigger-workflows)。这里主要用到的就是定时触发选项。

需要注意的是，必须使用[POSIX cron语法](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html)，指定工作流在特定的UTC时间（国际标准时间，北京时间减去8个小时）运行，并且最短时间间隔是5分钟。

（3）jobs

`jobs`字段是workflow的主体，表示要执行一项或多项任务，这里我们只设置了一个任务，叫做`sign_in`。

- 3.1 jobs.sign_in.runs-on

`runs-on`字段指定所需要运行的虚拟机，它是必填字段，这里指定最新版ubuntu。

- 3.2 jobs.sign_in.steps

`steps`字段指定每个job的步骤，可以包含一个或多个步骤，每个步骤可以包含三个字段：`name`步骤名称、`run`运行的命令、`env`该步骤所需的环境变量。

`env`环境变量的用法可以参考[官方文档](https://docs.github.com/cn/actions/configuring-and-managing-workflows/using-environment-variables)，可以定义步骤环境变量`jobs.<job_id>.steps.env`、作业环境变量`jobs.<job_id>.env`、整个工作流程的环境变量`env`。

这里定义环境变量，主要是为了隐藏用户名和密码等敏感信息，需要在仓库的设置中找到Secrets选项，增加环境变量，Name和Value分别对应变量名和变量的值。

## 使用该脚本进行自动签到

如果你恰好也有这个需求，可以Fork[这个项目](https://github.com/bychen009/hao4k-auto-sign-in)直接使用，注意Fork之后需要在你的仓库Settings下新建环境变量，才能正常运行。


