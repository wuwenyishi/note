ModbusTcp.h

```c++
#ifndef MODBUSTCP_H
#define MODBUSTCP_H

#include <QTimer>
#include <QQueue>
#include <QThread>
#include <QTcpSocket>
#include <QByteArray>
#include <QDebug>
#include <QObject>
#include <cmath>

#include <functional>
#include <iostream>

#include <QSharedPointer>

enum RType
{
    //MBAP报文头((适用于TCP))
    TAGMatterHi,      //事务标识符Hi 1byte
    TAGMatterLo,      //事务标识符Lo 1byte
    TAGProtocol,    //协议标识符 2byte
    NumLength,      //需要的长度 2byte(从本字节下一个到最后)
    TAGElement,     //单元标识符 1byte

    GetType,        //要数类型（功能码）1byte
    ByteLength,     //需要读取的数据字节个数
    ReadData,       //需要读取数据 2byte * n

};

struct MDBTData{
    //MBAP报文头(适用于TCP)
    quint8 TAGMatterHi;     //事务标识符Hi 1byte
    quint8 TAGMatterLo;     //事务标识符Lo 1byte
    quint16 TAGProtocol;    //协议标识符 2byte
    quint16 NumLength;      //需要的长度 2byte(从本字节下一个到最后)
    char TAGElement;        //单元标识符 1byte
    //PDU报文体
    char AddressCode;       //地址码(适用于RTU) 1byte
    char GetType;           //要数类型（功能码）1byte
    quint16 ByteLength;     //偏移量offset(读取数据的开始位置)，两个字节
    QByteArray ReadData;    //读取数量，两个字节
    quint16 Crc;            //crc校验(适用于RTU) 2byte
    MDBTData(){
        TAGMatterHi = 0;    //事务标识符Hi 1byte
        TAGMatterLo = 0;    //事务标识符Lo 1byte
        TAGProtocol = 0;    //协议标识符 2byte
        NumLength = 0;      //需要的长度 2byte(从本字节下一个到最后)
        TAGElement = 0;     //单元标识符 1byte
        //PDU报文体
        AddressCode = 0;    //地址码(适用于RTU) 1byte
        GetType = 0;        //要数类型（功能码）1byte
        ByteLength = 0;     //偏移量offset(读取数据的开始位置)，两个字节
        ReadData = QByteArray(2, 0);      //读取数量，两个字节
        Crc = 0;            //crc校验(适用于RTU) 2byte
    }
};

typedef struct Read_Value{
    QByteArray Read_Data;
    QByteArray Before_Data;
    std::function<void(QByteArray Arrary)> callback;
}RV_t;

//=============================================================

struct readData{
    float angle_set;
    float speed_set;
    float angle_pv;
    float speed_pv;
    float angle_pause;
    float jia_speed_set;
    float hand_speed_set;
    float weight;



    bool check1;
    bool check2;
    bool start;
    bool pause;

    bool xiaojing;

    short v1;
    short v2;
    short v3;
};


struct ReciveData8{
    short _model;
    short _time;
};

struct ReciveData10{
    short _Model_Flag;
    ushort _Scheduling_PV;
    ushort _Scheduling_SV;
    ushort _Timing_PV;
    ushort _Timing_SV;
};

struct _BagT{
    int PID_ENA_D_P;         // 15500 袋 p 开关
    int PID_ENA_D_I;         // 15501
    int PID_ENA_D_D;         // 15502
    int   Te_Start;         //温度启动  415438 int
    float Te_Z_PV;       	//袋温反馈值	415407	real
    float Te_Z_SV;      	//袋温设定值	415409	real
    float Te_Z_P;        	//袋温P值	415411	real
    float Te_Z_I;        	//袋温I值	415413	real
    float Te_Z_D;            //袋温D值	415415	real
    float Te_Z_Dead_z;   	//袋温正死区设定值	415417	real
    float Te_Z_Dead_f;   	//袋温负死区设定值	415419	real
    float Te_Z_Iqrd;         //袋温积分切入值	415421	real
    short Te_Z_HL_ENA;   	//袋温报警上限使能	415440	int
    float Te_Z_HL;           //袋温报警上限设定值	415441	real
    short Te_Z_LL_ENA;   	//袋温报警下限使能	415443	int
    float Te_Z_LL;           //袋温报警下限设定值	415445	real
    short Te_Z_HL_ALM;       //袋温上限报警	415444	int
    short Te_Z_LL_ALM;       //袋温下限报警	415447	int
    short Te_Z_OFF_Line; 	//袋温断线报警	415456	int
    float Te_Z_OP;           //袋温PID输出%（0-100）	415459	real
    float Te_Z_OFFEST;       //袋温零点补偿值	415465	real
    short Te_Z_OFFSET_ENA;	//袋温断线使能	415496	int
};

struct _HeatT{
    int PID_ENA_T_P;         // 15503 袋 p 开关
    int PID_ENA_T_I;         // 15504
    int PID_ENA_T_D;         // 15505
    float Te_F_PV;          //加热毯温度反馈值	415423	real
    float Te_F_SV;          //加热毯温度设定值	415425	real
    float Te_F_P;           //加热毯温度P值	415427	real
    float Te_F_I;           //加热毯温度I值	415429	real
    float Te_F_D;           //加热毯温度D值	415431	real
    float Te_F_Dead_z;      //加热毯温度正死区设定值	415433	real
    float Te_F_Dead_f;      //加热毯温度负死区设定值	415435	real
    float Te_F_Iqrd;        //加热毯温度积分切入值	415437	real
    short Te_F_HL_ENA;      //加热毯温度报警上限使能	415448	int
    float Te_F_HL;          //加热毯温度报警上限设定值	415449	real
    short Te_F_LL_ENA;      //加热毯温度报警下限使能	415451	int
    float Te_F_LL;          //加热毯温度报警下限设定值	415453	real
    short Te_F_HL_ALM;      //加热毯温度上限报警	415452	int
    short Te_F_LL_ALM;      //加热毯温度下限报警	415455	int
    short Te_F_OFF_Line;	//加热毯温度断线报警	415457	int
    float Te_F_OP;          //加热毯温度PID输出%（0-100）	415461	real
    float Te_F_OFFEST;      //加热毯温度零点补偿值	415467	real
    float Te_F_SV_LIMIT;	//加热毯输出保护设定值	415463	real
    short Te_F_OFFSET_ENA;	//加热毯断线使能	415497	int

};

struct _Weigh{
    float WT_PV;            //称重实际值                 415469	real
    float WT_LP_SP;         //称重两点校准第一点设定值	   415471	real
    float WT_HP_SP;         //称重两点校准第二点设定重	   415473	real
    float WT_1P_SP;         //称重单点校准设定重	   	415475	real
    float WT_HL_SP;         //称重上限设定值	   	415477	real
    float WT_LL_SP;         //称重下限设定值	415479	real
    short WT_LP_EN;      	//称重两点校准第一点使能	415481	int
    short WT_HP_EN;     	//称重两点校准第二点使能	415482	int
    short WT_1P_EN;     	//称重单点校准使能	415483	int
    short WT_1P_UNEN;    	//称重单点校准取消使能	415484	int
    short WT_HL_EN;     	//称重上限使能	415485	int
    short WT_LL_EN;     	//称重下限使能	415486	int
    short WT_ENAQP_EN;   	//称重去皮使能	415487	int
    short WT_UNENAQP_EN; 	//称重取消去皮使能	415488	int
    short WT_HL_AL;         //称重上限报警	415489	int
    short WT_LL_AL;     	//称重下限报警	415490	int
    short WT_OFFLINE_AL; 	//称重断线报警	415491	int
    short WT_LP_Sta;     	//称重两点校准第一点状态（0无 1成功 2失败）	415492	int
    short WT_HP_STA;     	//称重两点校准第二点状态（0无 1成功 2失败）	415493	int
    short WT_1P_STA;     	//称重单点校准状态（0无 1成功 2失败）	415494	int
    short WT_ENAQP;      	//称重去皮状态（0无 1成功 2失败）	415495	int
    short WT_OFFLINE_ENA;	//称重断线使能	415498	int
};





#define MdbsTCP ModbusTcp::returnMTCP()

class ModbusTcp : public QObject
{
Q_OBJECT
public:
    ModbusTcp(QObject *parent = nullptr);
    static ModbusTcp *returnMTCP();

    QByteArray floatBigendian(float input);
    QByteArray shortBigendian(short input);
    float floatFromBigEndian(QByteArray data);
    short shortFromBigEndian(const QByteArray &data);
    ushort ushortFromBigEndian(const QByteArray &data);
    long longFromBigEndian(const QByteArray &data);

    short toShort(QByteArray ary);
    float toFloat(QByteArray ary);
    QByteArray floatToByte(float input);
    QByteArray shortToByte(short input);
    QByteArray doubleBigendian(double input);


    void Init_QTcpSocket();
//==========================================
    void Set_JiTing(bool flag);         //.3
    void Set_GuanGuan(bool flag);       //.4
    void Set_XiaoJing(bool flag);       // .5


    void Manage_10000(QByteArray ary);
    void Manage_10125(QByteArray ary);
    void Manage_10250(QByteArray ary);
    void Manage_10375(QByteArray ary);
    void Manage_10500(QByteArray ary);
    void Manage_10625(QByteArray ary);
    void Manage_10750(QByteArray ary);
    void Manage_10875(QByteArray ary);
    void Manage_15000(QByteArray ary);
    void Manage_15125(QByteArray ary);
    void Manage_15250(QByteArray ary);
    void Manage_15375(QByteArray ary);
    void Manage_15500(QByteArray ary);
    void Manage_15625(QByteArray ary);
    void Manage_15750(QByteArray ary);
    void Manage_15875(QByteArray ary);
    void Manage_16000(QByteArray ary);
    void Manage_16125(QByteArray ary);
    void Manage_16250(QByteArray ary);
    void Manage_16375(QByteArray ary);
    void Manage_16500(QByteArray ary);
    void Manage_16625(QByteArray ary);
    void Manage_16750(QByteArray ary);
    void Manage_16875(QByteArray ary);
    void Manage_20000(QByteArray ary);
    void Manage_20125(QByteArray ary);



    _BagT BagT;
    _HeatT HeatT;
    _Weigh Weigh;

    readData Read_Data_15002;
    ReciveData8 Read_Data_10000;
    ReciveData10 Read_Data_30800;


    QByteArray alarm_arrary_10010;            // 报警点位
    QByteArray xiaojing_array;                // 消警点位
    QByteArray start_array;                   // 启停点位    30064.3
    QByteArray reset_array;                   // 复位点位
    QByteArray zero_array;                    // 零点点位 10019.1
    QByteArray sifu_array;                    // 伺服点位 1000


    void Init_Read();

    void
    Init_Read(QByteArray Array,int ids,int stw);
//===========================================
    bool Get_Connect();

    bool Write(char feature, ushort startAdiss, QByteArray array, ushort sized = 0,quint8 tAGMatterHi=0xff, quint8 tAGMatterLo=0xff);

    void Write_Bool(char feature, ushort startAdiss,QByteArray Ary,int offset,bool Flag);

    bool Init_Read(char feature,ushort startAdiss,quint16 number,std::function<void(QByteArray Arrary)> callback);

    void Execute_Callback(QByteArray Data);

    QByteArray Get_QByteArray(char feature, ushort startAdiss, QByteArray & array, ushort sized = 0,quint8 tAGMatterHi=0, quint8 tAGMatterLo=0);

    void Start_Read(int Sec=100);

    void Start_Queue(int Sec=20);

    void Thread_Rcv();

    void Set_IP(QString ip){_ip = ip;}

public slots:
    bool QByte_ReadData();

    bool QByte_WriteData();

    void ReadError(QAbstractSocket::SocketError);

public:

    QString _ip;

    QTimer * _timing;

    QTimer * _timing_map;

    QTcpSocket * _tcpClient;


    QQueue<QByteArray> * _wirteMsg;

    bool _client_flag;

    QByteArray _buffer;  //用于存储

    RType _read_index;     //用于确认标识

    int _toReadlen;      //用于确认需要读取的长度

    QMap<quint16,RV_t>  Modbus_TCP_Read_Map;

    QSharedPointer<MDBTData> code;
};



#endif // MODBUSTCP_H

```



ModbusTcp.cpp

```c++
#include <globalconstant.h>
#include <global_api.h>
#include "modbustcp.h"

ModbusTcp::ModbusTcp(QObject *parent) : QObject(parent)
{
    _client_flag = false;
    _ip = "172.16.13.10";
//    _ip = "172.22.80.1";

    _wirteMsg = new QQueue<QByteArray>();

    _read_index = TAGMatterHi;

    _tcpClient = new QTcpSocket();
    connect(_tcpClient, SIGNAL(readyRead()), this, SLOT(QByte_ReadData()));
    connect(_tcpClient, SIGNAL(error(QAbstractSocket::SocketError)), this, SLOT(ReadError(QAbstractSocket::SocketError)));

    _timing = new QTimer(this);
    connect(_timing,&QTimer::timeout, this,&ModbusTcp::QByte_WriteData);


    _timing_map = new QTimer(this);
    connect(_timing_map,&QTimer::timeout, this,[=](){
        for(auto it = Modbus_TCP_Read_Map.begin() ;it != Modbus_TCP_Read_Map.end(); it++){
            _wirteMsg->enqueue(it.value().Read_Data);
        }
    });

    code = QSharedPointer<MDBTData>(new MDBTData());

}

ModbusTcp *ModbusTcp::returnMTCP()
{
    static ModbusTcp tcp;
    return &tcp;
}

bool ModbusTcp::Get_Connect()
{

    if(_tcpClient->waitForConnected(300))       //会阻塞函数直到新的数据出现
    {
        _client_flag = true;
        return true;
    }

    _tcpClient->abort();                    //取消原有连接
    _tcpClient->connectToHost(_ip,502);
    _client_flag = _tcpClient->waitForConnected(1000);

    qDebug() << _ip << "2通讯状态::"  << _client_flag;
    return _client_flag;
}

void ModbusTcp::Set_JiTing(bool flag)
{
    if(Get_Connect() == false){return;}

    QByteArray data;
    if(flag){
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) | 0x8);
    }else{
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) & 0xF7);
    }
    Write(0x06,0x3c29,data);
}

void ModbusTcp::Set_GuanGuan(bool flag)
{
    if(Get_Connect() == false){return;}

    QByteArray data;
    if(flag){
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) | 0x10);
    }else{
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) & 0xEF);
    }
    Write(0x06,0x3c29,data);
}

void ModbusTcp::Set_XiaoJing(bool flag)
{
    if(Get_Connect() == false){return;}

    QByteArray data;
    if(flag){
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) | 0x20);
    }else{
        data.append(MdbsTCP->xiaojing_array.at(0));
        data.append(MdbsTCP->xiaojing_array.at(1) & 0xDF);
    }
    Write(0x06,0x3c29,data);
}


bool ModbusTcp::Write(char feature, ushort startAdiss, QByteArray array, ushort sized, quint8 tAGMatterHi, quint8 tAGMatterLo)
{
    if(_timing->isActive()){

    }else{
        Start_Queue();
    }

    QByteArray ary = Get_QByteArray(feature,startAdiss,array,sized,tAGMatterHi,tAGMatterLo);

    if(ary.isEmpty()){
        return false;
    }else{
        _wirteMsg->enqueue(ary);
    }

}

void ModbusTcp::Write_Bool(char feature, ushort startAdiss, QByteArray Ary,int offset, bool Flag)
{

    QByteArray data;
    char c = Ary[0];
    c = c | 0x00;
    data.append(c);
    char h = Flag ? pow(2, offset) : (255 - pow(2, offset));
    h = h | Ary[1];
    data.append(h);

    Write(feature,startAdiss,data);
}

bool ModbusTcp::Init_Read(char feature, ushort startAdiss, quint16 number, std::function<void (QByteArray)> callback)
{
    static quint16 Index = 256;

    QByteArray array;
    array.append(static_cast<char>((number >> 8) & 0xFF));
    array.append(static_cast<char>(number & 0xFF));
    QByteArray ary = Get_QByteArray(feature,startAdiss,array,0x00,static_cast<char>((Index >> 8) & 0xFF),static_cast<char>(Index & 0xFF));
    if(ary.isEmpty()){return false;}


    RV_t Read_Value;
    Read_Value.Read_Data = ary;
    Read_Value.callback = callback;

    Modbus_TCP_Read_Map.insert(Index,Read_Value);
    Index++;
    return true;
}

void ModbusTcp::Execute_Callback(QByteArray Data)
{
    quint16 Index = Data.mid(0,2).toHex().toUShort(nullptr,16);   //头
    quint16 fcd = Data.mid(7,1).toHex().toUShort(nullptr,16);  //功能码

//    qDebug()<<"Size::"<<Data.size() << "Data::" << Data;
//    qDebug() << Index << "头" << fcd << "功能码";

    if(Modbus_TCP_Read_Map.contains(Index)){
        if(Modbus_TCP_Read_Map[Index].Before_Data != Data){
            Modbus_TCP_Read_Map[Index].callback(Data);
            Modbus_TCP_Read_Map[Index].Before_Data = Data;
        }
    }

}

QByteArray ModbusTcp::Get_QByteArray(char feature, ushort startAdiss, QByteArray &array, ushort sized, quint8 tAGMatterHi, quint8 tAGMatterLo)
{
    ushort lenth = static_cast<ushort>(array.size());

    if( lenth < 2) return "";

//    QSharedPointer<MDBTData> code = QSharedPointer<MDBTData>(new MDBTData());

//    code->ReadData.clear();
    code->ReadData = { };
    code->TAGMatterHi = tAGMatterHi;
    code->TAGMatterLo = tAGMatterLo;
    code->TAGProtocol = 0x0000;
    code->TAGElement = 0x01;



    switch (feature) {
        case 0x01:
        {//读线圈
            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x01;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//读输入寄存器数量，2个字节
        }
            break;
        case 0x02:
        {//读离散输入
            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x02;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//读输入寄存器数量，2个字节
        }
            break;
        case 0x03:
        {//读保持寄存器


            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x03;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//读输入寄存器数量，2个字节      26个
        }
            break;
        case 0x04:
        {//读输入寄存器
            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x04;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//读输入寄存器数量，2个字节
        }
            break;
        case 0x05:
        {//写单个线圈
            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x05;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//写入数据
        }
            break;
        case 0x06:
        {//写单个保持寄存器
            code->NumLength = (0x06); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x06;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(array,2);//保持寄存器数据，n个字节
        }
            break;
        case 0x0f:
        {//写多个线圈
            code->NumLength = (0x07 + lenth); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x0f;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte
            code->ReadData.append(sized);//要写入的数量，两个字节
            code->ReadData.append(lenth);//数据长度(字节数)，一个字节
            code->ReadData.append(array,lenth);//保持寄存器数据，n个字节
        }
            break;
        case 0x10:
        {//写多个保持寄存器(此处的寄存器地址必须是连续的)
            code->NumLength = (0x07 + lenth); //需要的长度 2byte(从本字节下一个到最后)
            code->GetType = 0x10;//要数类型（功能码）1byte
            code->ByteLength = startAdiss;//读取数据的开始位置2byte <--
            ushort l = lenth / 2;
            code->ReadData.append(l>>8);//要写入的数量，两个字节h
            code->ReadData.append(l);//要写入的数量，两个字节l
            code->ReadData.append(lenth);//数据长度(字节数)，一个字节
            code->ReadData.append(array,lenth);//保持寄存器数据，n个字节
        }
            break;
        default:
            qDebug()<<"erre: 功能码 不存在";
            return "";
    }

    // =======================================
    QByteArray ary;
    ary.append(code->TAGMatterHi);
    ary.append(code->TAGMatterLo);

    ary.append((code->TAGProtocol & 0xFF00)>>8);
    ary.append((code->TAGProtocol & 0x00FF));

    ary.append((code->NumLength & 0xFF00)>>8);
    ary.append((code->NumLength & 0x00FF));

    ary.append(code->TAGElement);

    ary.append(code->GetType);

    ary.append((code->ByteLength & 0xFF00)>>8);
    ary.append((code->ByteLength & 0x00FF));


    ary.append(code->ReadData);

    if(!ary.isEmpty()) {
        return ary;
    }
    return "";
}

void ModbusTcp::Start_Read(int Sec)
{
    _timing_map->start(Sec);
}

void ModbusTcp::Start_Queue(int Sec)
{
    _timing->start(Sec);
}

void ModbusTcp::Thread_Rcv()
{
    Start_Queue();
    Start_Read();
}

bool ModbusTcp::QByte_ReadData()
{
    QByteArray Data = _tcpClient->readAll();

    if(!Data.isEmpty()){
        int index = 0;
        int netNum = 0;
        while(index < Data.size())
        {
            const char Char = Data.at(index);

            switch (_read_index){
                case TAGMatterHi: {
                    ++ index;
                    if (Char){
                        if(!_buffer.isEmpty()){_buffer.clear();}
                        _buffer.append(Char);
                        _read_index = TAGMatterLo;
                    }
                    continue;
                }
                case TAGMatterLo: {
                    _buffer.append(Char);
                    index ++;
                    _read_index = TAGProtocol;
                    continue;
                }
                case TAGProtocol: {
                    ++ index;
                    ++ netNum;
                    _buffer.append(Char);
                    if(netNum == 2){netNum = 0;_read_index = NumLength;}
                    continue;
                }
                case NumLength : {
                    ++ index;
                    ++ netNum;
                    _buffer.append(Char);
                    if(netNum == 2){netNum = 0;_read_index = TAGElement;}
                    continue;
                }
                case TAGElement: {
                    _buffer.append(Char);
                    index ++;
                    _read_index = GetType;
                    continue;
                }
                case GetType: {
                    _buffer.append(Char);
                    index ++;
                    _read_index = ByteLength;
                    continue;
                }
                case ByteLength: {
                    _buffer.append(Char);
                    index ++;
                    _toReadlen = static_cast<uchar>(Char);
                    _read_index = ReadData;
                    continue;
                }
                case ReadData: {
                    const int Count = Data.length() - index;

                    if (Count >= _toReadlen){
                        _buffer.append(Data.mid(index, _toReadlen));
                        index += _toReadlen;
                        _toReadlen = 0;
                        _read_index = TAGMatterHi;  //先处理再清空
                        Execute_Callback(_buffer);
                        _buffer.clear();
                        continue;
                    }else{
                        _buffer.append(Data.mid(index, Count));
                        index += Count;
                        _toReadlen -= Count;
                    }
                    continue;
                }
            }

        }
        return true;
    }
    return false;
}

bool ModbusTcp::QByte_WriteData()
{
    if(_wirteMsg->isEmpty() || !Get_Connect()){return false;}

    return _tcpClient->write(_wirteMsg->dequeue());
}

void ModbusTcp::ReadError(QAbstractSocket::SocketError)
{
    _tcpClient->disconnectFromHost();
}


//float 转字节数组
QByteArray ModbusTcp::floatBigendian(float input)
{
    QByteArray data;
    char p[4], q[4];
    memcpy(p, &input, sizeof(input));    //cd ab   abcd bcda    0123  2301 3201   3210 abcd   1032    cdab
    q[0] = p[1];
    q[1] = p[0];
    q[2] = p[3];
    q[3] = p[2];
    data.append(QByteArray(q,4));
    return data;
}

QByteArray ModbusTcp::doubleBigendian(double input)
{
    QByteArray data;
    char p[4], q[4];
    memcpy(p, &input, sizeof(input));
    q[0] = p[7];
    q[1] = p[6];
    q[2] = p[5];
    q[3] = p[4];
    q[4] = p[3];
    q[5] = p[2];
    q[6] = p[1];
    q[7] = p[0];
    data.append(QByteArray(q,8));
    return data;
}

void ModbusTcp::Init_QTcpSocket()
{
    if(_tcpClient == nullptr){
        _tcpClient = new QTcpSocket();
        connect(_tcpClient, SIGNAL(readyRead()), this, SLOT(QByte_ReadData()));
        connect(_tcpClient, SIGNAL(error(QAbstractSocket::SocketError)), this, SLOT(ReadError(QAbstractSocket::SocketError)));
    }

}

void ModbusTcp::Init_Read()
{

    Init_Read(0x03,10000,125,[=](QByteArray ary){MdbsTCP->Manage_10000(ary);});
    Init_Read(0x03,10125,125,[=](QByteArray ary){MdbsTCP->Manage_10125(ary);});
    Init_Read(0x03,10250,125,[=](QByteArray ary){MdbsTCP->Manage_10250(ary);});
    Init_Read(0x03,10375,125,[=](QByteArray ary){MdbsTCP->Manage_10375(ary);});
    Init_Read(0x03,10500,125,[=](QByteArray ary){MdbsTCP->Manage_10500(ary);});
    Init_Read(0x03,10625,125,[=](QByteArray ary){MdbsTCP->Manage_10625(ary);});

    Init_Read(0x03,15000,122,[=](QByteArray ary){MdbsTCP->Manage_15000(ary);});
    Init_Read(0x03,15122,122,[=](QByteArray ary){MdbsTCP->Manage_15125(ary);});
    Init_Read(0x03,15244,122,[=](QByteArray ary){MdbsTCP->Manage_15250(ary);});
    Init_Read(0x03,15366,122,[=](QByteArray ary){MdbsTCP->Manage_15375(ary);});
    Init_Read(0x03,15488,122,[=](QByteArray ary){MdbsTCP->Manage_15500(ary);});
    Init_Read(0x03,15610,122,[=](QByteArray ary){MdbsTCP->Manage_15625(ary);});
    Init_Read(0x03,15732,122,[=](QByteArray ary){MdbsTCP->Manage_15750(ary);});
    Init_Read(0x03,15854,122,[=](QByteArray ary){MdbsTCP->Manage_15875(ary);});

    Init_Read(0x03,16000,122,[=](QByteArray ary){MdbsTCP->Manage_16000(ary);});
    Init_Read(0x03,16122,122,[=](QByteArray ary){MdbsTCP->Manage_16125(ary);});
    Init_Read(0x03,16244,122,[=](QByteArray ary){MdbsTCP->Manage_16250(ary);});
    Init_Read(0x03,16366,122,[=](QByteArray ary){MdbsTCP->Manage_16375(ary);});
    Init_Read(0x03,16488,122,[=](QByteArray ary){MdbsTCP->Manage_16500(ary);});
    Init_Read(0x03,16610,122,[=](QByteArray ary){MdbsTCP->Manage_16625(ary);});
    Init_Read(0x03,16732,122,[=](QByteArray ary){MdbsTCP->Manage_16750(ary);});
    Init_Read(0x03,16854,122,[=](QByteArray ary){MdbsTCP->Manage_16875(ary);});

    Init_Read(0x03,20000,122,[=](QByteArray ary){MdbsTCP->Manage_20000(ary);});
    Init_Read(0x03,20122,60,[=](QByteArray ary){MdbsTCP->Manage_20125(ary);});
}

void ModbusTcp::Manage_10000(QByteArray ary){
//    Init_Read(ary,10000,9);

    int startPoint = 10000;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(shortFromBigEndian(ary.mid(startIndex,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(floatFromBigEndian(ary.mid(startIndex,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(longFromBigEndian(ary.mid(startIndex,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10125(QByteArray ary){
//    Init_Read(ary,10125,9);

    int startPoint = 10125;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10250(QByteArray ary){
//    Init_Read(ary,10250,9);

    int startPoint = 10250;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10375(QByteArray ary){
//    Init_Read(ary,10375,9);

    int startPoint = 10375;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10500(QByteArray ary){
//    Init_Read(ary,10500,9);

    int startPoint = 10500;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10625(QByteArray ary){
//    Init_Read(ary,10625,9);

    int startPoint = 10625;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10750(QByteArray ary){
//    Init_Read(ary,10750,9);

    int startPoint = 10750;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_10875(QByteArray ary){
//    Init_Read(ary,10875,9);

    int startPoint = 10875;
    int startIndex = 9;
    for (int i = 0; i < 125; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15000(QByteArray ary){
//    Init_Read(ary,15000,9);

    int startPoint = 15000;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15125(QByteArray ary){
//    Init_Read(ary,15122,9);

    int startPoint = 15122;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15250(QByteArray ary){
//    Init_Read(ary,15244,9);

    int startPoint = 15244;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15375(QByteArray ary){
//    Init_Read(ary,15366,9);

    int startPoint = 15366;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15500(QByteArray ary){
//    Init_Read(ary,15488,9);

    int startPoint = 15488;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15625(QByteArray ary){
//    Init_Read(ary,15610,9);

    int startPoint = 15610;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(shortFromBigEndian(ary.mid(startIndex,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(floatFromBigEndian(ary.mid(startIndex,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(startPoint + 400001) << ":" << QString::number(longFromBigEndian(ary.mid(startIndex,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15750(QByteArray ary){
//    Init_Read(ary,15732,9);

    int startPoint = 15732;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_15875(QByteArray ary){
//    Init_Read(ary,15854,9);

    int startPoint = 15854;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16000(QByteArray ary){
//    Init_Read(ary,16000,9);

    int startPoint = 16000;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16125(QByteArray ary){
//    Init_Read(ary,16122,9);

    int startPoint = 16122;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16250(QByteArray ary){
//    Init_Read(ary,16244,9);

    int startPoint = 16244;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16375(QByteArray ary){
//    Init_Read(ary,16366,9);

    int startPoint = 16366;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16500(QByteArray ary){
//    Init_Read(ary,16488,9);

    int startPoint = 16488;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16625(QByteArray ary){
//    Init_Read(ary,16610,9);

    int startPoint = 16610;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16750(QByteArray ary){
//    Init_Read(ary,16732,9);

    int startPoint = 16732;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
};
void ModbusTcp::Manage_16875(QByteArray ary){
//    Init_Read(ary,16854,9);

    int startPoint = 16854;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
}
void ModbusTcp::Manage_20000(QByteArray ary){
//    Init_Read(ary,20000,9);

    int startPoint = 20000;
    int startIndex = 9;
    for (int i = 0; i < 122; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
}
void ModbusTcp::Manage_20125(QByteArray ary){
    int startPoint = 20122;
    int startIndex = 9;
    for (int i = 0; i < 60; ++i) {
        if (startIndex + 2 >= ary.size()){
            break;
        }

        if (_POINT_TYPE.contains(startPoint + 400001)){
            int l = _POINT_TYPE.value(startPoint + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(shortFromBigEndian(ary.mid(startIndex,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                startIndex = startIndex + 2;
                startPoint = startPoint + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(floatFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(startPoint + 400001), QString::number(longFromBigEndian(ary.mid(startIndex,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                startIndex = startIndex + 4;
                startPoint = startPoint + 2;
            }else{
                startPoint = startPoint + 1;
            }
        } else{
            startPoint = startPoint + 1;
            startIndex = startIndex + 2;
        }
    }
}

void ModbusTcp::Init_Read(QByteArray Array,int ids,int stw)
{
    for (int i = 0; i < 125; ++i) {
        if (stw + 2 >= Array.size()){
            break;
        }
        if (_POINT_TYPE.contains(ids + 400001)){
            int l = _POINT_TYPE.value(ids + 400001);
            if (l == 1){
                GLOBAL->point_value.insert(QString::number(ids + 400001), QString::number(shortFromBigEndian(Array.mid(stw,2))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(shortFromBigEndian(Array.mid(stw,2)));
                stw = stw + 2;
                ids = ids + 1;
            }
            else if (l == 2){
                GLOBAL->point_value.insert(QString::number(ids + 400001), QString::number(floatFromBigEndian(Array.mid(stw,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(floatFromBigEndian(Array.mid(stw,4)));
                stw = stw + 4;
                ids = ids + 2;
            }
            else  if (l == 3){
                GLOBAL->point_value.insert(QString::number(ids + 400001), QString::number(longFromBigEndian(Array.mid(stw,4))));
//                qDebug() << QString::number(ids + 400001) << ":" << QString::number(longFromBigEndian(Array.mid(stw,4)));
                stw = stw + 4;
                ids = ids + 2;
            }else{
                ids = ids + 1;
            }
        } else{
            ids = ids + 1;
            stw = stw + 2;
        }
    }
}




QByteArray ModbusTcp::shortBigendian(short input)
{
    QByteArray data;
    char p[2], q[2];
    memcpy(p, &input, sizeof(input));
    q[0] = p[1];
    q[1] = p[0];
    data.append(QByteArray(q,2));
    return data;
}
//字节数组转 float// huichuan shun xu bu dui
float ModbusTcp::floatFromBigEndian(QByteArray data)
{
    if (data.size() < 4) {
        return 0.0f;
    }

    // 使用CDAB格式将4个字节拼接为32位整数
    quint32 intRepresentation =
            (static_cast<quint8>(data[2]) << 24) |  // C -> 高位
            (static_cast<quint8>(data[3]) << 16) |  // D
            (static_cast<quint8>(data[0]) << 8)  |  // A
            (static_cast<quint8>(data[1]));         // B -> 低位

    // 将32位整数转换为float
    float result;
    memcpy(&result, &intRepresentation, sizeof(result));
    return result;
}


short ModbusTcp::shortFromBigEndian(const QByteArray &data)
{
    if (data.size() < 2){
        return 0;
    }
    // 使用静态转换和移位操作来替换 memcpy
    short _short = static_cast<short>((static_cast<quint8>(data[0]) << 8) | static_cast<quint8>(data[1]));
    return _short;
}

ushort ModbusTcp::ushortFromBigEndian(const QByteArray &data)
{
    char	 q[2];
    ushort	_short;
    q[0] = data[1];
    q[1] = data[0];
    memcpy(&_short, q, sizeof(ushort));
    return _short;
}

long ModbusTcp::longFromBigEndian(const QByteArray &data)
{
    if (data.size() < 4) {
        return 0;
    }

    // 使用移位和按位或将4个字节拼接为32位整数
    // 使用CDAB格式将4个字节拼接为32位整数
    qint32 result =
            (static_cast<quint8>(data[2]) << 24) |  // C -> 高位
            (static_cast<quint8>(data[3]) << 16) |  // D
            (static_cast<quint8>(data[0]) << 8)  |  // A
            (static_cast<quint8>(data[1]));         // B -> 低位


    return result;
}


short ModbusTcp::toShort(QByteArray ary)
{
    if(ary.size() < 2) return 0;
    int b = ary[0];
    short a = 0;
    a =  a | (b<<8);
    b = ary[1];
    a =  a | b;

    return a;
}

float ModbusTcp::toFloat(QByteArray ary)
{
    char	 q[4];
    float	_float;
    q[0] = ary[0];
    q[1] = ary[1];
    q[2] = ary[2];
    q[3] = ary[3];
    memcpy(&_float, q, sizeof(float));
    return _float;
}

QByteArray ModbusTcp::floatToByte(float input)
{
    QByteArray data;

    char p[4], q[4];
    memcpy(p, &input, sizeof(input));
    q[0] = p[0];
    q[1] = p[1];
    q[2] = p[2];
    q[3] = p[3];
    data.append(QByteArray(q,4));

    return data;
}
QByteArray ModbusTcp::shortToByte(short input)
{
    QByteArray data;

    char p[2], q[2];
    memcpy(p, &input, sizeof(input));
    q[0] = p[0];
    q[1] = p[1];
    data.append(QByteArray(q,2));

    return data;
}
```

