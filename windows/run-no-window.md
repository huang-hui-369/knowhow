# start  /b

window 还存在，不响应 ctrl+c

```bat
start /b B.bat >NUL 2>&1
```



# create run-no-window.vbs

```vbs
CreateObject("Wscript.Shell").Run "your_batch_file.bat", 0, True
```



# 利用 cmd /c 命令 会创建新的process

在 console中输入以下命令，再回车

```bat
cmd /c java -jar ftp-download_schedual-1.jar >NUL 2>&1
```

然后通过tasklist 查找java

```bat
tasklist | findstr java
```

