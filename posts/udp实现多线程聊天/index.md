# TCP实现多线程聊天


<!--more-->


### 接收线程
 1. 创建套接字Socket ,监听端口
 2. 等待并接收数据包
 3. 读取数据包的数据
 4. 判断是否结束
```java
package test03;

import java.io.BufferedReader;
import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.SocketException;

public class Receiver implements Runnable{
    DatagramSocket socket=null;
    private int port;
    private String msgFrom;
    Receiver(int port,String msgFrom){
        this.port=port;
        this.msgFrom=msgFrom;
        try {
            //创建socke，监测端口
            socket=new DatagramSocket(port);
        } catch (SocketException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        while(true){
            try {
                //准备接收数据包
                byte[] buff=new byte[1024];
                DatagramPacket packet=new DatagramPacket(buff,0,buff.length);
                socket.receive(packet);
				//读取数据包的数据
                byte[] data=packet.getData();
                String receiveData=new String(data,0,data.length);
                System.out.println(msgFrom+": "+receiveData);
                //如果是bye 则停止接收
                if(receiveData.equals("bye")){
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        socket.close();
    }
}

```
### 发送线程
 1. 创建套接字Socket（和接收方的socket监测的端口号不同）
 2. 读取键盘输入字符串
 3. 创建数据包
 4. 发送数据包
 5. 判断是否结束
```java
package test03;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;

public class Sender implements  Runnable{
    DatagramSocket socket=null;
    BufferedReader reader=null;
    private  int fromPort;
    private String toIP;
    private  int toPort;
    Sender(int formPort,String toIP,int toPort){
        this.fromPort=formPort;
        this.toIP=toIP;
        this.toPort=toPort;
        try {
            //创建对象 , 该 Socket 会监听固定端口 ; 注意该端口是用于监听数据接收的 ; 发送数据使用的不是该端口号 ;
            socket = new DatagramSocket(fromPort);
            //读取键盘输入
            reader=new BufferedReader(new InputStreamReader(System.in));
        }catch (Exception e){
        }
    }

    @Override
    public void run() {
        while (true) {
            try {
                //读取键盘输入一行信息
                String data=reader.readLine();
                //将字符流转为字节流
                byte[] datas=data.getBytes();
                //创建数据包
                //参数 ：数据、数据开始和结束的位置、接收方的ip和端口号
                DatagramPacket packet=new DatagramPacket(datas,0,datas.length,
                        new InetSocketAddress(this.toIP,this.toPort));
                //发送数据包
                socket.send(packet);
                //输入bye结束
                if(data.equals("bye")){
                    break;
                }
            } catch (Exception e) {
            }
        }
        socket.close();
    }
}

```
### 用户
创建发送线程和接收线程

```java
package test03;

public class UserA {
    public static void main(String[] args) {
        //开启两个线程
        //己方发送套接字监听的端口号7777，对方接收套接字的IP和端口号9999
         Sender sender=new Sender(7777,"127.0.0.1",9999);
         //己方接收套接字端口8888
         Receiver receiver=new Receiver(8888,"学生");
         new Thread(sender).start();
         new Thread(receiver).start();
    }
}
```

```java
package test03;

public class UseB {
    public static void main(String[] args) {
        //开启两个线程
        Sender sender=new Sender(5555,"127.0.0.1",8888);
        Receiver receiver=new Receiver(9999,"老师");
        new Thread(sender).start();
        new Thread(receiver).start();
    }
}

```


