# 添加sudo权限

## 方法1

1. root用户登录

2. 编辑/etc/sudoers文件 ,追加用户

   ```
   visudo
   ```

   找到"root ALL=(ALL) ALL" 这一行，

   添加"user1 ALL=(ALL) ALL" (这里的user1是你的用户名)，然后保存退出。

   ```
   ## Allow root to run any commands anywhere
   root    ALL=(ALL)       ALL
   user1    ALL=(ALL)     NOPASSWD:　  ALL
   ```

   

## 方法2

1. root用户登录

2. 添加文件的写权限。

   ```shell
   chmod u+w /etc/sudoers
   ```

   

3.  编辑/etc/sudoers文件 ,追加用户

   ```
   vi /etc/sudoers
   ```

   找到"root ALL=(ALL) ALL" 这一行，

   添加"user1 ALL=(ALL) ALL" (这里的user1是你的用户名)，然后保存退出。

   ```
   ## Allow root to run any commands anywhere
   root    ALL=(ALL)       ALL
   user1    ALL=(ALL)     NOPASSWD:　  ALL
   ```

4. 去除文件的写权限

   ```
   chmod u-w /etc/sudoers
   ```

   