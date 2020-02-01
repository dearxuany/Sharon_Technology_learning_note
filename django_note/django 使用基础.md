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
参考：</br>
https://www.cnblogs.com/hszstudypy/p/11512244.html</br>
https://www.cnblogs.com/leffss/p/11555556.html</br>
```
# 报错 sqlite 版本太低
    raise ImproperlyConfigured('SQLite 3.8.3 or later is required (found %s).' % Database.sqlite_versi
django.core.exceptions.ImproperlyConfigured: SQLite 3.8.3 or later is required (found 3.7.17).

# 更新后查看虚拟环境中加载的 sqlite3 版本
(.mysite) [sharonli@vmw-dev-code-01 mysite]$ python
Python 3.6.1 (default, Nov  2 2019, 21:01:57) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sqlite3
>>> sqlite3.sqlite_version
'3.27.2'
>>> 
```
重新生成数据库表
```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying sessions.0001_initial... OK

```

#### 内置开发服务器
启动 django 内置轻量级开发服务器，django 会动态加载现有代码更新到服务中无需重启</br>
执行 runserver 后，服务器于终端前台启动，默认端口 8000</br>
注：新增文件必须重启服务器，此服务器无法在生产使用
```
$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
February 01, 2020 - 15:31:03
Django version 3.0.2, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
[01/Feb/2020 15:33:03] "GET / HTTP/1.1" 200 16351
```
