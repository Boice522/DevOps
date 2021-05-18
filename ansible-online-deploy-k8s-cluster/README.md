# Ansible online deploy kuberbetes cluster
主要用于快速部署Kubernetes1.20高可用集群

## 说明
通过kubeadm，以在线的模式部署Kubernetes集群，目前已验证支持1.20.4-6版本。通过ansible-playbook一键部署。
etcd采用Stacked方式，高可用采用keepalived+haproxy方式，默认kube-proxy使用ipvs方式，加入了dashboard、ingress-nginx、metrics。

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
```
# 检测ansible到集群节点
ansible -i hosts all -m ping
```
使用默认ansible.cfg即可，这也我也附上了ansible.cfg的配置。建议ansible到集群机器做免密，也可以设置同样的密码通过-k来传递密码。
主要就是hosts文件，按照实际情况修改即可
```
[master]
# 如果部署单Master，只保留一个Master节点
# 默认Naster节点也部署Node组件
192.168.50.201 node_name=k8s-master01 cluster_role=master
192.168.50.202 node_name=k8s-master02 cluster_role=backup
192.168.50.203 node_name=k8s-master03 cluster_role=backup

# node节点可以随便加，cluster_role必填，值为node即可
[node]
192.168.50.204 node_name=k8s-node01 cluster_role=node
192.168.50.205 node_name=k8s-node02 cluster_role=node

# 如果是单master，这可以可以不写
[lb]
192.168.50.201 lb_role=MASTER keepalived_priority=100
192.168.50.202 lb_role=BACKUP keepalived_priority=90
192.168.50.203 lb_role=BACKUP keepalived_priority=80

[k8s:children]
master
node
```
其次还有变量，在group_vars/all.yml定义
```
# 工作目录
k8s_work_dir: '/opt/kubernetes'

# docker版本
docker_version: 'docker-ce-19.03.15-3.el7'

# kubernetes版本
k8s_version: '1.20.5'

# 网卡名称
nic_name: 'ens33'

# lb名称
lb_name: 'k8s-lb'
# keealived VIP
lb_vip: '192.168.50.200'
# lb端口
vip_port: '16443'
ha_stats_port: '8888'

#集群网络
service_cidr: '10.96.0.0/24'
pod_cidr: '172.16.0.0/16'
image_repository: 'registry.cn-hangzhou.aliyuncs.com/google_containers'
service_nodeport_range: '80-32767'

# calico版本,大版本即可
calico_version: 'v3.18'

# Dashboard，NodePort类型,端口号范围参考service_nodeport_range变量
dashboard_port: '30001'

# ingress-nginx，port范围参考service_nodeport_range变量
ingress_version: '0.46.0'
ingress_http_port: '80'
ingress_https_port: '30443'
```
## 部署Kubernetes集群
```
cd ansible-online-deploy-k8s-cluster
ansible-playbook -i hosts cluster-initialization.yml
# 如果只需执行其中部分可以使用tag来选择
ansible-playbook -i hosts cluster-initialization.yml --tags "ha"
```

