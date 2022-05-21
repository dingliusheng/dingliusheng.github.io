# js笔记


<!--more-->

# 一、js的使用方法

## 1.1 内联式

通过script标签，写JavaScript代码,位置可以用放在**head**或者**body**内

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--script标签内，写JavaScript代码,位置可以用放在head\body-->
    <script>
        alert("hello");
    </script>
</head>
<body>
    <script>
        console.log("hello");
    </script>
</body>
</html>
```

## 1.2  外联式

先在**js**文件里编写代码，然后在**HTML script**标签**src** 资源引用

tset_01.js

```javascript
alert("hello");
console.log("hello");
```

test_01.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--通过在src引用文件资源-->
    <script  src="tset_01.js"></script>
</head>
<body>
</body>
</html>
```

# 二、快速入门

### 2.1浏览控制台的使用

在浏览器打开开发者工具进行调试，具体使用方法

[开发者工具使用教程]:https://www.cnblogs.com/yaoyaojing/p/9530728.html

重点：可以设置断点和在**console中直接输入变量来查看变量值**

### 2.2js数据类型

==number==

js 不区分小数和整数

```javascript
123 //整数
123.1  //浮点数
1.123e3 //科学计数法
-99 //负数
NaN //not a number
Infinity //表示无限大
```

==字符串==

```javascript
'abc'
"我的"
```

==布尔值==

```javascript
true
false
```

==逻辑运算==

```javascript
&&  //与
||  //或
!   //取反
```

==比较运算符==

```javascript
==   //等于（类型不一样，值一样也为真）
===  //等于（类型和值都相同为真）
```

坚持不要使用==比较

1. NaN===NaN: NaN与所有的数值都不相等，包括自己。只能通过isNaN()来判断这个数是否是NaN

浮点数问题：
~~~javascript
console.log((1/3)=== (1-2/3))
~~~

**尽量避免使用浮点数进行运算，存在精度问题！**

==null和undefined==

- null 空

- undefined 未定义

==数组==

java数组是一组类型相同的对象，js不需要这样

```javascript
var arr=[1,2,3,4,"hello",null,true];
new array(1,2,3,4,"hello",null,true);
```

>取数组下标ined：如果越界了就会**undefined**

==对象==
> 每个属性之间使用逗号隔开，最后一个不要添加
```javascript
var person={
	name:"张三",
    age:12,
    tags:['js','python','c++']
}
```

>取对象的值

```javascript
person.name
>“张三”
person.age
>12
```

==变量==

```javascript
var 王者荣耀=倔强青铜；
```

js没有限定变量类型

### 2.3、严格检查模式strict

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!--'use strict';严格检查模式，预防JavaScript的随意性导致一些问题
		必须要放在第一行
        局部变量建议都使用 let 定义-->
    <script >
        'use strict';
         let i=1;
    </script>
</head>
<body>
</body>
</html>
```

# 三、数据类型

### 3.1、字符串

- 正常字符串使用单引号或者双引号包起来
- 注意转义字符 

```javascript
\'
\n		换行
\t		Tab空格
\u4e2d Unicode字符
\x41   Ascii字符
```
- 多行字符串编写
```javascript
  //tab键上面，esc下面，反引号
var msg=`hello
		 world
		 你好
		 世界`
```

-   模板字符串

```JavaScript
let name="zhangsan"
//用tab键上面，esc下面，反引号
console.log(`hello,${name}`)
```

- 字符串长度

```javaScript
console.log(str.length)
```

- 字符串的可变性，不可变

- 大小写转换
```javaScript
str.toUpperCase()
str.toLowerCase()
```

- 查找字符下标

```
str.indexOf('t')
```

- 截取字符串

```
str.substring(1)//从第一个字符截取到最后一个字符
str.substring(1,3)//[1,3)包含第一个字符，不包含第三个字符
```

###   3.2、数组

**array可以包含任意的数据类型**

```javascript
var arr=[1,2,3,"hello",true]
```
- 长度

```javascript
arr.length
```

  注意：给arr.length赋值，数组大小会发生变化，如果赋值过小，元素就会丢失

- indexOf

```
arr.indexOf(2)
```

字符串的“1”和数字1是不一样的

- slice()

```javascript
arr.slice(1) //下标从0开始，截取下标为1开始的所有的元素
arr.slice(1,5)//截取下标为1开始到下标为5之前的所有元素，包头不包尾
```

截取数组的一部分，返回一个新数组，类似于String中的substring

- push()

- pop()

```javascript
arr.push('a','b') //压入到尾部
arr.pop()//弹出尾部的一个元素
```

- unshift()

- shift()

```javascript
arr.unshift('a','b')//压入到头部
arr.shift()//弹出头部的一个元素
```

- sort()

```
arr.sort() //数组排序
```

- reverse()

```
arr.reverse()//元素反转
```

- concat()

```
arr.concat([1,2,3]) //数组拼接，返回一个新数组
```

注意：concat()并没有修改原数组，只是返回一个新的数组

- join()

```
arr.join("-")//使用选定的字符串连接数组元素，返回拼接后的字符串
```

- 多维数组

```
arr=[[1,2],[2,4],['a','b']]
```

数组：存储数据（如何存，如何取）

### 3.3、对象类型

若干个键值对

```javascript
//定义了person对象
var person{
	name:"张三",
    age:12,
    sex:man
}
```

js中对象，{......}表示一个对象，键值对描述属性，多个属性之间逗号隔开。

js对象所有键都是字符串，值可以是任意对象。

- 对象赋值

```
person.name="李四"
```

- 使用一个不存在的对象属性，不会报错！undefined

```
person.emal="12345678"
```

- 动态删减属性,通过delete删除对象的属性

```
delete person.name
```

- 动态的添加属性，直接**对象.属性**赋值

```
person.haha="haha"
```

- 判断属性值是否在对象中，xx in xx

```javascript
 name in  person
```

- 判断一个属性是否是这个对象自身拥有的

```
person.hasOwnProperty('toString')
```

### 3.4、Map和Set

```js
var m = new Map([['Michael', 95], ['Bob', 75], ['Tracy', 85]]);

m.get('Michael'); // 95，通过键获取对应的值
m.set('lisi',80)  //  新增或修改键值对
m.delete('Tracy') // 删除键值对
```

map 类似于python的字典，通过键来获取相应的值，**键只能用单引号**包起来

```js
let set = new Set([3,4,2,7,9，3]); //定义Set
console.log(Array.from(set)); //输出set 所有元素
set.add(2);  //添加元素
set.delete(3); //删除元素
console.log(set.has(3)); //判断set是否有这个元素
```

**set是无序不重复的集合**,new set()接收一个数组

### 3.5、iterator

遍历数组

```js
var arr=[1,2,3,4,5,6]
//num是下标
for(let num in arr){
	console.log(arr[num]);
}
//num是arr的元素
for(let num of arr){
    console.log(num);
}
```

遍历map

```js
var map=new Map([['zhangsan',12],['lisi',23],['wangwu',56]])
for(let x of map){
    console.log(x);
}
```

遍历set

```js
var set=new Set([5,6,7])
for(let x of set){
	console.log(x);
}
```



# 四、流程控制

### 4.1、if判断

```javascript
if(age>12){
	alert("haha");
}else if(age>2 && age<10){
	alert("smile");
}else{
	alert("cry");
}
```

### 4.2、while循环

```javascript
var age=3;
while(age<30){
	age=age+1;
}
```

### 4.3、for循环

```
for(let i=0;i<100;i++){
	console.log(i);
}
```

### 4.4、forEach循环

```js
var arr=[1,2,3,4,5,6]
//需要放在函数中
arr.forEach(function(value){
	console.log(value);
})
```

### 4.5、  for  in 遍历

```js
var arr=[1,2,3,4,5,6]
//num是下标
for(var num in arr){
	console.log(arr[num]);
}
//num是arr的元素
for(var num of arr){
    console.log(num);
}
```

# 五、函数

### 5.1、定义函数

>定义方式一

```js
function abs(x){
    if(x>=0){
        return x;
    }else{
        return -x;
    }
}
```

执行到return代表函数结束，返回结果。如果没有执行return，函数执行完也会返回结果undfined

>定义方式二

```js
var abs=function(x){
	if(x>=0){
        return x;
    }else{
        return -x;
    }
}
```

function(x){......}这是一个匿名函数，可以把结果赋值给abs,通过abs就可以调用函数。

方式一和方式二是一样的！

### 5.2、调用函数

```js
abs(-10) //10
abs(10)  //10
```

>参数问题：Javascript可以传递任意个参数，也可以不传递参数。参数是否存在？参数是否符合要求？

```js
var abs=function(x){
	//如果参数不是数字就抛出异常
	if(typeof(x)!=='number'){
		throw 'Not a number';
	}
	if(x>=0){
        return x;
    }else{
        return -x;
    }
}
```

>arguments  参数数组

`arguments`是一个关键字

```js
var abs=function(x){
	//通过arguments打印每一个参数
	for(let i=0;i<arguments.length;i++){
		console.log(arguments[i]);
	}
	
	if(x>=0){
        return x;
    }else{
        return -x;
    }
}
```

问题：arguments包含所有的参数，有时候需要使用多余的参数来进行附加操作，需要排除已有的参数。

>rest   

ES6引入的新特性，获取除了已经定义的参数外的所有参数

```js
//需要用...表示多余的多个参数
function concat(a,b,...rest){
	console.log(rest)
	return a+b;
}
concat('a','b','c','d')
```

### 5.3、变量的作用域

在JavaScript中，var定义变量实际是有作用域的。

假设在函数体中，则在函数体外不可以使用

```js
function a(){
	var x=1;
}
x=x+2;//x is not defined 
```

两个函数使用相同的变量名，不冲突

```js
funtion a(){
	var i=0;
}
function b(){
	var i='a';
}
```

内部函数可以访问外部函数的成员，反之则不行

内部函数与外部函数变量同名时，根据就近原则，内部函数优先选内部定义的变量值

```js
function a(){
	var i=0;
	function b(){
		var i='a';
		console.log(i);
	}
	b();
	console.log(i);
}
```

>提升变量的作用域

```js
function a(){
	var x="x"+y;
	console.log(x);
	var y='y';
}
```

结果： x  undfined 

说明：js执行引擎，自动提升了y的声明，但是不会提升变量y的赋值,等于下面这个效果

```js
function a(){
    var y;
	var x="x"+y;
	console.log(x);
	y='y';
}
```

**规范：所有的变量定义都放在头部，便于代码维护；**

>全局变量

```js
//全局变量
var i=0;
function a(){
	i=6;
	console.log(i);
}
function b(){
	i='a';
	console.log(i);
}
```

可以在函数中引用修改全局变量

>全局对象 window

```js
var x='xxx';
alert(x);
alert(window.x);//默认所有的全局变量，都会自动绑定在window对象下；
```

alert()这个函数本身也是一个`window`变量；

```
var alert_old=window.alert;
window.alert=function(){
	//重写alert方法
}
```

JavaScript实际上只有一个全局作用域，任何变量（函数也可以视为变量），假设没有在函数作用范围内找到，就会向外查找，如果在全局作用域都没有找到，报错`RefrenceError`

>规范

由于所有的全局变量都会绑定到window上，如果不同的js文件使用了相同的全家变量，发生冲突

如何减少冲突？

```js
//定义唯一全局对象
var myData={}
//定义全局变量
myData.name='zhangsan';
myData.x='a';
myData.add=function(a,b){
    return a+b;
}
```

用一个唯一全局对象来代替window对象来存储全局变量的功能，降低全局命名冲突的问题

>局部作用域  let

```js
function a(){
	for(var i=0;i<10;i++){
		console.log(i);
	}
	console.log(i+1);//问题？i出了作用域还可以使用
}
```

ES6 let 关键字解决局部域冲突问题

```js
function a(){
	for(let i=0;i<10;i++){
		console.log(i);
	}
	console.log(i+1);//报错，出了i的作用域
}
```

建议使用`let`定义局部变量；

>常量 const

```js
const PI=3.14;
PI=3;
```

用全部大写字母命名的变量就是常量，不建议修改

