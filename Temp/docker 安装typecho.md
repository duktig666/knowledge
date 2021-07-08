

```
docker pull 80x86/typecho
```



```
docker run -d --name=typecho-blog --restart always -e PHP_TZ=Asia/Shanghai -e PHP_MAX_EXECUTION_TIME=600 -p 7001:80 80x86/typecho:latest
```



报错docker-compose build nginx

nginx uses an image, skipping

