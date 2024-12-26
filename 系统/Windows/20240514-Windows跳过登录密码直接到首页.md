Ctrl + R ，输入 `control userpasswords2`，进入下图

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLvED4MbUa8NsovrpXwicGqwynZVpaHog8ibqcI3G4eZWgvWyicG7Cqdy9wRAhnsHicKrdnRicKibOGiaahKQ/0?wx_fmt=png&from=appmsg) 

在有些系统中，可能会没有 *要使用本计算机，用户必须输入用户名和密码*  复选框，可使用下面脚本

将下面脚本，复制到一个txt文件下，然后将txt文件类型改为reg

```ABAP
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device]
"DevicePasswordLessBuildVersion"=dword:00000000
```

运行之后，再次Ctrl + R ，输入 `control userpasswords2`，再次进入 

如果出现 *要使用本计算机，用户必须输入用户名和密码*  复选框，指定用户取消这个复选框，然后应用，按步骤操作即可；

设置好后可重启计算机测试一下，是否可以跳过登录页面，直接进入首页；



如果脚本还是不行，需要去注册表里找到如下图：

![](https://mmbiz.qpic.cn/mmbiz_png/3eqXwttvOLttp1sg8zRMhfbZ9ichGZy0g4CXbSCBnJktnnzfeCsaib7MwR41GgDe3ps6GWREIo3cmJq4oNDlXI6w/0?wx_fmt=png&from=appmsg)

将AutoAdminLogon设置为1，然后 Windows + R，输入 control userpasswords2，看是否有 `要使用本计算机，用户必须输入用户名和密码` 复选框。