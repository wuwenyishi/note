此脚本使运行的exe程序只有一个运行，避免运行多个

```bash
@echo off

REM 设置 main.exe 的路径
set APP_PATH=E:\main_2024050901

REM 检查 main.exe 是否在运行
tasklist /FI "IMAGENAME eq main.exe" 2>NUL | find /I /N "main.exe">NUL
if "%ERRORLEVEL%"=="0" (
    echo main.exe is already running.
    exit
)

REM 启动 main.exe
cd  /d  "E:\main_2024050901"
start main.exe

REM 退出脚本
exit
```

