## ADB 远程控制

项目地址：

- https://github.com/Genymobile/scrcpy



启用 adb 无线调试：

1. 永久启用需要 Root，手机重启后依然有效，以下示例已在 Android 10 测试成功

   ```bash
   # 授权 shell 使用 root
   adb shell
   
   # 进入 shell 后切换为 root 用户
   su
   
   # 设置永久启用远程调试端口
   setprop persist.adb.tcp.port 5555
   
   # 检查
   getprop persist.adb.tcp.port
   
   # 如果 USB 连接失效，可使用以下方式切换为 USB
   setprop persist.adb.tcp.port ""
   ```

   >如果连接 USB 调试后，出现无法使用 TCP 连接 adb 时（表现为连接拒绝等，可能端口被关闭），可尝试重启手机恢复

adb 常用命令：

```bash
# 连接设备
adb connect 192.168.0.5:5555

# 查看设备
adb devices

# 断开连接
adb disconnect 192.168.0.5:5555

# 亮屏，亮屏执行即锁屏
adb shell input keyevent 26

# 密码解锁
adb shell input text 1234

# 获取电池信息
adb shell dumpsys battery
```

