# django 使用基础
## 安装 django
虚拟环境下使用 pip 安装
```
pip install django
```

## 创建 django 项目
### 生成项目初始目录结构
项目名为 mysite
```
django-admin startproject mysite
```
初始目录结构
```
.
└── mysite
    ├── bin
    │   └── venv_activate.sh
    ├── manage.py  # 命令行工具，django-admin 封装, 无需编辑此文件
    └── mysite
        ├── asgi.py
        ├── __init__.py
        ├── settings.py  # 默认配置
        ├── urls.py   # 映射 URL 路径到视图
        └── wsgi.py   # web 网关接口 WSGI

3 directories, 7 files
```
### setting.py 配置 
#### 数据库
数据库配置，用于定义需要在数据库中生成的表，常用于数据库迁移
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
在 SQLite 3 中生成表
```
$ python manage.py migrate
```
sqlite 版本过低需升级</br>
参考：https://www.cnblogs.com/hszstudypy/p/11512244.html
```
# 报错 sqlite 版本太低
    raise ImproperlyConfigured('SQLite 3.8.3 or later is required (found %s).' % Database.sqlite_versi
django.core.exceptions.ImproperlyConfigured: SQLite 3.8.3 or later is required (found 3.7.17).
```
