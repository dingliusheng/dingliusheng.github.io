# javaweb实例-订单管理系统（6）修改密码模块


<!--more-->

### 修改密码流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111225805181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
1、BaseDao 工具类

```java
//编写增删改公共方法
    public static int  excuteUpdate(Connection connection,String sql,Object[] parms) throws Exception {
        int updateRows;
        preparedStatement = connection.prepareStatement(sql);
        for (int i = 0; i < parms.length; i++) {
            preparedStatement.setObject(i+1,parms[i]);
        }
        updateRows = preparedStatement.executeUpdate();
        return  updateRows;
    }
```
2、UserDao

```java
 //修改用户密码
    public int updatePwd( Connection connection,String newPassword,int userId) throws Exception{
        String sql="update userinfo set userPassword = ? where userID = ? ";
        Object[] parms={newPassword,userId};
        int updateRow = BaseDao.excuteUpdate(connection, sql, parms);
        return updateRow;
    }
```
3、UserService

```java
   //用户修改密码,失败返回false ，成功为true
    public boolean updatePwd(String newPassword, int userID) throws Exception {
        boolean flag=false;
        Connection connection = BaseDao.getconnection();
        if(connection!=null){
            int i = userDao.updatePwd(connection, newPassword, userID);
            if(i>0){
                flag=true;
            }
        }
        BaseDao.closeResource(connection,null);
        return flag;
    }
```
4、UserServlet

```java
package com.tin.servlet.user;

import com.alibaba.fastjson.JSONArray;
import com.tin.pojo.UserInfo;
import com.tin.service.user.UserService;
import com.tin.service.user.UserServiceImpl;
import com.tin.util.Constants;
import javafx.application.Application;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.HashMap;
import java.util.Map;

public class UserServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //点击前端提交按钮，传递了参数method
       String method=req.getParameter("method");
       if(method.equals("savepwd")&&method!=null){
               this.updatePwd(req,resp);
       }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
    //修改密码
    public void updatePwd(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
        //从session 中拿userid
        UserInfo user =(UserInfo) req.getSession().getAttribute(Constants.USER_SESSION);
        //拿到重置的密码,input 的name属性
        String newPwd=req.getParameter("newpassword");
        boolean flag=false;
        //如果新密码获取成功
        if(newPwd!=null){
            UserService userService=new UserServiceImpl();
            try {
                flag = userService.updatePwd(newPwd, user.getUserID());
                if(flag){
                    req.setAttribute("message","密码修改成功，请退出重新登入");
                    //密码修改成功，移除当前session
                    req.getSession().removeAttribute(Constants.USER_SESSION);
                }else{
                    req.setAttribute("message","密码修改失败");
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }else{
            req.setAttribute("message","新密码有问题");
        }
        req.getRequestDispatcher("pwdmodify.jsp").forward(req,resp);
    }
  
}

```
5、前端接口：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210112091819370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
```xml
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@include file="/jsp/common/head.jsp"%>
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
    </section>
<%@include file="/jsp/common/foot.jsp" %>
<script type="text/javascript" src="${pageContext.request.contextPath }/js/pwdmodify.js"></script>

```
pwdmodify.js  //用Ajax 验证

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
	
	oldpassword.on("blur",function(){
		$.ajax({
			type:"GET",
			url:path+"/jsp/user.do",
			data:{method:"pwdmodify",oldpassword:oldpassword.val()},
			dataType:"json",
			success:function(data){
				if(data.result == "true"){//旧密码正确
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
	
	
	saveBtn.on("click",function(){
		oldpassword.blur();
		newpassword.blur();
		rnewpassword.blur();
		//
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


