# 利用TCP实现文件上传


<!--more-->



## 服务端
 1. 创建服务器
 2. 监听客户连接
 3. 接收文件
 4. 返回提示信息
```java
package test02;

import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.io.File;

public class TcpServerDemo {
    public static void main(String[] args) {
        try{
            //创建服务器
            ServerSocket serverSocket=new ServerSocket(9999);
            //监听客户端连接
            Socket socket=serverSocket.accept();
            //获取输入流
            InputStream is=socket.getInputStream();
            //文件输出
            File f=new File("E:/b.txt");
            FileOutputStream fos=new FileOutputStream(f);
            byte[] buffer=new byte[1024];
            int len;
            while((len=is.read(buffer))!=-1){
                fos.write(buffer,0,len);
            }
            //通知客户端接收完毕
            OutputStream os=socket.getOutputStream();
            os.write("文件上传完毕".getBytes());
            os.close();

            //关闭通道
            fos.close();
            is.close();
            socket.close();
            serverSocket.close();

        }catch (Exception e){

        }
    }
}

```
## 客户端
 1. 获取服务器IP和端口号
 2. 创建socket连接
 3. 上传文件
 4. 通知服务器上传完毕
 5. 接收提示信息
```java
package test02;

import java.io.*;
import java.net.InetAddress;
import java.net.Socket;

public class TcpClientDemo {
    public static void main(String[] args) {
        try {
            //获取IP和端口号
            InetAddress serverIP = InetAddress.getByName("127.0.0.1");
            int port = 9999;
            //创建一个socket连接
            Socket socket=new Socket(serverIP,port);
            //创建输出流
            OutputStream os=socket.getOutputStream();
            //读取文件
            File f=new File("E:/a.txt");
            FileInputStream fis=new FileInputStream(f);
            //读出文件内容
            byte[] buffer=new byte[1024];
            int len;
            while((len=fis.read(buffer))!=-1){
                os.write(buffer,0,len);
            }
            //通知服务器端文件传输完毕
            socket.shutdownOutput();
            //接收服务端返回信息
            InputStream is=socket.getInputStream();
            ByteArrayOutputStream baos= new ByteArrayOutputStream();
            byte[] buffer2=new byte[1024];
            int len2;
            while((len2=is.read(buffer2))!=-1){
                baos.write(buffer2,0,len2);
            }
            System.out.println(baos.toString());
            //关闭通道
            fis.close();
            os.close();
            socket.close();
        } catch (Exception e) {
                e.printStackTrace();
        }
    }
}
```


