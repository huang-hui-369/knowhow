# 更新安全补丁,不升级内核版本
1. 修改yum的配置文件
   `vim /etc/yum.conf`
   在  [main]  的最后添加 
   ```
   [main]
    cachedir=/var/cache/yum/$basearch/$releasever
    keepcache=0
    debuglevel=2
    logfile=/var/log/yum.log
    exactarch=1
    obsoletes=1
    gpgcheck=1
    plugins=1
    installonly_limit=5
    exclude=kernel*   # ←追加。 不升级kernel
    exclude=centos-release* # ←追加 不升级centos。
   ```
2. 重启yum服务
   `systemctl restart yum-cron`

# 手动更新安全补丁
1. 安装yum-security插件
   `yum install yum-security`
   这个插件赋给yum命令 --security、--cve、-bz、--advisory等可选参数
   list-security、info-security两个命令
2. 检查安全更新
   `yum -–security check-update`   
3. 只安装安全更新
   `yum --security upgrade`

4. 检查特定软件有无安全更新
   yum list-security software_name

5. 列出更新的详细信息·
   `yum info-security software_name`

# 除过kernel的更新
`yum –exclude=kernel* update`

# 更新所有
`yum update -y`

# 设置自动更新
1. 安装yum-cron
   `yum install yum-cron -y`

2. 修改 yum-cron.conf 
   `vi /etc/yum/yum-cron.conf`

   ```
    update_cmd = security
    update_messages = yes
    download_updates = yes
    apply_updates = yes
   ```

3. 启动yum-cron
   `systemctl start yum-cron`

4. 设置开机启动
   `systemctl enable yum-cron`
