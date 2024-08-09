
## [Windows下go项目打包注意项，交叉编译](https://www.cnblogs.com/yylyhl/p/18292325 "发布于 2024-07-09 18:10")

go版本1.21.4，Windows11

 
打包成linux程序-cmd

1.设置Linux编译环境，必须**分别执行**以下命令
SET CGO_ENABLED=0
set GOARCH=amd64
set GOOS=linux
go build -o filename

若用‘&amp;&amp;’批量执行编译环境设置，则会提示不支持：“go: unsupported GOOS/GOARCH pair linux/amd64”。出现不支持提示后需重复步骤2.

![](https://img2024.cnblogs.com/blog/401721/202407/401721-20240709165901209-2065742614.png )

2.想一次性执行可换个命令即可：go env -w CGO_ENABLED=0 GOARCH=amd64 GOOS=linux && go build -o filename


打包成linux程序-PowerShell

1.设置Linux编译环境，必须分别执行以下命令

$env:CGO_ENABLED="0"
$env:GOARCH="amd64"
$env:GOOS="linux"
go build -o filename