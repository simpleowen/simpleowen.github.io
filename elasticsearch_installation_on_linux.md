# 在 Linux 下安装 Elasticsearch



## 环境

- Ubuntu 18.04


## 前提

- 官方推荐 Java 8


## 下载

>curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-linux-x86_64.tar.gz
>
>tar -xvf elasticsearch-7.0.0-linux-x86_64.tar.gz


## 启动

> cd elasticsearch-7.0.0/bin
>
> ./elasticsearch


## 配置服务 ip 和 端口

进入 elasticsearch 安装目录，打开 config/elasticsearch.yml 配置 net.host 和 http.port

> net.host: 0.0.0.0
>
> http.port: 9200


## 停止

> CTRL + C


## 坑

1. 用户 
   Linux 下不要使用 root 用户运行 Elasticsearch, 否则会报异常 `can not run elasticsearch as root`
2. 引导检查
   如果你是使用 .zip 或 .tar.gz 安装包，有些配置需要手工配置，否则将导致启动失败，如：
   > ERROR: [3] bootstrap checks failed
   > [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
   > [2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
   > [3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

   在 Elasticsearch 以前的版本中，其中的一些配置错误被记录为警告信息。可以理解的是，用户有时会忽略这些日志消息。为了确保这些设置受到应有的重视，Elasticsearch在启动之前会做一些检查 [Bootstrap Checks | Elasticsearch Reference [7.0] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html)。这些 `启动检查`  [Important Elasticsearch configuration | Elasticsearch Reference [7.0] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html) 会检测 Elasticsearch 和系统的各种设置，并比较与那些为值是否是 Elasticsearch 的操作安全值。如果 Elasticsearch 处于开发模式，任何启动检失败被记录为 Elasticsearch 警告日志。如果 Elasticsearch 是在生产模式下，任何启动检查失败会导致 Elasticsearch 拒绝启动。

   要解决以上问题，需进行如下配置：

   - 文件描述符配置
     - 临时生效
       - 使用 `root` 用户
       - 运行 `ulimit -n 65536`
     - 永久生效
       - 使用 `root` 用户
       - 进入 `/etc/security/limits.conf` 
       - 添加一行 `user - nofile 65536`
   - 虚拟内存配置
     - 临时生效
       - 使用 `root` 用户
       - 运行 `sysctl -w vm.max_map_count=262144`
     - 永久生效
       - 使用 `root` 用户
       - 进入 `/etc/sysctl.conf` 
       - 添加或更新一行 `vm.max_map_count=262144`
   - 自动发现配置
     - 单节点
     - 进入 elasticsearch 安装目录，打开 config/elasticsearch.yml
     - 添加或更新一行 `discovery.type: single-node` 

## 链接

  - [Install Elasticsearch from archive on Linux or MacOS | Elasticsearch Reference [7.0] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)
  - [Elasticsearch 参考指南（引导检查） - 风继续吹 - SegmentFault 思否](https://segmentfault.com/a/1190000016750051)