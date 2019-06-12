演示案例：使用Java的ServerSocket和多线程，模拟一个Web服务器
● 实现步骤：
	1) 读取本地资源
	2) 发送给不同的客户端(给学生地址去访问)
	3) 采用多线程的方式
	
● 代码：
```java
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.io.OutputStream;
	import java.net.ServerSocket;
	import java.net.Socket;
	import java.sql.Timestamp;

        //项目下创建一个html文件, 了里面的内容是"百度一下, 你就什么也不知道了.
	//采用多线程的方式
	public class MyTomatServer  extends Thread {
		
		private Socket socket;
		
		public MyTomatServer(Socket socket) {
			this.socket = socket;
		}
		
		@Override
		public void run() {
			//创建文件输入流
			try(FileInputStream fis = new FileInputStream("index.html");
					OutputStream os = socket.getOutputStream();) {
				//创建字节的数组
				byte[] buf = new byte[1024];
				int len = 0;
				while((len=fis.read(buf))!=-1) {
					os.write(buf, 0, len);
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		//每一个新的用户连接上服务器的话，创建一个新的线程对象，让每一个线程去服务一个用户
		public static void main(String[] args) {
			System.out.println("服务器启动" + new Timestamp(System.currentTimeMillis()));
			//创建ServerSocket
			try(ServerSocket serverSocket = new ServerSocket(8080);) {
				//与客户端通信的对象
				while(true) {
					Socket socket = serverSocket.accept();
					System.out.println(new Timestamp(System.currentTimeMillis()) + " 用户的IP地址：" + 
						socket.getInetAddress().getHostAddress());
					new MyTomatServer(socket).start();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

	}
```
效果如下: 必须使用浏览器的http://访问, 否则访问也打不开网页. 
![](images/2019-06-10-21-57-12.png)