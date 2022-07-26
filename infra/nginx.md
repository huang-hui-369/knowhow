

# add http header

1.  在 server設置

   如果在location中設置了  add_header，在 server中設置的add_header就不会起作用

```sh
server {

add_header X-Frame-Options 'sameorigin always';
listen       443 ssl;
server_name  futari-passport.srcmaker.net;
ssl_certificate  conf.d/ssl_keys/current/fullchain.pem;
ssl_certificate_key conf.d/ssl_keys/current/privkey.pem;
ssl_session_timeout 5m;
ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
#access_log  /var/log/nginx/host.access.log  main;

#location / {
#    root   /usr/share/nginx/html;
#    index  index.html index.htm;
#}
```

2. 在location中设置


```sh
# html vue
location / {
            add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
            add_header X-Frame-Options 'sameorigin always';
            add_header X-Content-Type-Options 'nosniff';

    root /dist/;
    index  index.html;
    expires modified +90d;
}
```
# 一个完整的设置

```sh
gzip on;
gzip_static on;
gzip_comp_level 2;
gzip_types text/plain text/html text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# sendfile off;

server {
    listen       80;
    server_name futari-passport.srcmaker.net;
    client_max_body_size 20m;
    # html vue

    location / {
                add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
        root /dist/certbot/;
        index  index.html;
        expires modified +90d;
    }
    # 设置通过80访问的强制转到https端口访问
    return 301 https://futari-passport.srcmaker.net:443$request_uri;
}

server {

    listen       443 ssl;
    server_name  futari-passport.srcmaker.net;
    # ssl设置
    # ssl fullchain
    ssl_certificate  conf.d/ssl_keys/current/fullchain.pem;
    # ssl privateKey
    ssl_certificate_key conf.d/ssl_keys/current/privkey.pem;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    # ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    # 可用ssl协议的设置，如果要禁用TLSv1.1只要去掉TLSv1.1就可以
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    #access_log  /var/log/nginx/host.access.log  main;

    # 设置根路径html vue
    location / {
                add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
                add_header X-Frame-Options 'sameorigin always';
                add_header X-Content-Type-Options 'nosniff';

        root /dist/;
        index  index.html;
        expires modified +90d;
    }

    # broswer histoy back
    location ~ ^/(index|K0001|search_list|search_list_info|user-entry|K0030|vistor_apply_notice|vistor-faq|store-entry|store-faq|app-download|store_apply|store_apply_notice|K0211|K0212|K0213|K0232|K0110|K0111|K0112|K0120|K0130|K0131|K0141|K0301|K0080|K0082|K0083|K0084|K0090|K0091|K0410|K0411|K0413|K0414|K0220|K0221|K0222|K0223|K0224|K0225|K0226|K0230|K0308|K0302|K0303|K0304|K0330|K0331|K0333|K0340|K0341|K0342|K0405|K0350|K0352|K0353|K0354|K0362|K0363|K0364|K0372|K0373|K0374|warning|404|K0380)$ {

		add_header Cache-Control 'no-cache, must-revalidate, proxy-revalidate, max-age=0';
         add_header X-Frame-Options 'sameorigin always';
         add_header X-Content-Type-Options 'nosniff';

        root /dist/;
           # try_files指令 返回第一个找到的文件或文件夹(结尾加斜线表示为文件夹)，如果所有的文件或文件夹都找不到，会进行一个内部重定向到最后一个参数。
           # 这里查找$uri/文件夹，如果不存在，转向/路径
            try_files $uri $uri/ /;
            index  index.html;
            expires modified +90d;
     }

    # pic,doc... static resource
    # 这里注意配置root /;    当你访问/doc/a.html时，回取读取/doc/a.html
    # 这里注意配置root /doc; 当你访问/doc/a.html时，回取读取/doc/doc/a.html
    location /doc/ {
            root /;
            autoindex on;
            autoindex_exact_size off;
            autoindex_localtime on;
    }

    # backup service forward
    # 设置访问/web/路径的请求转发到http://futari-spring:8088/
    location ^~/web/ {
            proxy_pass http://futari-spring:8088/;
        }
    location ^~/app/ {
        proxy_pass http://172.20.0.198:8088/app/;
    }

    error_page  404              /index.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```



# [虚拟目录alias和root目录](https://www.cnblogs.com/kevingrace/p/6187482.html)

## alias虚拟目录

location /huan/ {
   alias /home/www/huan/;
}

在上面alias虚拟目录配置下，访问http://www.wangshibo.com/huan/a.html实际指定的是/home/www/huan/a.html。
注意：alias指定的目录后面必须要加上"/"，即/home/www/huan/不能改成/home/www/huan



## root目录

location /huan/ {
    root /home/www/;
}

访问http://www.wangshibo.com/huan/a.html实际指定的是/home/www/huan/a.html。


