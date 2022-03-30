# 查找*.vbs的执行进程

*.vbs 是根据wscript来解释执行的

```bat
F:\huanghui\windows\batch>c.vbs

F:\huanghui\windows\batch>tasklist | findstr "wscript"
wscript.exe                  16016 Console                    1     25,052 K

F:\huanghui\windows\batch>taskkill /f -pid 16016
成功: PID 16016 のプロセスは強制終了されました。
```

# 查找java的执行进程

java的jar，class文件也是根据java.exe来解释执行的

```bat
tasklist | findstr "java"
```

# 根据pid杀死进程

```bat
taskkill /f -pid 16016
```

# 查找监听端口的进程

1. 查找所有的端口并显示进程id

   ```bat
   netstat -ano
   アクティブな接続
   
     プロトコル  ローカル アドレス      外部アドレス           状態            PID
     TCP         0.0.0.0:21             0.0.0.0:0              LISTENING       12636
     TCP         0.0.0.0:80             0.0.0.0:0              LISTENING       2916
     TCP         0.0.0.0:135            0.0.0.0:0              LISTENING       72
     TCP         0.0.0.0:445            0.0.0.0:0              LISTENING       4
     TCP         0.0.0.0:1433           0.0.0.0:0              LISTENING       4268
   ```

2. 查找所有的监听端口

   ```bat
   netstat -ano | findstr "LISTENING"
     TCP         0.0.0.0:21             0.0.0.0:0              LISTENING       12636
     TCP         0.0.0.0:80             0.0.0.0:0              LISTENING       2916
     TCP         0.0.0.0:135            0.0.0.0:0              LISTENING       72
     TCP         0.0.0.0:445            0.0.0.0:0              LISTENING       4
     TCP         0.0.0.0:1433           0.0.0.0:0              LISTENING       4268
   ```

3. 查找指定端口

   ```bat
   # 查找指定端口
   netstat -ano | findstr "8009"
     TCP         [::1]:8009             [::]:0                 LISTENING       11076
     TCP         [::1]:8009             [::1]:17946            ESTABLISHED     11076
     TCP         [::1]:8009             [::1]:17947            ESTABLISHED     11076
     TCP         [::1]:8009             [::1]:50640            ESTABLISHED     11076
     TCP         [::1]:8009             [::1]:50641            ESTABLISHED     11076
     TCP         [::1]:8009             [::1]:50664            ESTABLISHED     11076
     TCP         [::1]:17946            [::1]:8009             ESTABLISHED     4960
     TCP         [::1]:17947            [::1]:8009             ESTABLISHED     4960
     TCP         [::1]:50640            [::1]:8009             ESTABLISHED     4960
     TCP         [::1]:50641            [::1]:8009             ESTABLISHED     4960
     TCP         [::1]:50664            [::1]:8009             ESTABLISHED     4960
   # 查找进程名
   tasklist | findstr "11076"
   tomcat9.exe                  11076 Services                   0    464,436 K
   ```

   

