---
layout: post
title:  "virtualenv依赖包的生成的导入"
date:   2017-08-03 14:19:32 +0800
description: '在pycharm中生成requirements.txt文件以及使用requirement.txt文件安装依赖包'
tag: Python
---

下文中均以在pycharm中使用为例

#### 生成
pycharm的集成终端环境中, 激活virtualenv, 然后执行如下的命令:

```
pip freeze > requirements.txt
```

命令执行完成后, 会在当前项目根目录下生成一个叫做`requirements.txt`的文件. 这个文件中就是所有的依赖包及其版本号, 格式如下:

```
click==6.7
Flask==0.12.2
itsdangerous==0.24
Jinja2==2.9.6
MarkupSafe==1.0
Werkzeug==0.12.2
```

**注:**

- `==`左边是依赖包名称
- `==`右边是依赖包的版本号
- 如果依赖包可以向后兼容, 可以写成`>=`, 即:

```
Flask>=0.12.2
```

#### 导入

同样先在pycharm的集成终端环境中, 激活virtualenv, 然后执行以下命令:

```
pip install -r requirements.txt
```

等待命令执行结束, 就安装完成了.

