# 1. 刪除垃圾文件

- 删除 C:\Windows\Prefetch

- 删除 C:\Windows\SoftwareDistribution （自动更新 cache）

- c disk clean up

  ![image-20221108194828543](D:\github\knowhow\windows\clean-c-drive.assets\image-20221108194828543.png)

# 2. 创建link将C:\Users\huanghui\AppData\Local\tmp映射到d disk

```
mklink /j C:\Users\huanghui\AppData\Local\Temp d:\Users\huanghui\AppData\Local\Temp
```

