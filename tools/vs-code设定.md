# 默认安装目录

`c:\Users\user1\AppData\Local\Programs\Microsoft VS Code`

# extensionsが保存されている場所



### 默认安装目录

下载下来的插件默认安装在 如下目录【%USERPROFILE%\.vscode\extensions】

```
C:\Users\user1\.vscode\extensions
```

### 更改插件安装目录

1. 通过命令行

   ```bat
   code --extensions-dir <dir>
   ```

   

2. 通过设置link

   ```shell
   mklink /j C:\Users\user1\.vscode\extensions D:\app\vscode\extensions
   ```

   

# 工作区存储目录

默认在以下目录

`%APPDATA%\Code\User\workspaceStorage`

`C:\Users\user1\AppData\Roaming\Code\User\workspaceStorage`

# ユーザー設定ファイル

默认在以下目录

`%APPDATA%\Code\User\settings.json`

`C:\Users\user1\AppData\Roaming\Code\User\settings.json`

打开vscode，同时按下ctrl+，就会打开设定界面

![image-20211126181333565](D:\github\knowhow\tools\vs-code设定.assets\image-20211126181333565.png)

# ワークスペース設定ファイル

默认在以下目录

`<プロジェクトルート>/.vscode/settings.json`

