

## [nginx windows下重载配置失败](https://www.cnblogs.com/yylyhl/p/17434404.html "发布于 2023-05-26 12:30")

正常操作命令是：cd C:\Program Files\nginx &amp;&amp; nginx -s reload。

但是把nginx安装为windows服务运行后，再执行重载就不好使了。

会报错：nginx: [error] OpenEvent("Global\ngx_reload_4460") failed (5: Access is denied)，即便是用管理员运行cmd权限也不够。

![](https://img2023.cnblogs.com/blog/401721/202305/401721-20230526133351743-1432438629.png )

 

 

【解决办法】

1.下载[pstools](https://download.sysinternals.com/files/PSTools.zip "pstools")，解压后放入C:\Program Files目录下。

2.执行命令：C:\Program Files\PSTools\psexec.exe -s C:\Program Files\nginx\nginx.exe -p C:\Program Files\nginx -s reload

   此时会报错：PsExec could not start C:\Program on WIN-F0C8K :系统找不到指定的文件。

![](https://img2023.cnblogs.com/blog/401721/202305/401721-20230526133512359-2060350901.png )

 

因为路径有空格，需用引号包起来执行："C:\Program Files\PSTools\psexec.exe" -s "C:\Program Files\nginx\nginx.exe" -p "C:\Program Files\nginx" -s reload

3.执行结果：C:\Program Files\nginx\nginx.exe exited on WIN-F0C8K with error code 0.重载就完成了。

![](https://img2023.cnblogs.com/blog/401721/202305/401721-20230526133617938-285750391.png )

 

 

 

参考：

https://www.cnblogs.com/CoreXin/p/5743412.html

https://learn.microsoft.com/zh-cn/sysinternals/downloads/pstools



