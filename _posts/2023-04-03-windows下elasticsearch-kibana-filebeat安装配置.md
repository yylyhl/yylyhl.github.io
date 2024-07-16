## [windows下elk安装配置-elasticsearch/kibana/filebeat](https://www.cnblogs.com/yylyhl/p/17283794.html "发布于 2023-04-03 19:53")

以8.6.2为例，下载地址
elasticsearch：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.6.2-windows-x86_64.zip
kibana：https://artifacts.elastic.co/downloads/kibana/kibana-8.6.2-windows-x86_64.zip
filebeat：https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.2-windows-x86_64.zip
分别解压至
D:\Deploy\Elastic\search\8.6.2\
D:\Deploy\Elastic\kibana\8.6.2\
D:\Deploy\Elastic\filebeat\8.6.2\

 

**elasticsearch 安装配置**

1.新增环境变量 ElasticSearch 已由 JAVA_HOME 转用 ES_JAVA_HOME。
　　变量名：ES_JAVA_HOME
　　变量值：D:\Deploy\Elastic\search\8.6.2\jdk

2.修改配置 D:\Deploy\Elastic\search\8.6.2\bin\elasticsearch-env 8.6.2及以后版本不需要，之前版本未测试
　　

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
if [ ! -z "$JAVA_HOME" ]; then
JAVA="$JAVA_HOME/bin/java"
JAVA_TYPE="JAVA_HOME"
改为
if [ ! -z "$ES_JAVA_HOME" ]; then
JAVA="$ES_JAVA_HOME/bin/java"
JAVA_TYPE="ES_JAVA_HOME"
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

3.修改配置 D:\Deploy\Elastic\search\8.6.2\config\elasticsearch.yml
　　

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
#设置快照存储地址
path.repo: ["D:\\Deploy\\Elastic\\search\\8.6.2\\backup"]

#数据存放路径（可不设置，默认就是如下地址）
path.data: D:/Deploy/Elastic/search/8.6.2/datas
#日志存放路径
path.logs: D:/Deploy/Elastic/search/8.6.2/logs

#节点名称
node.name: node-1
#节点列表
discovery.seed_hosts: ["127.0.0.1"]
#初始化时master节点的选举列表
cluster.initial_master_nodes: ["node-1"]

#集群名称
cluster.name: es-main
#对外提供服务的端口
http.port: 9200
#内部服务端口
transport.port: 9300

#启动地址，如果不配置，只能本地访问
network.host: 127.0.0.1
#跨域支持
http.cors.enabled: true
#跨域访问允许的域名地址
http.cors.allow-origin: "*"
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

4.修改 JVM 内存（按需） D:\Deploy\Elastic\search\8.6.2\config\jvm.options
　　#需在将 ElasticSearch 安装为服务前设置，否则安装服务后再改，重启也不会生效。
　　#-Xms和-Xmx属性值需相同，否则在启动服务的时出错，导致启动 ElasticSearch 服务失败。
　　

```
#设置最小内存
-Xms2g
#设置最大内存
-Xmx2g
```


 

# 5.安装 ElasticSearch 服务
　　sc stop elasticsearch-service-x64 &amp;&amp; sc delete elasticsearch-service-x64
　　cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-service.bat install
　　执行输出如下：
　　　　

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
C:\Users\Administrator>cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-service.bat install
Installing service : elasticsearch-service-x64
Using ES_JAVA_HOME : D:\Deploy\Elastic\search\8.6.2\jdk
[2023-04-01 10:48:38] [info] ( prunsrv.c:2002) [30124] Apache Commons Daemon procrun (1.3.1.0 64-bit) started.
[2023-04-01 10:48:38] [debug] ( prunsrv.c:772 ) [30124] Installing service...
[2023-04-01 10:48:38] [info] ( prunsrv.c:829 ) [30124] Installing service 'elasticsearch-service-x64' name 'Elasticsearch 8.6.2 (elasticsearch-service-x64)'.
[2023-04-01 10:48:38] [debug] ( prunsrv.c:857 ) [30124] Setting service description 'Elasticsearch 8.6.2 Windows Service - https://elastic.co'.
[2023-04-01 10:48:38] [debug] ( prunsrv.c:862 ) [30124] Setting service user 'LocalSystem'.
[2023-04-01 10:48:38] [info] ( prunsrv.c:879 ) [30124] Service 'elasticsearch-service-x64' installed.
[2023-04-01 10:48:38] [info] ( prunsrv.c:2086) [30124] Apache Commons Daemon procrun finished.
The service 'elasticsearch-service-x64' has been installed
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

6.启动 ElasticSearch 服务
　　sc start elasticsearch-service-x64

7.配置 SSL 证书（可选）
　　7.1.执行命令：cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-certutil ca
　　7.2.输入证书地址：D:\Deploy\Elastic\search\8.6.2\certs\elastic-stack-ca.p12
　　7.3.输入证书密码：password
　　#集群证书
　　7.4.输入证书命令：cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-certutil cert --ca D:\Deploy\Elastic\search\8.6.2\certs\elastic-stack-ca.p12
　　7.5.输入证书密码：password（步骤3设置的密码）
　　7.6.输入集群证书地址：D:\Deploy\Elastic\search\8.6.2\certs\elastic-stack-ca.p12
　　7.7.输入集群证书密码：password
　　　　7.7.1.1输入命令：cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
　　　　7.7.1.2输入密码：password（步骤7设置的密码）
　　　　7.7.2.1输入命令：cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
　　　　7.7.2.2输入密码：password（步骤7设置的密码）
　　7.8.将生成的证书拷贝到 D:\Deploy\Elastic\search\8.6.2\config\certs
　　7.9.在 D:\Deploy\Elastic\search\8.6.2\config\elasticsearch.yml 文件中增加配置：
　　　　

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
#开启xpack
xpack.security.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
#证书配置
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

8.重启服务
　　sc stop elasticsearch-service-x64 &amp;&amp; sc start elasticsearch-service-x64

9.设置账户密码
　　执行命令：cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-setup-passwords interactive
　　输入每个账户的密码和确认密码：password

　　执行输出如下：
　　　　

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
D:\Deploy\Elastic\search\8.6.2\bin>cd /D D:\Deploy\Elastic\search\8.6.2\bin &amp;&amp; elasticsearch-setup-passwords interactive
******************************************************************************
Note: The 'elasticsearch-setup-passwords' tool has been deprecated. This       command will be removed in a future release.
******************************************************************************

Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]:
Reenter password for [elastic]:
Enter password for [apm_system]:
Reenter password for [apm_system]:
Enter password for [kibana_system]:
Reenter password for [kibana_system]:
Passwords do not match.
Try again.
Enter password for [kibana_system]:
Reenter password for [kibana_system]:
Enter password for [logstash_system]:
Reenter password for [logstash_system]:
Enter password for [beats_system]:
Reenter password for [beats_system]:
Enter password for [remote_monitoring_user]:
Reenter password for [remote_monitoring_user]:
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

10.访问elasticsearch：http://127.0.0.1:9200，输入账户和密码(elastic/password(步骤9设置的密码))，输出如下

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
{
"name" : "node-1",
"cluster_name" : "es-main",
"cluster_uuid" : "DbPc6HE5Rs6s9isnyO9tJw",
"version" : {
"number" : "8.6.2",
"build_flavor" : "default",
"build_type" : "zip",
"build_hash" : "2d58d0f136141f03239816a4e360a8d17b6d8f29",
"build_date" : "2023-02-13T09:35:20.314882762Z",
"build_snapshot" : false,
"lucene_version" : "9.4.2",
"minimum_wire_compatibility_version" : "7.17.0",
"minimum_index_compatibility_version" : "7.0.0"
},
"tagline" : "You Know, for Search"
}
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

 

**kibana 安装配置**

1.更改配置 D:\Deploy\Elastic\kibana\8.6.2\config\kibana.yml 文件，在文件末尾增加如下配

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
#设置中文显示
i18n.locale: "zh-CN"

#设置访问用户
#elasticsearch.username: "elastic" #用此账号打不开：[FATAL][root] Error: [config validation of [elasticsearch].username]: value of "elastic" is forbidden. This is a superuser account that cannot write to system indices that Kibana needs to function. Use a service account token instead. Learn more: https://www.elastic.co/guide/en/elasticsearch/reference/8.0/service-accounts.html
elasticsearch.username: "kibana" #访问时若提示”Kibana 服务器尚未准备就绪。“等会再访问即可
#设置访问密码
elasticsearch.password: "password" #elasticsearch步骤9设置的密码

#ElasticSearch连接地址
elasticsearch.hosts: ["http://127.0.0.1:9200"]

#IP访问地址和端口号
server.host: "0.0.0.0"
server.port: 5601
#server.publicBaseUrl: "http://127.0.0.1:5601/"
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

 

2.安装 kibana 服务(用nssm.exe,路径D:\Deploy\Elastic\kibana\8.6.2\bin\kibana.bat)
　　sc stop elastic.kibana &amp;&amp; sc delete elastic.kibana
　　cd /D D:\Deploy\tools\nssm-2.2.4\win64 &amp;&amp; nssm install elastic.kibana

3.启动 kibana 服务
　　sc start elastic.kibana

4.访问kibana：http://127.0.0.1:5601，输入账户和密码(elastic/password(elasticsearch步骤9设置的密码))

 

 

**filebeat 安装配置**

 1.更改配置filebeat.yml

[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

```
filebeat.inputs:
- type: log
enabled: true
paths:
- D:\Deploy\logs\*
#- /var/log/*.log
#- c:\programdata\elasticsearch\logs\*

output.elasticsearch:
hosts: ["127.0.0.1:9200"]
username: "beats_system"
password: "password" #elasticsearch步骤9设置的密码
#/var/log/*.log：获取/var/log目录下所有以.log结尾的文件。
#/var/log/*/*.log：获取/var/log的子文件夹下所有的以.log结尾的文件。不会从/var/log文件夹本身抓取，不可能递归地抓取指定目录的子目录下文件。
```
[![复制代码](https://assets.cnblogs.com/images/copycode.gif )](javascript:void(0); "复制代码")

 

2.直接启动filebeat
　　cd /d D:\Deploy\Elastic\filebeat\8.6.2 &amp;&amp; filebeat -c filebeat.yml -e

3.以服务方式运行：filebeat
　　sc create elastic.filebeat binpath= "D:\\Deploy\\Elastic\\filebeat\\8.6.2\\filebeat.exe -c D:\\Deploy\\Elastic\\filebeat\\8.6.2\\filebeat.yml -e" start= auto
　　　　　　或sc create elastic.filebeat binpath= ""D:\Deploy\Elastic\filebeat\8.6.2\filebeat.exe" -c "D:\Deploy\Elastic\filebeat\8.6.2\filebeat.yml" -e" start= auto
　　sc start elastic.filebeat

 4.再补张图，filebeat启动后，访问kibana：Analytics-Discover页，日志已经出来了

![](https://img2023.cnblogs.com/blog/401721/202304/401721-20230403175947103-213718684.png )

 

 

 

参考：

https://www.cnblogs.com/qubernet/p/16849818.html
https://www.cnblogs.com/vipsoft/p/14808573.html



