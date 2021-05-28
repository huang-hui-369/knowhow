# 特定の文字が含まれている行数を出力

```
$ cat /var/log/nginx/access.log | grep {検索したい文字列} | wc -l
```

# 複数のアクセスログファイルを含めて行数を出力

```
# 数日前のアクセスログは圧縮されているため解凍する
$ gunzip /var/log/nginx/20190720.gz

# 行数の出力
$ find /var/log/nginx/ -type f -name "access.log-2019*" | xargs sudo grep jobTypes | wc -l
7388

# 解凍したファイルを圧縮し直しておく
$ gzip /var/log/nginx/20190720.gz
```

