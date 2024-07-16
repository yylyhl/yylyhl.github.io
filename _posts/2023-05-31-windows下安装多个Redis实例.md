
## [windows下安装多个Redis实例](https://www.cnblogs.com/yylyhl/p/17446134.html "发布于 2023-05-31 14:56")

 

1.复制配置: redis.windows-service.conf 为 redis.windows-service-6380.conf

2.更改配置: 如端口/密码等

3.安装实例: cd C:\Program Files\Redis &amp;&amp; redis-server.exe --service-install redis.windows-service-6380.conf --service-name Redis6380 --port 6380

4.启动实例: cd C:\Program Files\Redis &amp;&amp; redis-server.exe --service-start --service-name Redis6380




