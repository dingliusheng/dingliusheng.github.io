# TCP实现简单聊天


<!--more-->



##客户端
 1. 获取IP地址和端口
 2. 创建socket连接
 3. 输入发送信息
 4. 关闭管道
```java
package test01;

import java.io.OutputStream;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

//客户端
public class TcpClientDemo {
    public static void main(String[] args) {
      try{
            //知道服务器地址和端口
          InetAddress serverIP=InetAddress.getByName("127.0.0.1");
          int port=9999;
          //创建一个socket连接
          Socket socket=new Socket(serverIP,port);
          //发送消息
          OutputStream os=socket.getOutputStream();
          os.write("你好，我是XXX".getBytes());

          os.close();
          socket.close();
      }catch (Exception e){
          e.printStackTrace();
      }
    }
}

```
## 服务端
 1. 创建服务器
 2. 等待客户端连接
 3. 读取客户端信息
 4. 关闭管道

```java
package test01;

import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

//服务端
public class TcpSeriverDemo {
    public static void main(String[] args) {
        ServerSocket serverSocket=null;
        Socket socket=null;
        InputStream is=null;
        ByteArrayOutputStream baos=null;
        try{
            //建立服务端口
           serverSocket=new ServerSocket(9999);
            //等待客户端连接
            socket=serverSocket.accept();
            //读取客户端消息
            is=socket.getInputStream();
            //管道流
            baos=new ByteArrayOutputStream();
            byte[] buffer=new byte[1024];
            int len;
            while((len=is.read(buffer))!=-1){
                baos.write(buffer,0,len);
            }
            System.out.println(baos.toString());

            baos.close();
            is.close();
            socket.close();
            serverSocket.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```


