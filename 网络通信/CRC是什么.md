CRC（循环冗余校验，Cyclic Redundancy Check）是一种用于检测数据传输或存储错误的算法。CRC 在数据通信和存储设备中广泛使用，以确保数据的完整性。它将数据视为一个二进制多项式，利用多项式除法生成一个校验值（即 CRC 值），并附加到数据块上。接收端使用相同的算法对接收到的数据计算 CRC，并与附加的 CRC 值进行比较。如果两者不一致，说明数据在传输或存储过程中出现了错误。

### CRC的基本工作原理
1. **生成多项式**: CRC 算法使用一个预定义的生成多项式（也称为多项式除数）。
2. **数据编码**: 数据被视为一个二进制多项式。
3. **计算余数**: 将数据多项式除以生成多项式，得到的余数就是 CRC 校验值。
4. **附加CRC值**: 将 CRC 校验值附加到数据后面一起传输。
5. **接收端校验**: 接收端使用相同的生成多项式对接收到的数据进行同样的计算，得到的余数与附加的 CRC 校验值进行比较。如果匹配，则认为数据无误；否则，数据出现了错误。

### 常见的CRC类型
- **CRC-8**: 8位的 CRC 校验。
- **CRC-16**: 16位的 CRC 校验，常用于 Modbus 通信。
- **CRC-32**: 32位的 CRC 校验，广泛用于网络传输和文件校验。

### 示例：CRC-16 计算
下面是一个 Java 实现 CRC-16 Modbus 校验的示例代码：

```java
public class CRC16Modbus {

    // CRC16-MODBUS 生成多项式 0xA001
    private static final int POLYNOMIAL = 0xA001;

    public static void main(String[] args) {
        byte[] data = {0x01, 0x03, 0x00, 0x65, 0x00, 0x03};
        int crc = calculateCRC(data);
        System.out.printf("CRC16: %04X\n", crc);
    }

    public static int calculateCRC(byte[] data) {
        int crc = 0xFFFF; // 初始值

        for (byte b : data) {
            crc ^= (b & 0xFF); // 将字节与 CRC 的低字节异或
            for (int i = 0; i < 8; i++) {
                if ((crc & 0x0001) != 0) {
                    crc >>= 1;
                    crc ^= POLYNOMIAL;
                } else {
                    crc >>= 1;
                }
            }
        }
        return crc;
    }
}
```

### 解释
1. **初始值**: CRC 的初始值通常设置为 `0xFFFF`。
2. **异或操作**: 将每个数据字节与 CRC 的低字节进行异或操作。
3. **移位与异或**: 根据最低位是 `1` 还是 `0` 决定是右移还是右移后再与多项式进行异或操作。
4. **返回 CRC**: 最终的 CRC 值即为校验结果。

### 应用
CRC 被广泛应用于数据通信协议（如 Modbus、HDLC、以太网）、存储设备（如硬盘、光盘）、文件校验（如 zip 文件）等领域，以保证数据传输和存储的可靠性。