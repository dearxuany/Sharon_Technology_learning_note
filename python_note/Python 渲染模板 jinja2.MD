# Python 渲染模板 jinja2
## 渲染模板 jinja2 简介
### 关于渲染模板
模板在 Python web 开发中广泛使用，它能够有效地将业务逻辑和页面逻辑分离，使得工程师编写出可读性更好、更加容易理解和维护的代码。目前市面上有非常多的 Python 模板系统，其中最知名的是 Jinja2 和 Mako。</br>
* 渲染</br>
模板包含的是一个响应文本的文件，其中包含用占位变量表示的动态部分，其具体值只在请求的上下文中才能知道。使用真实的值替换变量，再返回最终得到的响应字符串，此过程被称为“渲染”。</br>
### 关于 jinja2
* 相对于 Template, Jinja2 更加灵活，它提供了控制结构、表达式和继承等
* 相对于 Mako, Jinja2 提供了仅有的控制结构，不允许在模板中编写太多的业务逻辑
* 相对于 Django 模板 Jinja2 的性能更好
* Jinja2 可读性好</br>

Jinja2 是 Flask 的一个依赖，如果已经安装了Flask, Jinja2 也会随之安装。当然，也可以单独安装 Jinja2 。

## Jinja2 语法
三种语法块
```
控制结构｛% ctrl %｝
变量取值 {{ variable }}
注释 {# comment #}
```
### Jinja2 变量
Jinja2 模板中使用的{{}}语法表示一个变量，它是一种特殊的占位符，告诉模板引这个位置的值在渲染模板时获取。</br>
Jinja2 识别所有的 Python 数据类型，甚至是一些复杂的类型， 如列表、字典和对象等。</br>

```
#! /bin/usr/env python3

import jinja2
import os
from monitor_data_collect import *

# tpl_path 指的是写有格式的html文件，**kwargs 指的是待渲染的多个输入
def render(tpl_path,**kwargs):
    path,filename = os.path.split(tpl_path)
    return jinja2.Environment(loader=jinja2.FileSystemLoader(path or './')
                             ).get_template(filename).render(**kwargs)


if __name__ == '__main__':
    data = collect_monitor_data()
    print(render('monitor.html',**data))
~                                         
```
monitor.html
```
<html>
    <head><title>linux 系统性能监控</title>
    <body>
        <table border='1'>
            <tr><td>收集时间</td><td>{{collect_time}}</td><tr>
            <tr><td>主机名</td><td>{{host_name}}</td><tr>
            <tr><td>开机时间</td><td>{{boot_time}}</td><tr>
            <tr><td>CPU个数</td><td>{{cpu_count}}</td><tr>
            <tr><td>CPU利用率</td><td>{{cpu_percent}}</td><tr>
            <tr><td>内存总量</td><td>{{mem_total}}</td><tr>
            <tr><td>内存利用率</td><td>{{mem_percent}}</td><tr>
            <tr><td>内存已用空间</td><td>{{men_used}}</td><tr>
            <tr><td>内存可用空间</td><td>{{mem_available}}</td><tr>
            <tr><td>磁盘总量</td><td>{{disk_total}}</td><tr>
            <tr><td>磁盘利用率</td><td>{{disk_percent}}</td><tr>
            <tr><td>磁盘已用空间</td><td>{{disk_used}}</td><tr>
            <tr><td>磁盘可用空间</td><td>{{disk_available}}</td><tr>
        </table>
    </body>
</html>
```
输出结果
```
$ python3 format.py
<html>
    <head><title>linux 系统性能监控</title>
    <body>
        <table border='1'>
            <tr><td>收集时间</td><td>2019-01-17 02:27:01</td><tr>
            <tr><td>主机名</td><td>centOSlearning.SharonLi</td><tr>
            <tr><td>开机时间</td><td>2019-01-17 01:17:51</td><tr>
            <tr><td>CPU个数</td><td>1</td><tr>
            <tr><td>CPU利用率</td><td>26.2</td><tr>
            <tr><td>内存总量</td><td>1003.60MB</td><tr>
            <tr><td>内存利用率</td><td>82.6</td><tr>
            <tr><td>内存已用空间</td><td>828.97MB</td><tr>
            <tr><td>内存可用空间</td><td>174.45MB</td><tr>
            <tr><td>磁盘总量</td><td>23.99GB</td><tr>
            <tr><td>磁盘利用率</td><td>46.6</td><tr>
            <tr><td>磁盘已用空间</td><td>11.18GB</td><tr>
            <tr><td>磁盘可用空间</td><td>12.81GB</td><tr>
        </table>
    </body>
</html>




```
