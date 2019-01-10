# Python PEP8 代码风格指南
官网：https://www.python.org/dev/peps/pep-0008/ </br>
参考：http://www.cnblogs.com/mrchige/p/6379864.html
## 关注点
* 代码排版
* 字符串引号
* 表达式和语句中的空格
* 注释
* 版本注记
* 命名约定
* 公共和内部接口
* 程序编写建议
## 代码排版
### 缩进
每层缩进使用4个空格。</br>
续行要么与圆括号、中括号、花括号这样的被包裹元素保持垂直对齐，要么放在 Python 的隐线（注：应该是相对于def的内部块）内部，或者使用悬挂缩进。</br>
使用悬挂缩进的注意事项：第一行不能有参数，用进一步的缩进来把其他行区分开。</br>
```
# Aligned with opening delimiter.
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# Add 4 spaces (an extra level of indentation) to distinguish arguments from the rest.
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)

# Hanging indents should add a level.
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)
```
