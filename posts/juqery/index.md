# Juqery笔记


<!--more-->
### 一、Jquery对象

#### 1.1 对象类型

##### DOM对象

>HTMLCollection []

##### Jquery包装集对象

>jQuery.fn.init(0)

#### 1.2对象转换

> //通过$(DOM对象）转为Jquery对象
>
>console.log($(document.getElementsByClassName("mainTable")));
>
>//通过  Jquery对象.属性 转为DOM对象
>
> console.log($(".mainTable").context);

### 二、Jquery 选择器

####  2.1、Jquery 基础选择器

|     选择器     |  名称   |                     示例                     |
| :------------: | :-----: | :------------------------------------------: |
|    id选择器    |   #id   |    $("#idtest")  获取id 为 idtest 的元素     |
| 元素名称选择器 | element |          $("div")  获取所有div元素           |
|    类选择器    | .class  | $(".className")获取class="className" 的元素  |
|  选择所有元素  |    *    |           $("*") 选择页面所有元素            |
|   组合选择器   |         | $("#idtest,div,.className") 同时选取多个元素 |

#### 2.2 Jquery 层次选择器

|   选择器   | 名称                 | 示例                                                  |
| :--------: | -------------------- | ----------------------------------------------------- |
| 后代选择器 | ancestor  descendant | $("#parent div") 选择id为parent的元素的所有div元素    |
| 子代选择器 | parent > child       | $("#parent > div")选择id为parent的元素的直接div子元素 |
| 相邻选择器 | prev + next          | $("#parent + div")选择id为parent的元素的下一个div元素 |
| 同辈选择器 | prev ~ sibling       | $("#parent ~ div")选择id为parent的元素之后的div元素   |

#### 2.3 Jquery 表单选择器

|     选择器     |   名称    |                             示例                             |
| :------------: | :-------: | :----------------------------------------------------------: |
|   表单选择器   |  :input   | $(":input")查找所有input元素   <br/> 注意：会匹配所有的input、textarea、select和button元素 |
|  文本框选择器  |   :text   |                  $(":text") 查找所有文本框                   |
|  密码框选择器  | :password |                $(":password") 查找所有密码框                 |
| 单选按钮选择器 |  :radio   |                 $(":radio")查找所有单选按钮                  |
|  复选框选择器  | :checkbox |                $(":checkbox") 查找所有复选框                 |
| 提交按钮选择器 |  :submit  |                 $(":submit")查找所有提交按钮                 |
|  图像域选择器  |  :image   |                  $(":image")查找所有图像域                   |
| 重置按钮选择器 |  :reset   |                 $(":reset")查找所有重置按钮                  |
|   按钮选择器   |  :button  |                  $(":button") 查找所有按钮                   |
|  文件域选择器  |   :file   |                   $(":file")查找所有文件域                   |

### 三、Jquery 操作元素

#### 3.1 、操作元素的属性

**属性的分类**：

1. 固有属性：元素本身有的属性（id、class、style)
2. 返回boolean的属性：如checked,selected,disabled
3. 自定义属性：用户自定义属性

**获取属性的方法**：

​	attr("属性")

​	prop("属性")

>​	$(spreadsheetReport.elem_content).attr("style");

两者之间的区别：

1. 如果是固有属性，attr 和 prop 方法都可以操作

2. 如果是自定义属性，attr 可以操作 ，prop不可以操作

3. 如果是boolean值属性，

   ​		如果设置了属性值，attr 获取返回属性值，prop 获取返回 true

   ​		如果没有设置属性值，attr 获取返回undefined, prop 获取返回false

**设置属性的方法：**

​	attr("属性","属性值")

​	prop("属性","属性值")

**移除属性的方法：**

​	removeAttr("属性名")

**总结：**

==如果属性的类型为boolean，则用prop()方法，否则使用attr()方法==

#### 3.2 、 操作元素的样式

**结合css ,操作元素的样式**

attr("class")                      获取元素的样式名

attr("class","样式名")       设置元素的样式（会覆盖原本的样式）

addClass("样式名")         添加样式（在原本的样式基础上添加样式，如果样式名中属性如background发生冲突，												则按css中定义的顺序，后定义的样式覆盖前定义的样式）

removeClass("样式名")    移除样式

```css
/*css 定义文件*/
.green {
    background:green;
}
.pink {
    background:pink;
}
```

```js
//获取元素的样式名
 $('input').attr("class");
//设置元素的样式
$('input').attr("class",".pink");
//添加样式
//在css文件中pink样式名定义在green样式名后，所有pink会覆盖掉green的背景色
$('input').attr("green");
//移除样式
$('input').removeClass("pink");
```

**操作元素的css样式中的具体内容**

css()        添加具体的样式（添加行内样式）

```js
//获取color属性值
$("div").css("color") 
//设置单个样式
$("div").css("color","red")
//设置多个样式
$("div").css({fontSize:"30px",color:"red"})
```

#### 3.3 、操作元素的内容

**操作表单元素内容**

表单元素：文本框text 、密码框input、单选框radio、复选框checkbox、文本域textarea、下拉框select

​	val()       获取表单元素的值

​	val("值")  设置表单元素的值

**操作非表单元素的内容**

​	html()      		获取元素的内容

​	html("内容")  	设置元素的内容（会覆盖原本的内容）

```js
$(spreadsheetReport.elem_content).html( )
$(spreadsheetReport.elem_content).html("1234")
```

### 四、Jquery 创建、添加、删除、遍历元素

#### 4.1 、 创建元素

```js
//$("元素") ,通过$将元素转为Jquery对象
var p="<p>这是一个p标签</p>"
$(p)
```

#### 4.2 、添加元素

| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| prepend(content)               | 在元素内部开头插入元素或者内容<br/>==content参数可以是字符串、html元素、JQuery对象== |
| $(content).prependTo(selector) | 把content元素或内容加入selector 元素内部开头                 |
| append(content)                | 在元素内部尾部插入元素或内容                                 |
| $(content).appendTo(selector)  | 把content元素或内容加入selector 元素内部结尾                 |
| before(content)                | 在元素前插入元素或内容                                       |
| after(content)                 | 在元素后插入元素或内容                                       |

```js
//  添加元素：内容可以是字符串、html元素、JQuery对象 ，指定元素必须是Jquery对象
/* 1、前追加子元素
	  指定元素.prepend(内容)            在指定元素内部的最前面追加内容
	  $(内容).prependTo(指定元素) 
	2、后追加子元素
	  指定元素.append(内容)             在指定元素内部的最后面追加内容
	  $(内容).appendTo(指定元素) 
	3、前追加同级元素
	  指定元素.before(内容)			  在指定元素相邻的前面位置追加内容
	4、后追加同级元素
	   指定元素.after(内容)			  在指定元素相邻的后面位置追加内容
*/
var p = "<h1>这是一个p标签</h1>";
var p1 = "<h1>这是一个p1标签</h1>";
var c = $(spreadsheetReport.elem_content);
c.before($(p));
c.before($(p1));
```

#### 4.3 、删除元素

| 方法     | 说明                                                 |
| -------- | ---------------------------------------------------- |
| remove() | 删除所选元素或指定的子元素，包括整个标签和内容都删除 |
| empty()  | 清空所有内容，保留标签                               |

#### 4.4 、遍历元素

**each()**

使用方法：

​	$(selector).each(function(index,element)) ;  遍历元素

​		参数 function 为遍历时的回调函数

​		index 为遍历的序列号，从0开始

​		element是当前的元素

```js
//可以不需要index、element参数，直接用this获取当前的子元素 
var c = $(spreadsheetReport.elem_content);
    c.each(function(index,element){
        console.log(index);
        console.log(element);
        console.log(this);
    });
```

### 五 、Jquery 事件

#### 5.1 、预加载事件 ready

当前的DOM结构加载完毕后执行，类似于js的load事件，ready事件可以写多个

**使用语法：**

```js
$(document).ready(function(){
	//要执行的代码
});
简写：
$(function(){
    //
});
```

#### 5.2 、绑定事件 

为被选元素添加一个或多个事件处理程序，并规定事件发生时运行的函数

##### **bind绑定事件**

```js
/*
$(selector).bind(eventType[,evevtData],handler(eventObject));
	selector: 被选元素
	eventType:所需要绑定的事件
    [,evevtData]:传递的参数，格式{名：值；名1：值1;...}
    handler(eventObject) 该事件触发执行的函数
    */
```

```js
绑定单个事件
		$("元素").blind("事件类型",function(){
			//
		});
```

```js
绑定多个事件
    	1、同时为多个事件绑定同一个函数
		$("元素").blind("事件类型1，事件类型2，事件类型3...",function(){
			//
		});
		2、为元素绑定多个事件，并设置对应的函数
		$("元素").blind("事件类型1",function(){
			//
		}).blind("事件类型2",function(){
			//
		}).blind("事件类型3",function(){
			//
		});
		3、为元素绑定多个事件，并设置对应的函数
        $("元素").blind({}
         "事件类型1"：function({}),
         "事件类型2"：function({}),
         "事件类型3"：function({})
         });
```

##### **直接绑定(建议使用)**

```
绑定单个事件
		$("元素").事件名(function(){
			//
		});
绑定多个事件
		$("元素").事件名1(function(){
		
		}).事件名2(function(){
		
		}).事件名3(function(){
		
		});
```

```js
   var c = $(spreadsheetReport.elem_content);
    c.after("<h1>click here</h1>");
    //设置<h1>鼠标样式
    $("h1").css("cursor","pointer");
    $("h1").click(function(){
        alert("你好");
    }).mouseover(function(){
         console.log("aaaa");
    });
```

### 六 、Ajax

#### 6.1、简介：

Ajax 异步局部刷新技术

​	异步：发送请求后不需要等待服务器返回结果后再去执行其他操作

   局部刷新：发送请求后不会刷新整个页面，只会刷新部分页面

#### 6.2、Jquery 调用ajax方法：

​	格式：$.ajax({})

​	参数说明：

​		type:请求方式 get/post

​		url：请求地址url

​		async:是否异步，默认是true表示异步

​		data:发送到服务器的数据

​		dataType:预期服务器返回的数据类型

​		contentType:设置请求头部

​       success:请求成功时调用此函数

​       error:请求失败时，调用此函数



​		

​			


