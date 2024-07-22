## 服务端

```java
import java.io.*;
import java.net.*;

public class TcpServer {
    public static void main(String[] args) {
        // 监听的端口号
        int port = 12345;
        try (ServerSocket serverSocket = new ServerSocket(port)) {
//            System.out.println("Server is listening on port " + port);

            while (true) {
                try (Socket socket = serverSocket.accept()) {
                    System.out.println("New client connected");

                    InputStream input = socket.getInputStream();
                    BufferedReader reader = new BufferedReader(new InputStreamReader(input));

                    OutputStream output = socket.getOutputStream();
                    PrintWriter writer = new PrintWriter(output, true);

                    String text;

                    // 读取客户端发送的消息
                    while ((text = reader.readLine()) != null) {
                        System.out.println("服务端接收到消息: " + text);
                        // 向客户端发送回应
                        writer.println("已接收的你发的消息-" + text);
                    }
                } catch (IOException e) {
                    System.out.println("Server exception: " + e.getMessage());
                    e.printStackTrace();
                }
            }

        } catch (IOException ex) {
            System.out.println("Server exception: " + ex.getMessage());
            ex.printStackTrace();
        }
    }
}
```



## 客户端

```java
import java.io.*;
import java.net.*;

public class TcpClient {
    public static void main(String[] args) {
        String hostname = "localhost";
        int port = 12345;

        try (Socket socket = new Socket(hostname, port)) {

            OutputStream output = socket.getOutputStream();
            PrintWriter writer = new PrintWriter(output, true);

            InputStream input = socket.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(input));

            // 向服务器发送消息
            writer.println("我们都已经成功了");

            String response = reader.readLine();
            System.out.println("接收到服务端返回的消息: " + response);

        } catch (UnknownHostException ex) {
            System.out.println("66666Server not found: " + ex.getMessage());
        } catch (IOException ex) {
            System.out.println("7777I/O error: " + ex.getMessage());
        }
    }
}


```

