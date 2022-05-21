# javaweb实例-订单管理系统（2）设计数据库


<!--more-->


## 订单管理系统E-R图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103195928561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### 创建表
根据简略的ER图创建六个表
1、用户表
用户id、用户姓名、用户密码、用户权限，手机号码、地址
```sql
CREATE TABLE USER(
   id INT PRIMARY KEY,
   userName VARCHAR(50),
   userPassword VARCHAR(50),
   roleID INT,
   phone VARCHAR(11),
   address VARCHAR(50)
);
```
2、商品表
商品id、商品名称、价格、计量单位、库存、商品信息（商品介绍）
```sql
create table product{
	productID int primary key,
	productName VARCHAR(50),
	price float,
	unit VARCHAR(50),
	inventory int ,
	productInfo VARCHAR(100)	
}
```
3、供应商
供应商id、供应商名称、供应商地址、手机
```sql
create table supplier(
	supplierID INT PRIMARY KEY,
	supplierName VARCHAR(50),
	phone VARCHAR(11),
	address VARCHAR(50)
);
```
4、订单表
用户id 、商品id 、购买数量、下单日期

```sql
CREATE TABLE orders(
	userID INT ,
	productID INT,
	quantity INT,
	createDate DATE,
	PRIMARY KEY (userID,productID) 
);
```
5、采购表
供应商id、商品id、采购数量、采购日期

```sql
create table purchase(
	supplierID INT ,
	productID INT,
	quantity INT,
	createDate DATE,
	PRIMARY KEY (supplierID,productID) 
);
```

6、权限表
角色id、角色名称、角色权限说明

```sql
CREATE TABLE role(
   roleId INT PRIMARY KEY,
   roleName VARCHAR(50),
   roleInfo VARCHAR(100)
);
```
### 创建实体类
 - 必须有一个无参数构造
 - 属性必须私有化
 - 必须有对应的get/set方法
 
 **一般用来和数据库字段作映射** 
 
 - 表--->类
 - 字段--->属性
 - 行记录--->对象

以用userInfo 表为例
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210103214548969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)

在实体类包里创建UserInfo 类

```java
package com.tin.pojo;

public class UserInfo {
    private int userID;
    private String userName;
    private String userPassword;
    private int roleID;
    private String phone;
    private String address;


    //通过联合查询获取用户的角色名称
    private String userRoleName;

    public UserInfo() {
    }

    public int getUserID() {
        return userID;
    }

    public String getUserName() {
        return userName;
    }

    public String getUserPassword() {
        return userPassword;
    }

    public int getRoleID() {
        return roleID;
    }

    public String getPhone() {
        return phone;
    }

    public String getAddress() {
        return address;
    }

    public void setUserID(int userID) {
        this.userID = userID;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setUserPassword(String userPassword) {
        this.userPassword = userPassword;
    }

    public void setRoleID(int roleID) {
        this.roleID = roleID;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}

```



