
# 查找一天前的文件

```
find ./ -mtime +1 -type f -name "*.log" | xargs ls  | wc -l

```

# 查找30分前的文件
```
find ./ -mmin +30 -type f -name "*.log" | xargs ls | wc -l

```

