---
layout: post
title:  "安装virtualenv和virtualenvwrapper"
date:   2017-07-24 10:15:12 +0800
description: 'virtualenv和virtualenvwrapper的安装的一些命令'
tag: Python
---
# 安装virtualenv和virtualenvwrapper

### virtualenv

- 安装:

```
pip install virtualenv
```

如果提示权限问题,使用

```
sudo pip install virtualenv
```

# virtualenvwrapper

我一般都是用virtualenvwrapper进行管理,关于virtualenv的操作,百度有很多

- 安装:

```
pip install virtualenvwrapper
```

- 如果有权限问题,同virtualenv

- 创建一个目录,用于存放所有的virtualenv

```
mkdir PythonVirtualEnv
```

我使用的终端是`iterm2`

在.zshrc文件中添加一些配置:

```
cd ~/.zshrc
export WORKON_HOME=$HOME/PythonVirtualEnv
source /usr/local/bin/virtualenvwrapper.sh
```

- 使用virtualenvwrapper

virtualenvwrapper的部分命令如下:

```
1. 列出虚拟环境列表: workon 或者 lsvirtualenv
2. 新建虚拟环境: mkvirtualenv [虚拟环境名称]
3. 新建一个python3的虚拟环境: mkvirtualenv -p python3 [虚拟环境名称]
4. 启动/切换虚拟环境: workon [虚拟环境名称]
5. 进入当前环境: cdvirtualenv
6. 查看环境里安装了那些包: lssitepackages
7. 进入当前环境的site-paceages: cdsitepackages
8. 进入当前环境的site-paceages的某个包(比如pip): cdsitepackages pip
9. 复制虚拟环境: cpvirtualenv env1 env3
10. 删除虚拟环境: rmvirtualenv [虚拟环境名称]
11. 离开虚拟环境: deactivate
```


