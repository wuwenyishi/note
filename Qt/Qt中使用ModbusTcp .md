## ModBusTcp的连接

在C++ Qt中使用`QModbusClient`进行Modbus通信，您可以按照以下步骤进行操作：

1. 包含必要的头文件：
```cpp
#include <QModbusClient>
#include <QModbusDataUnit>
#include <QModbusTcpClient> // 如果使用TCP连接
#include <QModbusRtuSerialMaster> // 如果使用串口连接
```

2. 创建`QModbusClient`对象：
```cpp
QModbusClient *modbusClient = new QModbusTcpClient(parent); // 如果使用TCP连接
QModbusClient *modbusClient = new QModbusRtuSerialMaster(parent); // 如果使用串口连接
```
请注意，`parent`是可选的，用于管理对象的生命周期。

3. 连接到Modbus设备：
```cpp
modbusClient->setConnectionParameter(QModbusDevice::NetworkPortParameter, 502); // 设置端口号，如果使用TCP连接
modbusClient->setConnectionParameter(QModbusDevice::NetworkAddressParameter, "192.168.0.1"); // 设置IP地址，如果使用TCP连接

modbusClient->connectDevice();
```
根据您的实际情况，选择适当的连接参数函数来设置连接的详细信息。

4. 监听连接状态变化：
```cpp
QObject::connect(modbusClient, &QModbusClient::stateChanged, [&](QModbusDevice::State state) {
    if (state == QModbusDevice::ConnectedState) {
        // 连接成功
    } else if (state == QModbusDevice::UnconnectedState) {
        // 连接断开
    }
});
```
通过连接`stateChanged`信号，您可以在连接状态发生变化时获取通知。

5. 发送Modbus请求：
```cpp
QModbusDataUnit readRequest(QModbusDataUnit::HoldingRegisters, 0, 10); // 创建读取请求
if (auto *reply = modbusClient->sendReadRequest(readRequest, slaveAddress)) {
    if (!reply->isFinished()) {
        QObject::connect(reply, &QModbusReply::finished, [&]() {
            if (reply->error() == QModbusDevice::NoError) {
                // 请求成功，处理回复数据
                const QModbusDataUnit &data = reply->result();
                // ...
            } else {
                // 请求失败，处理错误
                qDebug() << "Modbus read request error:" << reply->errorString();
            }
            reply->deleteLater();
        });
    } else {
        delete reply;
    }
} else {
    // 发送请求失败
}
```
这是一个示例的读取请求，您可以根据需要创建读取或写入请求，并通过`sendReadRequest`或`sendWriteRequest`函数发送请求。使用`QModbusReply`来处理请求的回复。



**注意：**

当使用`QModbusClient`的`connectDevice`函数连接设备时，成功连接设备后，`QModbusClient`的`state`属性可能仍然显示为`UnconnectedState`。这是因为`QModbusClient`的状态是异步更新的，`connectDevice`函数只是启动了连接过程，并不会立即更新状态。

为了获取准确的状态信息，您可以使用`QModbusClient`的`stateChanged`信号来监视状态的变化。当连接状态发生变化时，`stateChanged`信号会被触发，您可以在槽函数中获取最新的状态。

以下是一个示例代码，展示了如何使用`QModbusClient`连接设备并监视状态的变化：

```cpp
#include <QApplication>
#include <QModbusClient>
#include <QDebug>

int main(int argc, char *argv[]) {
    QApplication app(argc, argv);

    QModbusClient modbusClient;

    // 连接设备
    modbusClient.connectDevice();

    // 监听状态变化
    QObject::connect(&modbusClient, &QModbusClient::stateChanged, [&](QModbusDevice::State state) {
        qDebug() << "Modbus client state changed:" << state;
    });

    // 其他操作...

    return app.exec();
}
```

在这个示例中，我们创建了一个`QModbusClient`对象`modbusClient`，然后调用`connectDevice`函数连接设备。接着，通过连接`stateChanged`信号和一个槽函数，我们可以在状态发生变化时输出最新的状态信息。

请注意，连接设备的过程可能需要一些时间，因此在刚开始连接时，`stateChanged`信号可能不会立即触发。您可以在其他操作中等待一段时间，然后再检查`QModbusClient`的状态。



## QModbusTcpClient 是否支持长连接

根据我的最新了解，`QModbusTcpClient` 类本身不直接支持长连接。每次调用 `connectToDevice()` 方法时，将建立一个新的 TCP 连接，并在通信完成后自动关闭连接。

要实现长连接的效果，你可以通过以下方法之一来处理：

1. 自行管理连接状态：你可以使用 `QModbusTcpClient` 的 `stateChanged` 信号来监测连接状态的变化，并在连接断开后重新连接。你可以编写逻辑来处理连接断开事件，并在合适的时机调用 `connectToDevice()` 方法进行重新连接。
2. 使用继承：你可以继承 `QModbusTcpClient` 类并重写相关方法，以实现添加自动重新连接功能的子类。在子类中，你可以编写逻辑来处理连接断开事件，并在需要的时候重新连接。

请注意，通过以上两种方法实现的长连接都是基于逻辑处理的，而不是直接在 `QModbusTcpClient` 类级别上提供的内置功能。因此，在实际应用中，你需要根据具体需求和情况选择适合的方式来实现长连接的逻辑。



## C++ Qt如何实现ModbusTcp 读写

要在C++中实现Modbus TCP的读写操作，您可以使用QModbusTcpClient类来进行通信。以下是基本的读写操作示例：

1. 连接到Modbus服务器：

```cpp
QModbusTcpClient* client = new QModbusTcpClient(this);
client->setConnectionParameter(QModbusDevice::NetworkAddressParameter, "192.168.0.1");
client->setConnectionParameter(QModbusDevice::NetworkPortParameter, 502);

// 连接到Modbus服务器
if (!client->connectDevice()) {
    qDebug() << "无法连接到Modbus服务器：" << client->errorString();
    return;
}
```

2. 读取Modbus寄存器：

```cpp
QModbusDataUnit readUnit(QModbusDataUnit::HoldingRegisters, startAddress, numberOfEntries);
if (auto* reply = client->sendReadRequest(readUnit, slaveAddress)) {
    if (!reply->isFinished()) {
        connect(reply, &QModbusReply::finished, this, [=]() {
            if (reply->error() == QModbusDevice::NoError) {
                const QModbusDataUnit unit = reply->result();
                // 处理读取到的寄存器数据
            } else {
                qDebug() << "读取Modbus寄存器错误：" << reply->errorString();
            }
            reply->deleteLater();
        });
    } else {
        delete reply;
    }
} else {
    qDebug() << "无法发送Modbus读取请求：" << client->errorString();
}
```

3. 写入Modbus寄存器：

```cpp
QModbusDataUnit writeUnit(QModbusDataUnit::HoldingRegisters, startAddress, values);
if (auto* reply = client->sendWriteRequest(writeUnit, slaveAddress)) {
    if (!reply->isFinished()) {
        connect(reply, &QModbusReply::finished, this, [=]() {
            if (reply->error() == QModbusDevice::NoError) {
                // 写入成功
            } else {
                qDebug() << "写入Modbus寄存器错误：" << reply->errorString();
            }
            reply->deleteLater();
        });
    } else {
        delete reply;
    }
} else {
    qDebug() << "无法发送Modbus写入请求：" << client->errorString();
}
```

请根据您实际的Modbus设备和通信需求进行适当的修改和调整。确保在使用QModbusTcpClient之前，正确设置了Modbus服务器的地址和端口，并处理连接和请求的回调函数。



## Qt C++ QTcpSocket 替代 QModbusTcpClient 读写操作

如果你决定使用 QTcpSocket 类替代 QModbusTcpClient 来进行 Modbus 读写操作，你需要自己实现 Modbus 协议的编码和解码，并使用 QTcpSocket 来发送和接收数据。下面是一个简单的示例代码，演示如何使用 QTcpSocket 实现 Modbus 的读写操作：

```cpp
#include <QTcpSocket>
#include <QDataStream>

int main(int argc, char *argv[])
{
    QTcpSocket socket;
    socket.connectToHost("127.0.0.1", 502);  // 连接到 Modbus 设备的 IP 地址和端口号
    if (socket.waitForConnected())
    {
        // 发送 Modbus 读命令
        QByteArray request;
        QDataStream out(&request, QIODevice::WriteOnly);
        out.setByteOrder(QDataStream::BigEndian);  // 设置字节顺序为大端模式

        quint8 transactionIdentifier = 0x0001;
        quint16 protocolIdentifier = 0x0000;
        quint16 length = 0x0006;
        quint8 unitIdentifier = 0x01;
        quint8 functionCode = 0x03;
        quint16 startAddress = 0x0000;
        quint16 numRegisters = 0x0002;

        out << transactionIdentifier << protocolIdentifier << length << unitIdentifier << functionCode << startAddress << numRegisters;

        socket.write(request);
        socket.waitForBytesWritten();

        // 接收 Modbus 响应
        QByteArray response;
        if (socket.waitForReadyRead())
        {
            response = socket.readAll();

            // 解析 Modbus 响应
            QDataStream in(response);
            in.setByteOrder(QDataStream::BigEndian);  // 设置字节顺序为大端模式

            quint8 receivedUnitIdentifier;
            quint8 receivedFunctionCode;
            quint8 receivedByteCount;
            QVector<quint16> registerValues;

            in >> receivedUnitIdentifier >> receivedFunctionCode >> receivedByteCount;

            for (int i = 0; i < receivedByteCount / 2; ++i)
            {
                quint16 registerValue;
                in >> registerValue;
                registerValues.append(registerValue);
            }

            // 处理读取到的寄存器值
            for (quint16 registerValue : registerValues)
            {
                qDebug() << "Register Value: " << registerValue;
            }
        }

        socket.disconnectFromHost();
    }
    else
    {
        qDebug() << "Connection failed!";
    }
    return 0;
}
```

在上述示例中，我们使用 QTcpSocket 类与 Modbus 设备建立连接，并发送 Modbus TCP 读操作的请求命令。然后，通过解析接收到的响应数据来获取读取到的寄存器值。

需要注意的是，这只是一个简单的示例代码，实际情况下你可能需要根据 Modbus TCP 协议的规范，针对不同的读写操作进行编码和解码，并处理更复杂的数据传输和错误处理。确保了解 Modbus TCP 协议的细节和要求，并按照协议规范进行相应的操作。

另外，对于复杂的 Modbus 操作，包括写操作、多个寄存器的读取等，建议考虑使用 QModbusTcpClient 类或其他现有的 Modbus 库，它们已经实现了 Modbus TCP 协议的相关操作，并提供了更方便的接口和功能。



## ModBus 读写MX的点位

PLC中MX、MB、MW、MD的含义和长度：

M表示内部存储区。MB表示长度为字节的操作数在内部存储区，MW表示长度为字的操作数在内部存储区，MD表示长度为双字的操作数在内部存储区。
操作数包含两个要素：标识符Q和标识参数。标识符用来表示操作数存放区域及操作位数；标识参数用来表示操作数在该存储区域内的具体位置。
存储区域包括有：输入映像区()，输出映像区(Q),内部存储区(M),物理输入区(PI),物理输出区(PQ),数据块(DB),数据块(D),临时堆栈Q(L)
辅助标识符包括有：X(位)，B(字节)，W(字一2字节)，D(双字一4字节)

M表示是辅助存储单元Q
B是指长度占一个字节
W是指长度占一个字（两个字节）
D是指长度占一个双字（四个字节）

示例：

三个点位

```
int MW10000 
bool MX20020.2
float MD7521
```

MW10000转为HoldingRegisters 点位为 410001

MX20020.2 转为HoldingRegisters 点位为 410011 （20020除以2 + 400001），转为二进制，读写二进制的第二位值

MD7521 转为HoldingRegisters 点位为 415043 （7521 乘以2 + 40001）



MX20020.2 的写值步骤：

读取410011的当前值，如果需要在第二位写入1，可用下面方式：

```
quint16 anInt = 13; // 获取当前值
//将第3位写为1
anInt |= (1 << 3);
```

可将anInt 直接写入



果需要在第二位写入0，可用下面方式：

```
//将第3位写为0
anInt &= ~(1 << 3);
```

## 实际开发中的使用

### 创建连接池

.h 文件

```c++
#ifndef MODBUSTCPDEVICE_H
#define MODBUSTCPDEVICE_H


#include <QSqlDatabase>
#include <QSqlError>
#include <QSqlQuery>
#include <QQueue>
#include <QMutex>
#include <QDebug>
#include <QThread>
#include "config.h"
#include <QModbusTcpClient>
#include <QModbusDataUnit>
#include <QModbusTcpClient> // 如果使用TCP连接
#include <QModbusRtuSerialMaster> // 如果使用串口连接

class ModBusTcpDevice {
public:
    static ModBusTcpDevice &instance() {
        // 使用单例模式保证只创建一个连接池对象
        static ModBusTcpDevice pool;
        return pool;
    }

    QModbusTcpClient *getConnection() {
        // 获取连接
        m_mutex.lock();
        if (m_connections.isEmpty()) {
            addConnections(2);
        }
        QModbusTcpClient *modbusDevice = m_connections.dequeue();
        m_mutex.unlock();
        return modbusDevice;
    }

    QModbusTcpClient *getHcConnection() {
        // 获取连接
        hc_m_mutex.lock();
        if (hc_m_connections.isEmpty()) {
            addHcConnections(2);
        }
        QModbusTcpClient *modbusDevice = hc_m_connections.dequeue();
        hc_m_mutex.unlock();
        return modbusDevice;
    }


    void releaseConnection(QModbusTcpClient *modbusDevice) {
        // 释放连接
        m_mutex.lock();
        m_connections.enqueue(modbusDevice);
        m_mutex.unlock();
    }

private:
    ModBusTcpDevice() {
        // 初始化连接池
        addConnections(2);

        addHcConnections(2);
    }

    void addConnections(int count) {
        // 创建连接
        for (int i = 0; i < count; i++) {
            QModbusTcpClient *modbusDevice = new QModbusTcpClient();
            modbusDevice->setConnectionParameter(QModbusDevice::NetworkPortParameter, IP_PORT); // 设置端口号，如果使用TCP连接
            modbusDevice->setConnectionParameter(QModbusDevice::NetworkAddressParameter,
                                                 IP_ADDRESS); // 设置IP地址，如果使用TCP连接
            //设置超时时间
            modbusDevice->setTimeout(2000); //2秒
            //设置失败重试次数
            modbusDevice->setNumberOfRetries(-1);
            if (modbusDevice->connectDevice()) {
                m_connections.enqueue(modbusDevice);
            } else {
                qDebug() << "HS Failed to open database:";
            }
        }
    }

    void addHcConnections(int count) {
        // 创建连接
        for (int i = 0; i < count; i++) {
            QModbusTcpClient *modbusDevice = new QModbusTcpClient();
            modbusDevice->setConnectionParameter(QModbusDevice::NetworkPortParameter, IP_PORT); // 设置端口号，如果使用TCP连接
            modbusDevice->setConnectionParameter(QModbusDevice::NetworkAddressParameter,
                                                 HC_IP_ADDRESS); // 设置IP地址，如果使用TCP连接
            //设置超时时间
            modbusDevice->setTimeout(3000); //2秒
            //设置失败重试次数
            modbusDevice->setNumberOfRetries(3);
            if (modbusDevice->connectDevice()) {
                hc_m_connections.enqueue(modbusDevice);
            } else {
                qDebug() << "HC Failed to open database:";
            }
        }
    }

private:
    QMutex m_mutex;
    QQueue<QModbusTcpClient *> m_connections;

    QMutex hc_m_mutex;
    QQueue<QModbusTcpClient *> hc_m_connections;
};

#endif // MODBUSTCPDEVICE_H
```



### 读操作

.h 文件

```c++
#ifndef MODBUSREADERINT_H
#define MODBUSREADERINT_H

#include <QObject>
#include <QMap>
#include <QVariant>
#include <QTimer>
#include <QNetworkConfigurationManager>
#include <QNetworkSession>
#include <QModbusTcpClient>
#include "ModBusTcpDevice.h"
#include <timerthread.h>
#include <QFile>
#include <QJsonParseError>
#include <QJsonObject>
#include <QDateTime>
#include <QtConcurrent>

class ModBusReader : public QObject {
Q_OBJECT
public:
    explicit ModBusReader(QObject *parent = nullptr);

signals:

private:
    QTimer *ip_check_timer;
    bool boo_flag = false;
    QList<QModbusTcpClient *> clients;

//    bool connect_state = false;
//    QModbusTcpClient *modbusTcpClient = nullptr;

    void readQuint16(QModbusDataUnit dataUnit);

    void readFloat(QModbusDataUnit readUnit);

    void readInt32(QModbusDataUnit readUnit);

};

#endif // MODBUSREADERINT_H
```



.cpp 文件 

```c++
#include "modbusreader.h"

ModBusReader::ModBusReader(QObject *parent) : QObject(parent) {
    ip_check_timer = new QTimer();
    connect(ip_check_timer, &QTimer::timeout, [=]() {
        // 检查网络连接是否可用
        QNetworkConfigurationManager networkConfigManager;
        QNetworkConfiguration networkConfig = networkConfigManager.defaultConfiguration();
        QNetworkSession networkSession(networkConfig);
        if (networkSession.state() == QNetworkSession::Connected) {
            if (!boo_flag) {
                for (int j = 0; j < clients.size(); ++j) {
                    delete clients.at(j);
                }
                clients.clear();
                //创建100个客户端，为了提高读取效率
                for (int i = 0; i < 100; ++i) {
                    QModbusTcpClient *modbusTcpClient = ModBusTcpDevice::instance().getConnection();
                    connect(modbusTcpClient, &QModbusClient::stateChanged, [=]() {
                        if (modbusTcpClient->state() == QModbusDevice::ConnectedState) {
                            clients.append(modbusTcpClient);
                            boo_flag = true;
                        } else {
                            clients.removeAll(modbusTcpClient);
                            modbusTcpClient->connectDevice();
                        }
                    });
                }
            }
        } else {
            qDebug() << "HS 连接失败" << IP_ADDRESS;
            for (int j = 0; j < clients.size(); ++j) {
                delete clients.at(j);
            }
            clients.clear();
            boo_flag = false;
        }
    });
    ip_check_timer->start(1000);

    QFile file(":/files/point_1.json");
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) {
        return;
    }
    QByteArray data = file.readAll();
    QJsonParseError error;
    QJsonDocument jsonDoc = QJsonDocument::fromJson(data, &error);
    if (error.error != QJsonParseError::NoError) {
        return;
    }
    if (!jsonDoc.isObject()) {
        return;
    }
    QJsonObject rootObj = jsonDoc.object();
    QStringList keys = rootObj.keys();
    for (const auto &item : keys) {
        if (item.contains("A")) {
            const QList<QVariant> &toList = rootObj.value(item).toVariant().toList();
            QModbusDataUnit readUnit(QModbusDataUnit::HoldingRegisters, toList.at(0).toInt() - 400001, toList.size());
            readQuint16(readUnit);
        } else if (item.contains("B")) {
            const QList<QVariant> &toList = rootObj.value(item).toVariant().toList();
            QModbusDataUnit readUnit(QModbusDataUnit::HoldingRegisters, toList.at(0).toInt() - 400001,
                                     toList.size() * 2);
            readFloat(readUnit);
        } else if (item.contains("C")) {
            const QList<QVariant> &toList = rootObj.value(item).toVariant().toList();
            QModbusDataUnit readUnit(QModbusDataUnit::HoldingRegisters, toList.at(0).toInt() - 400001,
                                     toList.size() * 2);
            readInt32(readUnit);
        }
    }
}


//一个或多个点位读取
void ModBusReader::readQuint16(QModbusDataUnit dataUnit) {
    TimerThread *timerThread = new TimerThread(nullptr, 300);
    connect(timerThread, &TimerThread::timeout, this, [=]() {
        if (!clients.isEmpty()) {
            QModbusTcpClient *pClient = clients.takeAt(0);
            QModbusReply *pReply = pClient->sendReadRequest(dataUnit, 1);
            clients.append(pClient);
            if (!pReply->isFinished()) {
                connect(pReply, &QModbusReply::finished, [=]() {
                    if (pReply->error() == QModbusDevice::NoError) {
                        const QModbusDataUnit &dataUnit = pReply->result();
                        const QList<quint16> &list = dataUnit.values().toList();
                        int address = dataUnit.startAddress();
                        for (const auto &item : list) {
                            POINT_VALUE.insert(QString::number(address + 400001), QString::number(item));
                            address++;
                        }
                    }
                    pReply->deleteLater();
                });
            } else {
                delete pReply;
            }
        }

    });
    timerThread->start();
}

// Modbus无符号短整数转换为浮点数
float r_modbusToFloat(uint16_t highByte, uint16_t lowByte) {
    union {
        float floatValue;
        uint16_t ushortValue[2];
    } converter;

    // 将高字节和低字节合并
    converter.ushortValue[0] = highByte;
    converter.ushortValue[1] = lowByte;

    return converter.floatValue;
}

//一个或多个点位读取
void ModBusReader::readFloat(QModbusDataUnit dataUnit) {
    TimerThread *timerThread = new TimerThread(nullptr, 300);
    connect(timerThread, &TimerThread::timeout, this, [=]() {
        if (!clients.isEmpty()) {
            QModbusTcpClient *pClient = clients.takeAt(0);
            QModbusReply *pReply = pClient->sendReadRequest(dataUnit, 1);
            clients.append(pClient);
            if (!pReply->isFinished()) {
                connect(pReply, &QModbusReply::finished, [=]() {
                    if (pReply->error() == QModbusDevice::NoError) {
                        const QModbusDataUnit &dataUnit = pReply->result();
                        QList<quint16> qList = dataUnit.values().toList();
                        uint count = dataUnit.valueCount() / 2;
                        int address = dataUnit.startAddress();
                        for (int k = 0; k < count; ++k) {
                            quint16 x1 = qList.takeAt(0);
                            quint16 x2 = qList.takeAt(0);
                            float aFloat = r_modbusToFloat(x1, x2);
                            POINT_VALUE.insert(QString::number(address + 400001), QString::number(aFloat));
                            address += 2;
                        }
                    }
                    pReply->deleteLater();
                });
            } else {
                delete pReply;
            }
        }

    });
    timerThread->start();
}

//一个或多个点位读取
void ModBusReader::readInt32(QModbusDataUnit dataUnit) {
    TimerThread *timerThread = new TimerThread(nullptr, 300);
    connect(timerThread, &TimerThread::timeout, this, [=]() {
        if (!clients.isEmpty()) {
            QModbusTcpClient *pClient = clients.takeAt(0);
            QModbusReply *pReply = pClient->sendReadRequest(dataUnit, 1);
            clients.append(pClient);
            if (!pReply->isFinished()) {
                connect(pReply, &QModbusReply::finished, [=]() {
                    if (pReply->error() == QModbusDevice::NoError) {
                        const QModbusDataUnit &dataUnit = pReply->result();
                        const QVector<quint16> &vector = dataUnit.values();
                        if (!vector.isEmpty()) {
                            const QModbusDataUnit &dataUnit = pReply->result();
                            QList<quint16> qList = dataUnit.values().toList();
                            uint count = dataUnit.valueCount() / 2;
                            int address = dataUnit.startAddress();
                            for (int k = 0; k < count; ++k) {
                                int lowInt = qList.takeAt(0);
                                int highInt = qList.takeAt(0);
                                int dintValue = (highInt << 16) | lowInt;
                                POINT_VALUE.insert(QString::number(address + 400001), QString::number(dintValue));
                                address += 2;
                            }
                        }
                    }
                    pReply->deleteLater();
                });
            } else {
                delete pReply;
            }
        }
    });
    timerThread->start();
}
```



### 写操作

.h 文件

```c++
#ifndef MODBUSWRITE_H
#define MODBUSWRITE_H

#include <QObject>
#include <QMap>
#include <QVariant>
#include <QTimer>
#include <QNetworkConfigurationManager>
#include <QNetworkSession>
#include <QModbusTcpClient>
#include "ModBusTcpDevice.h"
#include <timerthread.h>

class ModBusWrite : public QObject {
Q_OBJECT
public:
    explicit ModBusWrite(QObject *parent = nullptr);

    static QVariantMap f_point_data;
    static QVariantMap i16_point_data;
    static QVariantMap i32_point_data;

    static void insetWrite(int point, quint16 data);

    static void insetWrite(int point, quint32 data);

    static void insetWrite(int point, float data);

signals:

private:
    QTimer *ip_check_timer;
    bool boo_flag = false;

    QList<QModbusTcpClient *> clients;

    void writeTimer();

    bool write(int point, quint16 data);

    bool write(int point, quint32 data);

    bool write(int point, float data);
};

#endif // MODBUSWRITE_H
```



.cpp 文件

```c++
#include <QEventLoop>
#include <baseserver.h>
#include "modbuswrite.h"


QVariantMap ModBusWrite::f_point_data = QVariantMap();
QVariantMap ModBusWrite::i16_point_data = QVariantMap();
QVariantMap ModBusWrite::i32_point_data = QVariantMap();

ModBusWrite::ModBusWrite(QObject *parent) : QObject(parent) {
    ip_check_timer = new QTimer();
    connect(ip_check_timer, &QTimer::timeout, [=]() {
        // 检查网络连接是否可用
        QNetworkConfigurationManager networkConfigManager;
        QNetworkConfiguration networkConfig = networkConfigManager.defaultConfiguration();
        QNetworkSession networkSession(networkConfig);
        if (networkSession.state() == QNetworkSession::Connected) {
            if (!boo_flag) {
                for (int j = 0; j < clients.size(); ++j) {
                    delete clients.at(j);
                }
                clients.clear();
                //创建100个客户端，为了提高读取效率
                for (int i = 0; i < 5; ++i) {
                    QModbusTcpClient *modbusTcpClient = ModBusTcpDevice::instance().getConnection();
                    connect(modbusTcpClient, &QModbusClient::stateChanged, [=]() {
                        if (modbusTcpClient->state() == QModbusDevice::ConnectedState) {
                            clients.append(modbusTcpClient);
                            boo_flag = true;
                        } else {
                            clients.removeAll(modbusTcpClient);
                            modbusTcpClient->connectDevice();
                        }
                    });
                }
            }
        } else {
            qDebug() << "HS 连接失败" << IP_ADDRESS;
            for (int j = 0; j < clients.size(); ++j) {
                delete clients.at(j);
            }
            clients.clear();
            boo_flag = false;
        }
    });
    ip_check_timer->start(1000);

    TimerThread *timerThread = new TimerThread(nullptr, 200);
    connect(timerThread, &TimerThread::timeout, this, [=]() {
        writeTimer();
    });
    timerThread->start();
}


void ModBusWrite::insetWrite(int point, quint16 data) {
    ModBusWrite::i16_point_data.insert(QString::number(point), data);
}

void ModBusWrite::insetWrite(int point, quint32 data) {
    ModBusWrite::i32_point_data.insert(QString::number(point), data);
}

void ModBusWrite::insetWrite(int point, float data) {
    ModBusWrite::f_point_data.insert(QString::number(point), data);
}


void ModBusWrite::writeTimer() {
    if (!f_point_data.isEmpty()) {
        const QString &firstkey = f_point_data.firstKey();
        int _point = firstkey.toInt();
        float f_data = f_point_data.value(firstkey).toFloat();
        f_point_data.remove(firstkey);
        write(_point, f_data);
    }
    if (!i16_point_data.isEmpty()) {
        const QString &firstkey = i16_point_data.firstKey();
        int _point = firstkey.toInt();
        quint16 i_data = i16_point_data.value(firstkey).toInt();
        i16_point_data.remove(firstkey);
        write(_point, i_data);
    }

    if (!i32_point_data.isEmpty()) {
        const QString &firstkey = i32_point_data.firstKey();
        int _point = firstkey.toInt();
        quint32 i_data = i32_point_data.value(firstkey).toInt();
        i32_point_data.remove(firstkey);
        write(_point, i_data);
    }
}


bool ModBusWrite::write(int point, quint16 data) {
    if (!clients.isEmpty()) {
        bool back = false;
        // 准备写入数据单元
        QModbusDataUnit writeUnit(QModbusDataUnit::HoldingRegisters, point - 400001, 1);
        // 设置要写入的浮点数值
        writeUnit.setValue(0, data);

        QModbusTcpClient *pClient = clients.takeAt(0);
        // 执行写操作
        if (auto *reply = pClient->sendWriteRequest(writeUnit, 1)) {  // 1是serverId
            if (!reply->isFinished()) {
                QEventLoop loop;
                QObject::connect(reply, &QModbusReply::finished, &loop, &QEventLoop::quit);
                loop.exec();
            }
            if (reply->error() == QModbusDevice::NoError) {
                BaseServer redis(CACHE_REDIS);
                redis.setCacheOne(QString::number(point), QString::number(data));
                qDebug() << "quint16,point=" << point << ",value=" << data << " write Successful";
                POINT_VALUE.insert(QString::number(point), data);
                back = true;
            } else {
                qDebug() << "quint16,point=" << point << ",Write operation failed:" << reply->errorString();
                i16_point_data.insert(QString::number(point), data);
            }
            reply->deleteLater();
        } else {
            qDebug() << "quint16,point=" << point << ",Failed to send write request";
            i16_point_data.insert(QString::number(point), data);
        }
        clients.append(pClient);
        return back;
    } else {
        i16_point_data.insert(QString::number(point), data);
        return false;
    }
}

// 浮点数转换为Modbus无符号短整数
void w_floatToModbus(float f, uint16_t &highByte, uint16_t &lowByte) {
    union {
        float floatValue;
        uint16_t ushortValue[2];
    } converter;

    converter.floatValue = f;
    // 高字节在前，低字节在后
    highByte = converter.ushortValue[0];
    lowByte = converter.ushortValue[1];
}

bool ModBusWrite::write(int point, float data) {
    if (!clients.isEmpty()) {
        bool back = false;
        // 准备写入数据单元
        QModbusDataUnit writeUnit(QModbusDataUnit::HoldingRegisters, point - 400001, 2);  // 写入两个寄存器
        uint16_t highByte, lowByte;
        w_floatToModbus(data, highByte, lowByte);
        // 设置要写入的浮点数值
        writeUnit.setValue(0, highByte);
        writeUnit.setValue(1, lowByte);

        QModbusTcpClient *pClient = clients.takeAt(0);
        // 执行写操作
        if (auto *reply = pClient->sendWriteRequest(writeUnit, 1)) {  // 1是serverId
            if (!reply->isFinished()) {
                QEventLoop loop;
                QObject::connect(reply, &QModbusReply::finished, &loop, &QEventLoop::quit);
                loop.exec();
            }
            if (reply->error() == QModbusDevice::NoError) {
                BaseServer redis(CACHE_REDIS);
                redis.setCacheOne(QString::number(point), QString::number(data));
                qDebug() << "float,point=" << point << ",value=" << data << " write Successful";
                POINT_VALUE.insert(QString::number(point), data);
                back = true;
            } else {
                qDebug() << "float,point=" << point << ",Write operation failed:" << reply->errorString();
                f_point_data.insert(QString::number(point), data);
            }
            clients.append(pClient);
            reply->deleteLater();
        } else {
            f_point_data.insert(QString::number(point), data);
        }
        return back;
    } else {
        f_point_data.insert(QString::number(point), data);
        return false;
    }
}


bool ModBusWrite::write(int point, quint32 data) {
    if (!clients.isEmpty()) {
        bool back = false;
        // 准备写入数据单元
        QModbusDataUnit writeUnit(QModbusDataUnit::HoldingRegisters, point - 400001, 2);
        // 设置要写入的浮点数值
        writeUnit.setValue(0, quint16(data));
        writeUnit.setValue(1, quint16(data >> 16));
        QModbusTcpClient *pClient = clients.takeAt(0);
        // 执行写操作
        if (auto *reply = pClient->sendWriteRequest(writeUnit, 1)) {  // 1是serverId
            if (!reply->isFinished()) {
                QEventLoop loop;
                QObject::connect(reply, &QModbusReply::finished, &loop, &QEventLoop::quit);
                loop.exec();
            }
            if (reply->error() == QModbusDevice::NoError) {
                BaseServer redis(CACHE_REDIS);
                redis.setCacheOne(QString::number(point), QString::number(data));
                qDebug() << "quint32,point=" << point << ",value=" << data << " write Successful";
                POINT_VALUE.insert(QString::number(point), data);
                back = true;
            } else {
                qDebug() << "quint32,point=" << point << ",Write operation failed:" << reply->errorString();
                i32_point_data.insert(QString::number(point), data);
            }
            clients.append(pClient);
            reply->deleteLater();
        } else {
            qDebug() << "quint32,point=" << point << ",Failed to send write request";
            i32_point_data.insert(QString::number(point), data);
        }
        return back;
    } else {
        i32_point_data.insert(QString::number(point), data);
        return false;
    }
}
```


