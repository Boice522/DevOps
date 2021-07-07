# Ansible elasticsearch cluster offline install
主要用于快速部署elasticsearch高可用集群

## 说明
由于是内网环境，所以采用离线部署的方式，其中JDK和Elasticsearch安装包并有上传至仓库，需要自己下载并放到files目录下面
实验中采用jdk版本为11.0.11，es版本为7.6.2

## anshible
安装ansible工具
```
# 这里我是用了阿里云的epel源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
yum clean all && yum makecache

# 安装ansible
yum install ansible -y
```
## ansible配置

使用默认ansible.cfg即可，这也我也附上了ansible.cfg的配置。建议ansible到集群机器做免密，也可以设置同样的密码通过-k来传递密码。
主要就是hosts文件，按照实际情况修改即可
```
[master]
10.2.49.101 node_name=bigdata-master01
10.2.49.102 node_name=bigdata-master02
10.2.49.103 node_name=bigdata-master03
[data]
10.2.49.104 node_name=bigdata-data01
10.2.49.105 node_name=bigdata-data02
10.2.49.106 node_name=bigdata-data03
[escluster:children]
master
data
```
检测ansible到集群节点是否能连通
```
ansible -i hosts all -m ping
```
其次还有变量，在group_vars/all.yml定义
```
# 安装包存放目录
es_package_name: 'elasticsearch-7.6.2-linux-x86_64.tar.gz'

# es用户
es_user: 'elasticsearch'

# es版本
es_version: '7.6.2'

# es集群名称
es_cluster_name: 'bigdata-es-cluster'

# 数据和日志存放路径
es_data_dirs: '/data/elasticsearch'

# 是否允许跨域
http_cors: true

# 是否开启认证
xpack_security_enabled: true

# JVM 宿主机内存一半
es_master_heap_size: '2g'
es_data_heap_size: '2g'

# 是否升级jdk
update_jdk: true
```
## 部署Kubernetes集群
```
cd elastic_offline_install
ansible-playbook -i hosts install_es.yaml -k
# 如果只需执行其中部分可以使用tag来选择
ansible-playbook -i hosts cluster-initialization.yml --tags ""
```

