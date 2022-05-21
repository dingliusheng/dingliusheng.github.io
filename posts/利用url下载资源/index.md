# 利用URL下载资源


<!--more-->

### URL概念
URL：统一资源定位符
作用：定位互联网上的某一资源
>==DNS 域名解析== 将域名转为ip

DNS域名解析过程：
	1.输入域名
	2.检查本机的C:\Windows\System32\drivers\etc\hosts配置文件下有没有这个域名		   映射
		   如果找到了就返回对应的IP地址
	　　如果没有找到，就去DNS服务器找，找到返回，找不到则表示找不到资源
>==协议==：//==ip地址==：==端口==/==项目名==/==资源==
五部分组成

### 获取资源URL
 1. 打开浏览器开发者工具，选择Network
 2. 在左下角选择所需要的资源（以图片为例）
 3. 在右下角的Preview 查看是否是所需资源
 4. 在Header中找到URL地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201215192921224.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMTE2MDI3,size_16,color_FFFFFF,t_70#pic_center)
### java程序利用url下载资源

```java
package test04;

import java.io.FileOutputStream;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;

public class URLDemo {
    public static void main(String[] args) throws Exception {

            //下载地址
            URL url=new URL("https://goss.veer.com/creative/vcg/veer/800water/veer-171142505.jpg");
            //连接到这个资源
            HttpURLConnection urlConnection=(HttpURLConnection)  url.openConnection();
            //输入流
            InputStream inputStream=urlConnection.getInputStream();
            //复制到本地文件夹下
            FileOutputStream fos=new FileOutputStream("E:/download.jpg");
            byte[] buff=new byte[1024];
            int len;
            while((len=inputStream.read(buff))!=-1){
                fos.write(buff,0,len);
            }
            fos.close();
            inputStream.close();
            urlConnection.disconnect();//断开连接

    }
}

```


