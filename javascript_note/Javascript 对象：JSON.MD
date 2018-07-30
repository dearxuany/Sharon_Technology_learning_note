# JSON 
JavaScript 对象表示法（JavaScript Object Notation）;</br>
JSON 是轻量级的文本数据交换格式（纯文本）,字符集必须是UTF-8;</br>
JSON 使用 JavaScript 语法来描述数据对象，但是 JSON 仍然独立于语言和平台;</br>
JSON 能够使用内建的 JavaScript eval() 方法进行解析,可使用 AJAX 进行传输;</br>
JSON 解析器和 JSON 库支持许多不同的编程语言;</br>

## JSON文件
JSON 文件的文件类型是 ".json"，JSON 文本的 MIME 类型是 "application/json"。

## JSON语法规则
### JSON数据书写格式 key:value
```
"firstName":"Sharon"
```
等价于javascript中
```
firstName="Sharon"
```

### JSON的值格式
* 数字（整数或浮点数）
* 字符串（必需在双引号""中）
* 逻辑值（true 或 false）
* 序列（在方括号[]中）
* 对象（在花括号{}中）
* null

#### JSON对象
类似于python的字典，用{}栝起，两个item间用逗号，隔开
```
{"firstName":"Sharon","lastName":"Li"}
```
等价于js中
```
firstName = "Sharon"
lastName = "Li"
```

#### JSON数组
类似于python的列表，可用来存放多个字典，中间用逗号分隔。
首先是多个item组成列表
```
[{"firstName":"Sharon","lastName":"Li"},{"firstName":"Tom","lastName":"Tang"},{"firstName":"Jack","lastName":"Tang"}]
```
将上面的列表作为另外一个item的value
```
{
"students":[
{"firstName":"Sharon","lastName":"Li"},
{"firstName":"Tom","lastName":"Tang"},
{"firstName":"Jack","lastName":"Tang"}
]
}
```
通过js语法创建对象
```
var students=[
{"firstName":"Sharon","lastName":"Li"},
{"firstName":"Tom","lastName":"Tang"},
{"firstName":"Jack","lastName":"Tang"}
]；
```

## JSON使用
### 把 JSON 文本转换为 JavaScript 对象
创建包含 JSON 语法的 JavaScript 字符串，注意单引号和连字符：
```
var txt = '{"students":['+
'{"firstName":"Sharon","lastName":"Li"},'+
'{"firstName":"Tom","lastName":"Tang"},'+
'{"firstName":"Jack","lastName":"Tang"}]}';
```
### 用JS中的eval()将 JSON 文本转换为 JavaScript 对象
```
var obj = eval ("(" + txt + ")");
```
### 在网页中使用js对象
```
<html>
<body>
<h2>通过 JSON 字符串来创建对象</h3>
<p>
First Name: <span id="fname"></span><br /> 
Last Name: <span id="lname"></span><br /> 
</p> 
<script type="text/javascript">
var txt = '{"students":['+
'{"firstName":"Sharon","lastName":"Li"},'+
'{"firstName":"Tom","lastName":"Tang"},'+
'{"firstName":"Jack","lastName":"Tang"}]}';

var obj = eval ("(" + txt + ")");

document.getElementById("fname").innerHTML=obj.students[0].firstName 
document.getElementById("lname").innerHTML=obj.students[0].lastName 
</script>
</body>
</html>
```
打开浏览器执行html文件,以下为输出结果
```
通过 JSON 字符串来创建对象
First Name: Sharon
Last Name: Li
```

## js对象转换为JSON
把任何JavaScript对象变成JSON，就是把这个对象序列化成一个JSON格式的字符串。
```
var s = JSON.stringify(obj);
console.log(s);
```
把js对象person转换为json格式
```
<html>
    <head>
        <title>Javascript object to Json</title>
    </head>
    <body>
    <script language="JavaScript">
        var person = {
            name: 'Bob',
            age: 20,
            tags: ['js', 'web', 'mobile'],
            city: 'Beijing',
            hasCar: true,
            zipcode: null
        };
       var s = JSON.stringify(person);

       console.log(s);
    </script>
    </body>
</html>
```
chrome输出
```
{"name":"Bob","age":20,"tags":["js","web","mobile"],"city":"Beijing","hasCar":true,"zipcode":null}
```
也可更改参数，添加格式
```
var s = JSON.stringify(person,null,'  ');
console.log(s);

//输出
{
  "name": "Bob",
  "age": 20,
  "tags": [
    "js",
    "web",
    "mobile"
  ],
  "city": "Beijing",
  "hasCar": true,
  "zipcode": null
}
```




