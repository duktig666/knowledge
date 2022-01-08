## 统计nginx日志里访问次数最多的前十个IP

```sh
awk ‘{print $1}’ /var/log/nginx/access.log | sort | uniq -c | sort -nr -k1 | head -n 10
```

- -k1 第一列排序
- -nr 值大小排序 相反
- unqi -c 重复出现

