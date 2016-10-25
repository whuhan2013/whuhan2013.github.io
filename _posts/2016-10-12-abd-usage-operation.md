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













