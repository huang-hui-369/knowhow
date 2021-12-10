# 创建链接

![img](D:\github\knowhow\windows\link.assets\718864-20170926172409948-999405770.png)

- 硬链接

  链接一个文件链接，不能是目录，必须在同一个分区内，不需要管理权限

  ```sh
  mklink /h <リンク> <ターゲット>
  mklink /h hard_link.txt hoge.txt
  ```

  

- Junction

  创建一个目录链接，可以不在同一个分区，不需要管理权限

  ```sh
  mklink /j <リンク> <ターゲット>
  # 创建一个link将c的tmp目录指向d的tmp目录，这样在c的tmp目录下创建的文件都会保存到d的tmp目录
  mklink /j c:\tmp d:\tmp
  ```

  

- 软连接シンボリックリンク

  创建一个文件，目录链接，可以不在同一个分区，需要管理权限，Junction的增强版，兼容性不好

  ```sh
  # 创建一个文件软连接
  mklink <リンク> <ターゲット>
  # 创建一个目录软连接
  mklink /d <リンク> <ターゲット>
  ```

- 删除链接

  ```
  rmdir <リンク>
  ```

  







