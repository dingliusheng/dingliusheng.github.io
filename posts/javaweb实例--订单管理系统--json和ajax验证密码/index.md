# javaweb实例-订单管理系统（7）验证密码模块


<!--more-->

### web1.0时代
 - 早期网站，登入，如果失败，需要刷新页面，才能重新登入；不点击提交按钮不知道自己密码输错了。
 - 现在大多数的网站，都是局部刷新，不刷新整个页面的情况下，实现页面更新。
 - 像百度搜索框，当年输入字进去就会在下方出现提示选项
### 什么是json
JSON(JavaScript Object Notation, JS 对象简谱) 是一种轻量级的数据交换格式。
### json格式
```javascript
{"name","zhangsan"} //用{} 表示一个对象
[{"name","张三"},{"name","李四"}] //用[]表示一个数组
```
>json字符串：指的是符合json格式要求的js字符串。
>例如：var jsonStr = "{StudentID:'100',Name:'tmac',Hometown:'usa'}";
json对象：指符合json格式要求的js对象。
例如：var jsonObj = { StudentID: "100", Name: "tmac", Hometown: "usa" };
### 什么是AJAX
ajax一个前后台配合的技术，它可以让javascript发送http请求，与后台通信，获取数据和信息。ajax技术的原理是实例化xmlhttp对象，使用此对象与后台通信。jquery将它封装成了一个函数$.ajax()，我们可以直接用这个函数来执行ajax请求。
### AJAX的使用
常用参数：
1、url 请求地址
2、type 请求方式，默认是’GET’，常用的还有’POST’
3、dataType 设置返回的数据格式，常用的是’json’格式，也可以设置为’html’
4、data 设置发送给服务器的数据
5、success 设置请求成功后的回调函数
6、error 设置请求失败后的回调函数
7、async 设置是否异步，默认值是’true’，表示异步
### json和AJAX实现验证密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210112215813565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

首先是jsp
```xml
<div class="right">
            <div class="location">
                <strong>你现在所在的位置是:</strong>
                <span>密码修改页面</span>
            </div>
            <div class="providerAdd">
                <form id="userForm" name="userForm" method="post" action="${pageContext.request.contextPath }/jsp/user.do">
                    <input type="hidden" name="method" value="savepwd">
                    <!--div的class 为error是验证错误，ok是验证成功-->
                    <div class="info">${message}</div>
                    <div class="">
                        <label for="oldPassword">旧密码：</label>
                        <input type="password" name="oldpassword" id="oldpassword" value=""> 
						<font color="red"></font>
                    </div>
                    <div>
                        <label for="newPassword">新密码：</label>
                        <input type="password" name="newpassword" id="newpassword" value=""> 
						<font color="red"></font>
                    </div>
                    <div>
                        <label for="newPassword">确认新密码：</label>
                        <input type="password" name="rnewpassword" id="rnewpassword" value=""> 
						<font color="red"></font>
                    </div>
                    <div class="providerAddBtn">
                        <!--<a href="#">保存</a>-->
                        <input type="button" name="save" id="save" value="保存" class="input-button">
                    </div>
                </form>
            </div>
        </div>
```
用AJAX 传递数据到后台
**前提，引入jquery**
```xml
<script type="text/javascript" src="${pageContext.request.contextPath }/js/jquery-1.8.3.min.js"></script>
```
**jsp**
```javascript
var oldpassword = null;
var newpassword = null;
var rnewpassword = null;
var saveBtn = null;

$(function(){
	oldpassword = $("#oldpassword");
	newpassword = $("#newpassword");
	rnewpassword = $("#rnewpassword");
	saveBtn = $("#save");
	
	oldpassword.next().html("*");
	newpassword.next().html("*");
	rnewpassword.next().html("*");
	//blur 当鼠标移出输入框时触发事件
	//当输入旧密码后，失去聚焦事件触发，发出url:path+"/jsp/user.do"请求
	//在servlet 获取method 和 oldpassword 
	//接收返回的json
	oldpassword.on("blur",function(){
		$.ajax({
			type:"GET",
			url:path+"/jsp/user.do",
			data:{method:"pwdmodify",oldpassword:oldpassword.val()},
			dataType:"json",
			success:function(data){
				if(data.result == "true"){//旧密码正确
				//
					validateTip(oldpassword.next(),{"color":"green"},imgYes,true);
				}else if(data.result == "false"){//旧密码输入不正确
					validateTip(oldpassword.next(),{"color":"red"},imgNo + " 原密码输入不正确",false);
				}else if(data.result == "sessionerror"){//当前用户session过期，请重新登录
					validateTip(oldpassword.next(),{"color":"red"},imgNo + " 当前用户session过期，请重新登录",false);
				}else if(data.result == "error"){//旧密码输入为空
					validateTip(oldpassword.next(),{"color":"red"},imgNo + " 请输入旧密码",false);
				}
			},
			error:function(data){
				//请求出错
				validateTip(oldpassword.next(),{"color":"red"},imgNo + " 请求错误",false);
			}
		});
		//当鼠标移入输入框，聚焦事件
	}).on("focus",function(){
		validateTip(oldpassword.next(),{"color":"#666666"},"* 请输入原密码",false);
	});
	
	newpassword.on("focus",function(){
		validateTip(newpassword.next(),{"color":"#666666"},"* 密码长度必须是大于6小于20",false);
	}).on("blur",function(){
		if(newpassword.val() != null && newpassword.val().length > 5
				&& newpassword.val().length < 20 ){
			validateTip(newpassword.next(),{"color":"green"},imgYes,true);
		}else{
			validateTip(newpassword.next(),{"color":"red"},imgNo + " 密码输入不符合规范，请重新输入",false);
		}
	});
	rnewpassword.on("focus",function(){
		validateTip(rnewpassword.next(),{"color":"#666666"},"* 请输入与上面一致的密码",false);
	}).on("blur",function(){
		if(rnewpassword.val() != null && rnewpassword.val().length > 5
				&& rnewpassword.val().length < 20 && newpassword.val() == rnewpassword.val()){
			validateTip(rnewpassword.next(),{"color":"green"},imgYes,true);
		}else{
			validateTip(rnewpassword.next(),{"color":"red"},imgNo + " 两次密码输入不一致，请重新输入",false);
		}
	});
	
	//点击保存事件
	saveBtn.on("click",function(){
		oldpassword.blur();
		newpassword.blur();
		rnewpassword.blur();
		if(oldpassword.attr("validateStatus") == "true" &&
			 newpassword.attr("validateStatus") == "true"
			&& rnewpassword.attr("validateStatus") == "true"){
			if(confirm("确定要修改密码？")){
				$("#userForm").submit();
			}
		}
		
	});
});
```
**servlet**
```java
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
 //根据method 传的值不同来判断
       String method=req.getParameter("method");
       if(method.equals("savepwd")&&method!=null){
       //点击保存按钮，响应更新密码操作
               this.updatePwd(req,resp);
       }else if(method.equals("pwdmodify")&&method!=null){
       //鼠标移出原密码的输入框，响应返回原密码和输入的原密码是否相同的结果true or false
               this.pwdModify(req,resp);
       }
    }
```
//原密码的比较
```java
//验证旧密码
    public void pwdModify(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
    //获取存在session的用户对象的密码
        UserInfo user =(UserInfo) req.getSession().getAttribute(Constants.USER_SESSION);
        //获取输入的原密码（AJAX data:{method:"pwdmodify",oldpassword:oldpassword.val()}返回给服务器的参数）
        String oldpassword=req.getParameter("oldpassword");
        //结果集
        Map<String,String> resultMap=new HashMap<String, String>();
        if(user==null){
            //session 失效了
            resultMap.put("result","sessionerror");
        }else if(oldpassword==null){
            //输入密码为空
            resultMap.put("result","error");
        }else{
            String userpassword=user.getUserPassword();
            if(oldpassword.equals(userpassword)){
                //旧密码验证正确
                resultMap.put("result","true");
            }else{
                //旧密码验证不正确
                resultMap.put("result","false");
            }
        }
        //响应返回json 格式
        resp.setContentType("Application/json");
        PrintWriter writer = resp.getWriter();
        //JSONArray 阿里巴巴的json工具类，将结果集输出到页面去
        writer.write(JSONArray.toJSONString(resultMap));
        writer.flush();
        writer.close();
    }
```


