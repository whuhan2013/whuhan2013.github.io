---
layout: post
title: adb常用操作
date: 2016-10-12
categories: blog
tags: [android]
description: adb常用
---


**adb运行automation测试**  

```
adb shell am instrument -w -r   -e debug false -e class com.rock.myapplication.RunnerTest com.rock.myapplication.test/android.support.test.runner.AndroidJUnitRunner
```

参考：[Android, Error=Unable to find instrumentation info for: ComponentInfo](http://stackoverflow.com/questions/36753486/android-error-unable-to-find-instrumentation-info-for-componentinfo)

**Android获取包名的方法**

```
aapt dump badging ~/temp/automattest/apk/app-internal-release-3.6.21.apk 
```

参考：[Android获取包名的方法](http://blog.sina.com.cn/s/blog_5b478f870102v5bo.html)


**如何用adb启动apk**

```
adb shell am start -n breakan.test/breakan.test.TestActivity
```

**adb截图并发送到电脑** 

```
adb shell /system/bin/screencap -p /sdcard/screenshot.png（保存到SDCard）
adb pull /sdcard/screenshot.png d:/screenshot.png（保存到电脑）
```

**adb卸装与安装apk**

```
adb uninstall com.energymost.rocking

adb install -r app-internal-release-3.6.21.apk

在shell中运行
pm install /sdcard/com.schuser.showlog/apk/app-internal-release-3.6.21.apk
pm uninstall com.energymost.rocking

```

**参考链接：**[Android中实现静态的默认安装和卸载应用](http://blog.csdn.net/jiangwei0910410003/article/details/36427963)


**adb指定设备**        

1. 通过adb devices命令获取所有online设备的serial number。    

```
C:\Users\Administrator>adb devices
List of devices attached
emulator-5554   device
SH0A6PL00243    device
```

上面表示，当前有两个设备online，第一个emulator-5554是模拟器，后一个是真机会SH0A6PL00243。

2. 通过adb -s <serial number> cmd向设备发送adb命令。

```
比如：运行命令shell。
C:\Users\Administrator>adb -s SH0A6PL00243 shell
```




**利用android代码静默安装卸载**      

```
public static boolean RootCommand(String command){  
        Process process = null;  
        DataOutputStream os = null;  
        try{  
            process = Runtime.getRuntime().exec("su");  
            os = new DataOutputStream(process.getOutputStream());  
            os.writeBytes(command + "\n");  
            os.writeBytes("exit\n");  
            os.flush();  
            process.waitFor();  
        } catch (Exception e){  
            Log.d("*** DEBUG ***", "ROOT REE" + e.getMessage());  
            return false;  
        } finally{  
            try{  
                if (os != null){  
                    os.close();  
                }  
                process.destroy();  
            } catch (Exception e){  
            }  
        }  
        Log.d("*** DEBUG ***", "Root SUC ");  
        return true;  
    }  



//调用

new Thread(){  
    @Override  
    public void run(){  
        RootCommand("pm uninstall com.tencent.mm");  
    }  
}.start();  
```

**android运行adb命令**       
注意需要获得root权限

```
public class CommandExecution {
    public static final String TAG = "CommandExecution";

    public final static String COMMAND_SU       = "su";
    public final static String COMMAND_SH       = "sh";
    public final static String COMMAND_EXIT     = "exit\n";
    public final static String COMMAND_LINE_END = "\n";

    /**
     * Command执行结果
     * @author Mountain
     *
     */
    public static class CommandResult {
        public int result = -1;
        public String errorMsg;
        public String successMsg;
    }

    /**
     * 执行命令—单条
     * @param command
     * @param isRoot
     * @return
     */
    public static CommandResult execCommand(String command, boolean isRoot) {
        String[] commands = {command};
        return execCommand(commands, isRoot);
    }

    /**
     * 执行命令-多条
     * @param commands
     * @param isRoot
     * @return
     */
    public static CommandResult execCommand(String[] commands, boolean isRoot) {
        CommandResult commandResult = new CommandResult();
        if (commands == null || commands.length == 0) return commandResult;
        Process process = null;
        DataOutputStream os = null;
        BufferedReader successResult = null;
        BufferedReader errorResult = null;
        StringBuilder successMsg = null;
        StringBuilder errorMsg = null;
        try {
            process = Runtime.getRuntime().exec(isRoot ? COMMAND_SU : COMMAND_SH);
            os = new DataOutputStream(process.getOutputStream());
            for (String command : commands) {
                if (command != null) {
                    os.write(command.getBytes());
                    os.writeBytes(COMMAND_LINE_END);
                    os.flush();
                }
            }
            os.writeBytes(COMMAND_EXIT);
            os.flush();
            commandResult.result = process.waitFor();
            //获取错误信息
            successMsg = new StringBuilder();
            errorMsg = new StringBuilder();
            successResult = new BufferedReader(new InputStreamReader(process.getInputStream()));
            errorResult = new BufferedReader(new InputStreamReader(process.getErrorStream()));
            String s;
            while ((s = successResult.readLine()) != null) successMsg.append(s);
            while ((s = errorResult.readLine()) != null) errorMsg.append(s);
            commandResult.successMsg = successMsg.toString();
            commandResult.errorMsg = errorMsg.toString();
            Log.i(TAG, commandResult.result + " | " + commandResult.successMsg
                    + " | " + commandResult.errorMsg);
        } catch (IOException e) {
            String errmsg = e.getMessage();
            if (errmsg != null) {
                Log.e(TAG, errmsg);
            } else {
                e.printStackTrace();
            }
        } catch (Exception e) {
            String errmsg = e.getMessage();
            if (errmsg != null) {
                Log.e(TAG, errmsg);
            } else {
                e.printStackTrace();
            }
        } finally {
            try {
                if (os != null) os.close();
                if (successResult != null) successResult.close();
                if (errorResult != null) errorResult.close();
            } catch (IOException e) {
                String errmsg = e.getMessage();
                if (errmsg != null) {
                    Log.e(TAG, errmsg);
                } else {
                    e.printStackTrace();
                }
            }
            if (process != null) process.destroy();
        }
        return commandResult;
    }

}
```




命令总结

```
# 显示已连接的设备
adb devices
# 显示结果如下所示：
# List of devices attached
# 6e070d91        device
# 其中6e070d91是设备的id，device是设备的状态。
# 设备状态有3种：offline表示设备离线，device表示设备连接正常，no device表示没有设备连接
# 如果有多台手机连接到电脑，则需要用 -s 指定adb调用的手机，如
# adb -s 6e070d91 install helloWorld.apk

# 获取手机序列号
adb get-serialno

# 获取手机连接的状态即offline、device和no device
adb get-state

# 在手机状态变成device后执行install helloWorld.apk
adb wait-for-device install helloWorld.apk

# 安装helloWorld.apk到手机上，如果手机里已经安装该应用，可加 -r 重新安装并保留应用的数据
adb install helloWorld.apk

# 卸载包名为com.example.test的应用，可加 -k 在卸载时保留配置和缓存文件
adb uninstall com.example.test

# 显示logcat，可使用grep过滤log，如adb logcat | grep debug
adb logcat

# 复制手机的/sdcard/foo.txt文件到本地并命名为foo.txt
adb pull /sdcard/foo.txt foo.txt

# 将foo.txt文件复制到手机的/sdcard/文件夹并命名为foo.txt
adb push foo.txt /sdcard/foo.txt

# 开启adb服务
adb start-server

# 结束adb服务
adb kill-server

# 进入shell模式，可在手机里执行shell命令
adb shell

# 不进入shell模式，直接执行shellCommand指令，如adb shell ls
adb shell shellCommand

# 启动包名为com.example.test的应用入口activity即com.example.test.Helloworld
adb shell am start -n com.example.test/.Helloworld

# 强制关闭包名为com.example.test的应用
adb shell am force-stop com.example.test

# 杀死包名为com.example.test的应用进程
adb shell am kill com.example.test

# 杀死所有的后台进程
adb shell am kill-all

# 列出设备上安装的所有应用的包名
# -f 可显示应用对应的文件
# -d 只显示被禁用的应用
# -e 只显示启用的应用
# -s 只显示系统应用
# -3 只显示第三方应用
# -i 显示应用安装的方式
adb shell pm list packages

# 列出系统所支持的功能，如蓝牙、相机、定位等
adb shell pm list features

# 显示com.example.test的apk路径
adb shell pm path com.example.test

# 删除和com.example.test相关的数据
adb shell pm clear com.example.test

# 启用应用com.example.test
adb shell pm enable com.example.test

# 禁用应用com.example.test
adb shell pm disable com.example.test

# 查看系统默认的安装方式，0 [auto]是系统自动决定安装位置，1 [internal]是安装在系统内部存储空间，2 [external]是安装在外置存储卡上
adb shell pm get-install-location

# 录制屏幕并保存为demo.mp4。该功能只能在Android 4.4(API level 19)或更高的版本运行。按Ctrl-C停止录制，否则在3分钟后自动停止录制。可通过--time-limit 30设置录制时间为30秒。
adb shell screenrecord /sdcard/demo.mp4  
```















