实现Modbus RTU通信需要处理底层的串行通信协议，并根据Modbus RTU的标准进行数据包的组装和解析。以下是一个简单的示例，演示如何使用Java实现Modbus RTU通信。

首先，您需要一个可以处理串行端口通信的库。`jSerialComm`是一个流行的选择。您可以从Maven中央仓库中获取这个库。

### 1. 添加依赖项

在您的`pom.xml`文件中添加以下依赖项：

```xml
<dependency>
    <groupId>com.fazecast</groupId>
    <artifactId>jSerialComm</artifactId>
    <version>2.6.2</version>
</dependency>
```

### 2. 实现Modbus RTU通信

以下是一个基本的示例，演示如何使用Java和`jSerialComm`库进行Modbus RTU通信：

```java
import com.fazecast.jSerialComm.SerialPort;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;

public class ModbusRTUExample {

    private SerialPort serialPort;

    public ModbusRTUExample(String portName) {
        serialPort = SerialPort.getCommPort(portName);
        serialPort.setComPortParameters(9600, 8, SerialPort.ONE_STOP_BIT, SerialPort.NO_PARITY);
        serialPort.setComPortTimeouts(SerialPort.TIMEOUT_READ_BLOCKING, 1000, 1000);
    }

    public boolean open() {
        return serialPort.openPort();
    }

    public void close() {
        serialPort.closePort();
    }

    public byte[] readHoldingRegisters(int unitId, int startAddress, int quantity) {
        byte[] request = createReadHoldingRegistersRequest(unitId, startAddress, quantity);
        serialPort.writeBytes(request, request.length);

        byte[] response = new byte[5 + 2 * quantity]; // 5 bytes header + 2 bytes per register
        serialPort.readBytes(response, response.length);

        return response;
    }

    private byte[] createReadHoldingRegistersRequest(int unitId, int startAddress, int quantity) {
        ByteBuffer buffer = ByteBuffer.allocate(8);
        buffer.order(ByteOrder.BIG_ENDIAN);
        buffer.put((byte) unitId);
        buffer.put((byte) 0x03); // Function code for Read Holding Registers
        buffer.putShort((short) startAddress);
        buffer.putShort((short) quantity);
        int crc = calculateCRC(buffer.array(), 6);
        buffer.putShort((short) crc);
        return buffer.array();
    }

    private int calculateCRC(byte[] data, int length) {
        int crc = 0xFFFF;
        for (int pos = 0; pos < length; pos++) {
            crc ^= data[pos] & 0xFF;
            for (int i = 8; i != 0; i--) {
                if ((crc & 0x0001) != 0) {
                    crc >>= 1;
                    crc ^= 0xA001;
                } else {
                    crc >>= 1;
                }
            }
        }
        return crc;
    }

    public static void main(String[] args) {
        ModbusRTUExample modbus = new ModbusRTUExample("COM3");
        if (modbus.open()) {
            byte[] response = modbus.readHoldingRegisters(1, 0, 2);
            for (byte b : response) {
                System.out.printf("%02X ", b);
            }
            modbus.close();
        } else {
            System.out.println("Failed to open port.");
        }
    }
}
```

### 代码说明

1. **初始化串行端口**：`ModbusRTUExample`类初始化指定端口，并设置通信参数（波特率、数据位、停止位、奇偶校验）。
2. **打开和关闭端口**：`open()`和`close()`方法用于打开和关闭串行端口。
3. **发送Modbus请求**：`readHoldingRegisters`方法创建并发送读取保持寄存器的请求。
4. **创建Modbus请求**：`createReadHoldingRegistersRequest`方法生成读取保持寄存器的请求数据包。
5. **CRC校验**：`calculateCRC`方法计算Modbus RTU数据包的CRC校验码。
6. **主方法**：`main`方法演示如何使用上述方法进行Modbus RTU通信。

这个示例程序通过读取Modbus设备的保持寄存器来演示基本的Modbus RTU通信。如果需要与实际设备通信，您需要根据设备手册调整请求参数和处理响应数据。

### 注意事项

1. 确保您的串行端口参数（波特率、数据位、停止位、奇偶校验）与Modbus设备匹配。
2. 调整请求和响应的超时时间，以确保通信稳定性。
3. 在实际应用中，需要更复杂的错误处理和数据解析。

可以根据实际需求扩展和优化这个示例代码。