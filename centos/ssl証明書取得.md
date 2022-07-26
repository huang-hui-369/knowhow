# Certbot

[System Requirements](https://eff-certbot.readthedocs.io/en/stable/install.html#id4) 

Certbot currently requires Python 3.6+ running on a UNIX-like operating system. By default, it requires root access in order to write to `/etc/letsencrypt`, `/var/log/letsencrypt`, `/var/lib/letsencrypt`; to bind to port 80 (if you use the `standalone` plugin) and to read and modify webserver configurations (if you use the `apache` or `nginx` plugins).



# install Certbot from docker(最简单的方法)

1. docker run

   ```sh
   docker run -it --rm --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "/var/lib/letsencrypt:/var/lib/letsencrypt" -v "/home/futari/dist/certbot:/etc/letsencrypt/webroot" certbot/certbot certonly
   ```

2. 选择 webroot 方式

   ```shell
   How would you like to authenticate with the ACME CA?
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   1: Spin up a temporary webserver (standalone)
   2: Place files in webroot directory (webroot)
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
   ```

3. 输入domain

   ```shell
   Please enter the domain name(s) you would like on your certificate (comma and/or
   space separated) (Enter 'c' to cancel): futari-passport.srcmaker.net
   Renewing an existing certificate for futari-passport.srcmaker.net
   Input the webroot for futari-passport.srcmaker.net: (Enter 'c' to cancel): /home/futari/dist/certbot/
   ```

   

4. 输入webroot目录

   注意此处将docker的webroot目录/etc/letsencrypt/webroot映射到host的/home/futari/dist/certbot

   host的web目录必须是从外部80端口可以访问的目录，因为要通过此目录来写token来验证domain的所属

   ```
   Please enter the domain name(s) you would like on your certificate (comma and/or
   space separated) (Enter 'c' to cancel): futari-passport.srcmaker.net
   Renewing an existing certificate for futari-passport.srcmaker.net
   Input the webroot for futari-passport.srcmaker.net: (Enter 'c' to cancel): /etc/letsencrypt/webroot
   ```

5. 更新成功

   ```sh
   Successfully received certificate.
   Certificate is saved at: /etc/letsencrypt/live/futari-passport.srcmaker.net/fullchain.pem
   Key is saved at:         /etc/letsencrypt/live/futari-passport.srcmaker.net/privkey.pem
   This certificate expires on 2022-05-10.
   These files will be updated when the certificate renews.
   
   NEXT STEPS:
   - The certificate will need to be renewed before it expires. Certbot can automatically renew the certificate in the background, but you may need to take steps to enable that functionality. See https://certbot.org/renewal-setup for instructions.
   
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   If you like Certbot, please consider supporting our work by:
    * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    * Donating to EFF:                    https://eff.org/donate-le
   - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
   ```

6. 确认结果

   已经更新到了/etc/letsencrypt/live/目录下

   ```sh
   ll /etc/letsencrypt/live/futari-passport.srcmaker.net/
   total 4
   lrwxrwxrwx 1 root   root    52 Feb  9 15:01 cert.pem -> ../../archive/futari-passport.srcmaker.net/cert2.pem
   lrwxrwxrwx 1 root   root    53 Feb  9 15:01 chain.pem -> ../../archive/futari-passport.srcmaker.net/chain2.pem
   lrwxrwxrwx 1 root   root    57 Feb  9 15:01 fullchain.pem -> ../../archive/futari-passport.srcmaker.net/fullchain2.pem
   lrwxrwxrwx 1 root   root    55 Feb  9 15:01 privkey.pem -> ../../archive/futari-passport.srcmaker.net/privkey2.pem
   -rw-r--r-- 1 futari futari 692 Aug 11 14:04 README
   ```

7. 创建自动更新sh update_sslcert.sh

   ```sh
   #!/bin/sh
   
   MAILTO=huanghui@grandunit.net
   
   echo "start update futari-passport.srcmaker.net ssl cert file"
   
   # 1. update ssl cert file
   echo "1. update ssl cert file"
   docker run -it --rm --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "/var/lib/letsencrypt:/var/lib/letsencrypt" -v "/home/futari/dist/certbot:/etc/letsencrypt/webroot" certbot/certbot renew
   if [ $? != 0 ]; then
     echo "update ssl cert file fialed"
     echo "update 1. ssl cert file fialed" | mail -s 'update ssl cert for futari staging' $MAILTO
     exit
   fi
   sleep 10s
   
   # 2. check cert file date
   echo "2. check cert file date"
   TODAY_YMD=$(date +%Y-%m-%d)
   CERT_FILE_YMD=`stat -c %y /etc/letsencrypt/live/futari-passport.srcmaker.net/fullchain.pem | cut -d' ' -f1`
   echo "fullchain.pem update date"  $CERT_FILE_YMD
   if [ $TODAY_YMD != $CERT_FILE_YMD ]; then
     echo "2.check cert file date"
     echo "2.check cert file date failed" | mail -s 'update ssl cert for futari staging' $MAILTO
     exit
   fi
   
   # 3. cp latest cert file to nginx
   cp /etc/letsencrypt/live/futari-passport.srcmaker.net/* /home/futari/nginx/conf/ssl_keys/current
   CMD_STATUS=$?
   if [ "$CMD_STATUS" != 0 ]; then
       echo "3. copy cert file failed"
       echo "3. copy cert file failed" | mail -s 'update ssl cert for futari staging' $MAILTO
       exit
   fi
   
   # 4. restart futari nginx
   docker restart futari-nginx 
   CMD_STATUS=$? 
   if [ "$CMD_STATUS" != 0 ]; then
       echo "4. restart futari-nginx failed"
       echo "4. restart futari-nginx failed" | mail -s 'update ssl cert for futari staging' $MAILTO
       exit
   fi
   
   echo "Update SSL Certfile successed"
   echo "Update SSL Certfile succed" | mail -s 'update ssl cert for futari staging' $MAILTO
   
   ```

   実行権限を付与します。

   ```sh
   chmod 755 /home/user1/update_sslcert.sh
   ```

   

8. 设置到cron定期任务中

   ```sh
   crontab -e
   
   # 下記一行を追加する
   #  3か月ごと、3日の朝7時半に実行しようとした場合、以下のようになります。
   30 07 03 */3 * /home/user1/update_sslcert.sh > /var/log/certbot_log 2>&1
   ```

   

# install Certbot

```sh
sudo yum install epel-release
sudo yum install certbot 
# aapcheの場合、install　python-certbot-apache、nginxの場合、インストールしなくてもいいです。
sudo yum install python-certbot-apache
```

# ssl証明書の取得

```sh
#証明書インストールのテスト(example.com部分は適宜変更する)
certbot certonly --webroot -w /home/nginx/publichtml -d example.com --email demo@gmail.com --dry-run

#問題無ければ証明書インストール
# http://example.com/.well-known/acme-challenge/(取得したトークン)
certbot certonly --webroot -w /home/nginx/publichtml -d example.com --email demo@gmail.com
```

1. 引数
   
- -webroot: Apache/Nginxを起動したままの認証とする。
- -w: 認証時の Documentroot を指定。
- -d: ドメインを指定。
- –preferred-challenges: 認証方式を指定する。ここではHTTP認証とする。
- –agree-tos: 利用規約の同意し、確認画面を表示しない。
- -m: Let’s Encryptに登録するメールアドレス。登録したくなければ、代わりに –register-unsafely-without-email を指定すればよい。

2. 上記コマンドを実行したら、/home/nginx/publichtmlフォルダに.well-known/acme-challenge/tokenファイルを自動的に生成した。

3. /etc/letsencrypt/live/example.comのフォルダに下記ファイルをダウンロードした。

- 証明書: cert.pem
- 秘密鍵: privkey.pem
- 中間証明書: chain.pem
- 証明書＋中間証明書: fullchain.pem

   **SSL 需要的是privkey.pem，fullchain.pem**

4. 証明書の内容は、opensslコマンドで確認できます。
```
openssl x509 -text -noout -in /etc/letsencrypt/live/example.com/cert.pem

notBefore=Aug 11 04:04:31 2021 GMT
notAfter=Nov  9 04:04:29 2021 GMT
```
Not Before（発行日）、Not After（有効期限）

# ssl証明書の更新

1. 証明書更新のコマンド
    ```sh
    sudo /usr/bin/certbot renew --post-hook "systemctl restart nginx"
    ```
    オプションについて補足します。
    - –post-hook: 証明書の更新後に実行するコマンドを指定する。更新が必要なときのみ（＝デフォルトでは有効期限まで30日未満のときのみ）実行される。一般的には、更新後の証明書を反映するための、Apache/Nginxを再起動するコマンドを指定する。
    - –pre-hook: 証明書の更新前に実行するコマンドを指定する。更新が必要なときのみ（＝デフォルトでは有効期限まで30日未満のときのみ）実行される。
    - –dry-run: テスト実行する（いわゆるドライラン）。実際には証明書は更新しない。- - –pre-hook, –post-hookで指定したコマンドは実行される。このオプションはcertonlyサブコマンドでも使用できる。
    - –force-renewal: 有効期限までの日数に関わらず、強制的に証明書を更新する。ただし、一定期間内に更新できる回数には制限がある。
    - 
2. Config
   これは、certonlyサブコマンドで証明書を取得したときに、ドメインごとのConfigファイル /etc/letsencrypt/renewal/<ドメイン>.conf に、取得実行時のオプションが保存されており、renewサブコマンドによる更新実行時は、そのConfigを参照するからです。
    ```
    # view /etc/letsencrypt/renewal/example.com.conf
    
    --
    # renew_before_expiry = 30 days
    version = 1.13.0
    archive_dir = /etc/letsencrypt/archive/example.com
    cert = /etc/letsencrypt/live/example.com/cert.pem
    privkey = /etc/letsencrypt/live/example.com/privkey.pem
    chain = /etc/letsencrypt/live/example.com/chain.pem
    fullchain = /etc/letsencrypt/live/example.com/fullchain.pem
    
    # Options used in the renewal process
    [renewalparams]
    account = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    pref_challs = http-01,
    authenticator = webroot
    webroot_path = /var/www/certbot/example.com,
    server = https://acme-v02.api.letsencrypt.org/directory
    [[webroot_map]]
    example.com = /var/www/certbot/example.com--

    ```
3. 自動更新スクリプト
    ```sh
    #!/bin/bash
    #
    CERTBOT_CMD=/usr/bin/certbot
    WEBSERVER_RESTART_CMD="systemctl restart ngixn"
    
    MAILTO=<メール通知先アドレス>
    
    echo "===== Update SSL Certfile ====="
    echo "`date` Update SSL Certfile start"
    
    # 証明書の更新
    ${CERTBOT_CMD} renew \
    --post-hook "${WEBSERVER_RESTART_CMD}"
    
    LE_STATUS=$?
    
    # 証明書の取得に失敗したときはメールで通知
    if [ "$LE_STATUS" != 0 ]; then
        echo "Update SSL Certfile failed" |\
        mail -s "Update SSL Certfile in `hostname`" ${MAILTO}
    fi
    
    echo "`date` Update SSL Certfile end"

    ```
    実行権限を付与します。

    ```sh
    chmod 755 /root/bin/update_sslcert.sh
    ```

4. cronに登録する

    ```sh
    crontab -e
    # 下記一行を追加する
    #  3か月ごと、3日の朝7時半に実行しようとした場合、以下のようになります。
    00 04 08 */3 * /home/user1/batch/update_sslcert.sh 1>> /var/log/update_sslcert.log 2>&1
    ```

    ```sh
    crontab -e

    # 下記一行を追加する
    #  3か月ごと、3日の朝7時半に実行しようとした場合、以下のようになります。
    30 07 03 */3 * /usr/bin/certbot renew --post-hook "systemctl restart nginx" > /var/log/certbot_log 2>&1

    ```



# 可能遇到的问题

1. 执行 

   ```
   certbot certonly --webroot -w /home/futari/dist/certbot/ -d futari-passport.srcmaker.net --email ryu@grandunit.com --dry-run
   ```

   遇到下记error

   ```
   Saving debug log to /var/log/letsencrypt/letsencrypt.log
   Plugins selected: Authenticator webroot, Installer None
   Missing command line flag or config entry for this setting:
   Please choose an account
   ```

2. 查看log 

   ```shell
   tail -300 /var/log/letsencrypt/letsencrypt.log
   ```

   发现_make_request time out

   ```
   2022-02-09 11:07:02,455:DEBUG:certbot._internal.main:certbot version: 1.11.0
   2022-02-09 11:07:02,455:DEBUG:certbot._internal.main:Location of certbot entry point: /usr/bin/certbot
   2022-02-09 11:07:02,455:DEBUG:certbot._internal.main:Arguments: ['--webroot', '-w', '/home/futari/dist/certbot/', '-d', 'futari-passport.srcmaker.net', '--email', 'ryu@grandunit.com', '--dry-run']
   2022-02-09 11:07:02,455:DEBUG:certbot._internal.main:Discovered plugins: PluginsRegistry(PluginEntryPoint#manual,PluginEntryPoint#null,PluginEntryPoint#standalone,PluginEntryPoint#webroot)
   2022-02-09 11:07:02,475:DEBUG:certbot._internal.log:Root logging level set at 20
   2022-02-09 11:07:02,475:INFO:certbot._internal.log:Saving debug log to /var/log/letsencrypt/letsencrypt.log
   2022-02-09 11:07:02,476:DEBUG:certbot._internal.plugins.selection:Requested authenticator webroot and installer None
   2022-02-09 11:07:02,477:DEBUG:certbot._internal.plugins.selection:Single candidate plugin: * webroot
   Description: Place files in webroot directory
   Interfaces: IAuthenticator, IPlugin
   Entry point: webroot = certbot._internal.plugins.webroot:Authenticator
   Initialized: <certbot._internal.plugins.webroot.Authenticator object at 0x330b990>
   Prep: True
   2022-02-09 11:07:02,477:DEBUG:certbot._internal.plugins.selection:Selected authenticator <certbot._internal.plugins.webroot.Authenticator object at 0x330b990> and installer None
   2022-02-09 11:07:02,477:INFO:certbot._internal.plugins.selection:Plugins selected: Authenticator webroot, Installer None
   2022-02-09 11:07:02,555:DEBUG:certbot._internal.main:Picked account: <Account(RegistrationResource(body=Registration(status=None, terms_of_service_agreed=None, agreement=None, only_return_existing=None, contact=(), key=None, external_account_binding=None), uri=u'https://acme-staging-v02.api.letsencrypt.org/acme/acct/22974688', new_authzr_uri=None, terms_of_service=None), eca850b0d6df039304898d9fda97e3d4, Meta(creation_host=u'STG-SRCMAKER.cs1644cloud.internal', register_to_eff=None, creation_dt=datetime.datetime(2021, 8, 11, 4, 37, 56, tzinfo=<UTC>)))>
   2022-02-09 11:07:02,563:DEBUG:acme.client:Sending GET request to https://acme-staging-v02.api.letsencrypt.org/directory.
   2022-02-09 11:07:02,576:INFO:urllib3.connectionpool:Starting new HTTPS connection (1): acme-staging-v02.api.letsencrypt.org
   2022-02-09 11:07:02,875:DEBUG:certbot._internal.log:Exiting abnormally:
   Traceback (most recent call last):
     File "/usr/bin/certbot", line 9, in <module>
       load_entry_point('certbot==1.11.0', 'console_scripts', 'certbot')()
     File "/usr/lib/python2.7/site-packages/certbot/main.py", line 15, in main
       return internal_main.main(cli_args)
     File "/usr/lib/python2.7/site-packages/certbot/_internal/main.py", line 1421, in main
       return config.func(config, plugins)
     File "/usr/lib/python2.7/site-packages/certbot/_internal/main.py", line 1277, in certonly
       le_client = _init_le_client(config, auth, installer)
     File "/usr/lib/python2.7/site-packages/certbot/_internal/main.py", line 659, in _init_le_client
       return client.Client(config, acc, authenticator, installer, acme=acme)
     File "/usr/lib/python2.7/site-packages/certbot/_internal/client.py", line 255, in __init__
       acme = acme_from_config_key(config, self.account.key, self.account.regr)
     File "/usr/lib/python2.7/site-packages/certbot/_internal/client.py", line 43, in acme_from_config_key
       return acme_client.BackwardsCompatibleClientV2(net, key, config.server)
     File "/usr/lib/python2.7/site-packages/acme/client.py", line 831, in __init__
       directory = messages.Directory.from_json(net.get(server).json())
     File "/usr/lib/python2.7/site-packages/acme/client.py", line 1168, in get
       self._send_request('GET', url, **kwargs), content_type=content_type)
     File "/usr/lib/python2.7/site-packages/acme/client.py", line 1118, in _send_request
       response = self.session.request(method, url, *args, **kwargs)
     File "/usr/lib/python2.7/site-packages/requests/sessions.py", line 486, in request
       resp = self.send(prep, **send_kwargs)
     File "/usr/lib/python2.7/site-packages/requests/sessions.py", line 598, in send
       r = adapter.send(request, **kwargs)
     File "/usr/lib/python2.7/site-packages/requests/adapters.py", line 370, in send
       timeout=timeout
     File "/usr/lib/python2.7/site-packages/urllib3/connectionpool.py", line 544, in urlopen
       body=body, headers=headers)
     File "/usr/lib/python2.7/site-packages/urllib3/connectionpool.py", line 344, in _make_request
       self._raise_timeout(err=e, url=url, timeout_value=conn.timeout)
     File "/usr/lib/python2.7/site-packages/urllib3/connectionpool.py", line 314, in _raise_timeout
       if 'timed out' in str(err) or 'did not complete (read)' in str(err):  # Python 2.6
   TypeError: __str__ returned non-string (type Error)
   2022-02-09 11:07:02,885:ERROR:certbot._internal.log:An unexpected error occurred:
   2022-02-09 11:07:02,885:ERROR:certbot._internal.log:TypeError: __str__ returned non-string (type Error)
   ```

   3. 更新urllib request lib version

      In Centos 7, some python libraries are not quite up-to-date yet

      [Bump requests requirement to >=2.10 ](https://github.com/certbot/certbot/pull/4248)

      ```
      pip install urllib3==1.26.7
      pip install requests==2.11.1
      ```

   4. 执行 certbot 然后遇到 certificate verify failed

      **原因**

      [2021年にLet’s Encryptのルート証明書が変更](https://ssl.sakura.ad.jp/column/letsencrypt-root-certificate/)

      [Let’s EncryptのルートCA証明書期限切れ](https://japan.zdnet.com/article/35177496/)

      ```shell
      certbot certonly --webroot -w /home/futari/dist/certbot/ -d futari-passport.srcmaker.net --email ryu@grandunit.com --dry-run
      Saving debug log to /var/log/letsencrypt/letsencrypt.log
      Plugins selected: Authenticator webroot, Installer None
      Starting new HTTPS connection (1): acme-staging-v02.api.letsencrypt.org
      An unexpected error occurred:
      SSLError: ("bad handshake: Error([('SSL routines', 'ssl3_get_server_certificate', 'certificate verify failed')],)",)
      ```

      **解决办法**

      因为yum没有对应解决办法，

      只能先install snapd 再 从snapd  install certbot

      ```
      yum install ca-certificates openssl
      ```

# install snapd  then install certbot

1. install snapd

   ```
   yum install snapd
   ```

2. snapd有效化

   ```
   systemctl enable --now snapd.socket
   ```

3. create snap link

   ```
   ln -s /var/lib/snapd/snap /snap
   ```

   

4. install snapd core

   ```
   snap install core
   ```

   遇到如下问题 只能升级 linux kernel

   ```
   error: system does not fully support snapd: cannot read the value of fs.may_detach_mounts kernel
          parameter: open /proc/sys/fs/may_detach_mounts: no such file or directory
   ```

   

5. remove old certbot

   ```
   yum remove certbot
   ```

6. install certbot from snapd

   ```
   snap install --classic certbot
   ```

7. create certbot link

   ```
   ln -s /snap/bin/certbot /usr/bin/certbot
   ```

   

# 确认/etc/letsencrypt 目录

1. 获取的ssl证明书在archive目录

```sh
ll /etc/letsencrypt
total 0
drwx------ 4 futari futari  84 Aug 11 13:56 accounts
drwx------ 3 futari futari  41 Aug 11 14:04 archive
drwxr-xr-x 2 futari futari 114 Aug 11 14:04 csr
drwx------ 2 futari futari 114 Aug 11 14:04 keys
drwx------ 3 futari futari  54 Aug 11 14:04 live
drwxr-xr-x 2 futari futari  46 Nov  9 17:17 renewal
drwxr-xr-x 5 futari futari  40 Aug 11 13:37 renewal-hooks
```

2. accounts是domain用户的目录

   acme-staging-v02.api.letsencrypt.org 是staging site

   acme-v02.api.letsencrypt.org是正式 site

   e2ca4524c390b6aae7004cea4fa90e93目录为用户的目录

   ```shell
   ll /etc/letsencrypt/accounts
   drwx------ 3 futari futari 22 Aug 11 13:37 acme-staging-v02.api.letsencrypt.org
   drwx------ 3 futari futari 22 Aug 11 13:56 acme-v02.api.letsencrypt.org
   
   ll /etc/letsencrypt/accounts/acme-v02.api.letsencrypt.org/directory/
   total 0
   drwx------ 2 futari futari 61 Aug 11 13:56 e2ca4524c390b6aae7004cea4fa90e93
   ```

   

3. 确认log

   ```
   tail -300 /var/log/letsencrypt/letsencrypt.log
   ```

   

# letsencrypt 証明書手動で取得流れ
https://sslnow.ml/
