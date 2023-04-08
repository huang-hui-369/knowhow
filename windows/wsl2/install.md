# install wsl2

## Enable Virtualization
Task Manager を開いて Performance > CPU 画面から、 Virtualization: Enabled になっていることを確認します。

![](img\2021-05-12-16-04-19.png)

Virtualization: Enabled になっていない場合

コントロール パネル\プログラム\プログラムと機能を開く

![](img\2021-05-12-16-06-29.png)

## Enable Windows Feature
管理者モードで Windows PowerShell or PowerShell 6 を起動して二つのコマンドを実行します。再起動しないと機能が有効にならないため、コマンドを実行後に再起動してください。

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

或は以下の方法で設定する

Linux用Windows サブシステム

![](img\2021-05-12-16-09-50.png)

## Install WSL 2 kernel

このページ [Updating the WSL 2 Linux kernel | Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/wsl2-kernel) を開いて download the latest WSL2 Linux kernel を選択して Installer をダウンロードします。ダウンロードした Installer を実行してください。

## install ubuntu

Microsoft Store から Ubuntu-18.04 を Install します。今回は Ubuntu を利用しましたが、 Install Windows Subsystem for Linux (WSL) on Windows 10 | Microsoft Docs に記載されている通り、 Ubuntu ではなく好きな Linux でも問題ないと思います。 Microsoft Store 以外からの Install も可能と書いてありました。
1. Open the Microsoft Store and select your favorite Linux distribution.
    ![](img\2021-05-12-16-27-31.png)

2. From the distribution's page, select "Get".
   ![](img\2021-05-12-16-28-38.png)

3. The first time you launch a newly installed Linux distribution, a console window will open and you'll be asked to wait for a minute or two for files to de-compress and be stored on your PC. All future launches should take less than a second.

    第一次安装需要你创建一个新用户（dev）和输入新用户的密码

You will then need to create a user account and password for your new Linux distribution.

![](img\2021-05-12-16-29-28.png)

## Set your distribution version to WSL 1 or WSL 2

powershellを開く

```
PS C:\users\D2019-06> wsl -l -v
  NAME                   STATE           VERSION
* Ubuntu-18.04           Running         2
  docker-desktop-data    Running         2
  docker-desktop         Running         2
```

**デフォルトバージョン2に設定する**

```sh
wsl --set-default-version 2
# 上面的命令不行的話
 wsl --set-version Ubuntu-18.04 2
```
**デフォルトvm　Ubuntu-18.04に設定する**

```
wsl --set-default Ubuntu-18.04
```

# Ubuntu-18.04の操作

## 1. menue から開く

![](img\2021-05-12-16-37-06.png)

直接にUbuntu-18.04に入りました。

![](img\2021-05-12-16-38-49.png)

## 2. powershell　から

1. powershell wsl linux command
   
    ```
    PS D:\> wsl ls -ltr
    ls: 'System Volume Information': Permission denied
    total 0
    drwxrwxrwx 1 huang huang 4096 Sep 28  2020 '$RECYCLE.BIN'
    drwxrwxrwx 1 huang huang 4096 Sep 29  2020  プロジェクト
    drwxrwxrwx 1 huang huang 4096 Oct  1  2020  Users 
    ```

2. powershell で　wsl commandを実行する
   直接にUbuntu-18.04に入りました。
   
    ```
    PS D:\> wsl
    huang@GUT-D2019-06:/mnt/d$
    ```
   
3. exitを実行してUbuntu-18.04からpowershellへ戻る。
   
4. 修改ubuntu root用戶的密碼（刚装完ubuntu root用戶是没有密码的）

    ```sh
    $ sudo passwd root
    Enter new UNIX password: <任意のパスワード>
    Retype new UNIX password: <同じパスワードをもう一度>
    passwd: password updated successfully
    ```

    
## 3. 取消注册和重新安装分发版

尽管可以通过 Microsoft Store 安装 Linux 分发版，但无法通过 Store 将其卸载。 可以通过 WSL Config 来取消注册/卸载分发版。
取消注册后，还可以重新安装分发版。
警告： 注销后，与该分发关联的所有数据、设置和软件都将永久丢失。 从 Store 重新安装会安装分发版的干净副本。
```
wsl --unregister <DistributionName>
```
从 WSL 中取消注册分发版，以便能够重新安装或清理它。
例如：wsl --unregister Ubuntu-18.04 将从可用于 WSL 的分发版中删除 Ubuntu-18.04。 运行 wsl --list 时，不再会列出 Ubuntu-18.04。
若要重新安装，请在 Microsoft Store 中找到该分发版，然后选择“启动”。

## 4. ファイルの操作

1. 从 Linux 访问 Windows 驱动器文件系统 (DrvFS) 中的文件
在从 WSL 访问 Windows 文件（最有可能是通过 /mnt/c）时，会出现这些情况。

2. 使用 \\wsl$ 从 Windows 访问 Linux 文件
通过 \\wsl$ 访问 Linux 文件时将使用 WSL 分发版的默认用户。 因此，任何访问 Linux 文件的 Windows 应用都具有与默认用户相同的权限。
![](img\2021-05-12-16-56-23.png)



もっと知識は以下のサイトをご参照ください。
https://docs.microsoft.com/zh-cn/windows/wsl/wsl-config

# vs code からファイルの操作

## install vs code plugin remote wsl

![](img\2021-05-12-17-00-49.png)

## form vs code access ubuntu file

1. press f1 and type remote-wsl
    ![](img\2021-05-12-17-05-36.png)

2. select ubuntu
    ![](img\2021-05-12-17-07-15.png)

## in poweshell type code .

```
PS D:\> bash
huang@GUT-D2019-06:/mnt/d$ cd ~
huang@GUT-D2019-06:~$ code .

```

![](img\2021-05-12-17-12-23.png)


# install desk docker

1. download from https://docs.docker.com/get-docker/
   このページ [Docker Desktop for Mac and Windows | Docker](https://www.docker.com/products/docker-desktop) を開いて Installer をダウンロードします。ダウンロードした Installer を実行してください。 Install の途中で表示される Configuration は Enable WSL 2 Windows Features のチェックを外さないようにしてください。
    desk docker需要运行在linux环境中，所以会使用到windows的wsl技术，会要求在wsl中安装一个必须的linux内核。你也可以通过micro store安装一个ubuntu来替换这个linux内核。安装完成后实际上desk docker 就相当于windows的一个linux 子系统。

    ```
    PS D:\> wsl --list
    Linux 用 Windows サブシステム ディストリビューション:
    docker-desktop (既定)
    docker-desktop-data
    ```

2. setting
   
   1. checked use the wsl 2 based engine

   ![](img\2021-05-12-17-17-38.png)

   2. Resources 画面の Enable integration with my default WSL distro にチェックが付けてください。

    ![](img\2021-05-12-17-18-49.png)


# install Laravel docker by method1

1. ubuntuに入って下記のコマンドを実行する

```
curl -s https://laravel.build/example-app | bash
cd example-app
./vendor/bin/sail up
```
上記は一番簡単な方法ですが、けっこ時間がかかります。



# install Laravel docker by method2

## 下記コマンドを実行して。
docker-laravelのdocker-composeファイルの取得
```
git clone https://github.com/ucan-lab/docker-laravel.git
cd docker-laravel
docker-compose up -d
```

![](img\2021-05-12-17-48-26.png)

###  ERROR:Docker Got permission denied while trying to connect to the Docker daemon socket at unix://

```sh
# 正确的是将当前用户(dev)加入docker组
sudo gpasswd -a ${USER} docker
# 然后退出当前用户比如切换为root，再次切换为dev。然后执行docker-compose up -d就ok了。
su root
su user1
docker-compose up
```



以下の三つコンテナを作成しました。

```
├── app   ---  php-fpm php実行する環境
├── web   ---  nginx　
└── db    ---  mysql
```
## php 環境を確認する

1. docker-laravel/src/public/index.htmlを作成します。
```
Hello World
```
2. docker-laravel/src/public/index.phpを作成します。
```
<?php
phpinfo();
?>
```
3. http://localhost/index.html access
   ![](img\2021-05-12-17-53-44.png)

4. http://localhost/index.php access
    ![](img\2021-05-12-17-54-19.png)

php 環境はOKとなりました。

## php 環境の主な設定ファイル

1. Laravelプロジェクトのルートディレクトリ
    ```
    .
    ├── backend # Laravelプロジェクトのルートディレクトリ
    ├── infra
    │     └── docker
    │          ├── mysql
    │          │   ├── Dockerfile
    │          │   └── my.cnf
    │          ├── nginx
    │          │   ├── Dockerfile
    │          │   └── default.conf
    │          └── php
    │              ├── Dockerfile
    │              ├── php-fpm.d
    │              │   └── zzz-www.conf => unixドメインソケットの設定ファイル
    │              └── php.ini
    ├── Makefile
    └── docker-compose.yml
    ```
2. nginx.conf
    ```
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
    ```
    以上の設定により、.phpファイルはAppサーバーへ転送して実行する。

3. php-fpm.d/zzz-www.conf
    ```
    [www]
    listen = /var/run/php-fpm/php-fpm.sock
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0666
    access.log = /dev/stdout
    ```


## install Laravel

1. laravel-install
    ```sh
    # into php app servver containner
    docker compose exec app bash
    composer create-project laravel/laravel example-app
    composer create-project --prefer-dist "laravel/laravel=8.*" .
    root@257ea6dfb6a0:/work/backend# php artisan -V
    Laravel Framework 8.41.0
```
    
    ```
    docker compose exec app bash
    composer create-project laravel/laravel example-app
    cd example-app
    php artisan serve
```
    
2. create-project
    ```
    docker-compose exec docker-laravel_app_1 php artisan key:generate
	docker-compose exec docker-laravel_app_1 php artisan storage:link
	docker-compose exec docker-laravel_app_1 chmod -R 777 storage bootstrap/cache
    ```

3. access http://localhost/
   
   ![](img\2021-05-12-18-47-47.png)
   
4. install-recommend-packages (option)
    ```
    docker-compose exec app composer require doctrine/dbal "^2"
	docker-compose exec app composer require --dev ucan-lab/laravel-dacapo
	docker-compose exec app composer require --dev barryvdh/laravel-ide-helper
	docker-compose exec app composer require --dev beyondcode/laravel-dump-server
	docker-compose exec app composer require --dev barryvdh/laravel-debugbar
	docker-compose exec app composer require --dev roave/security-advisories:dev-master
	docker-compose exec app php artisan vendor:publish --provider="BeyondCode\DumpServer\DumpServerServiceProvider"
	docker-compose exec app php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
    ```
4. 

------------------------------------------------------

# install in windwos os host

# install php

## download from 
https://windows.php.net/downloads/releases/


# install composer

## download from
https://getcomposer.org/download/



## Command-line installation

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

# vs code extention

## PHP Debug

## PHP Intelephense
