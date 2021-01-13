# CentOS内核升级

# 应用背景
CentOS7.*默认的系统内核最高为3.10.xxx，只能满足Docker对内核最低要求。另外3.10还有一些BUG，在kubernetes部署中显然版本过低，比如在部署[Cilium CNI](https://docs.cilium.io/en/v1.9/operations/system_requirements/#linux-kernel)时，官方建议kernel版本最低为4.9.17。所以我们在使用中需要升级内核版本。


# 测试环境
系统版本为CentOS最小话安装

| 系统版本 | 当前kernel内核版本 | 升级后kernel版本 |
| --- | --- | --- |
| CentOS 7.9.2009 (Core) | Linux 3.10.0-1160.el7.x86_64 | 5.4.88-1.el7.elrepo.x86_64 |

# 操作步骤
## 小版本升级


**1.查看系统版本和内核版本**
```
[root@k8s-master yum.repos.d]# uname -sr
Linux 3.10.0-1160.el7.x86_64
[root@k8s-master yum.repos.d]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

**2.查看可以升级的内核版本**
```
yum list kernel
```
**3.升级内核**
```
yum update kernel -y
```

**4.重启并查看**
```
[root@k8s-master ~]# reboot
[root@k8s-master ~]# uname -sr
Linux 3.10.0-1160.11.1.el7.x86_64
```
## 大版本升级
我们使用[ELRepo](http://elrepo.org/tiki/HomePage)社区企业Linux仓库来升级CentOS内核版本。


**1.导入公钥**
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```
**2.安装repo源**
```
yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```
**3.载入elrepo-kernel元数据**
```
yum --disablerepo=\* --enablerepo=elrepo-kernel repolist
```
**4.查看可安装的版本**
```
yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/366760/1610447584758-57fd9e7a-9bc5-45ae-9c78-4000f924f0bb.png#align=left&display=inline&height=431&margin=%5Bobject%20Object%5D&name=image.png&originHeight=862&originWidth=2868&size=224275&status=done&style=none&width=1434)
```
内核版本介绍：
lt:longterm的缩写：长期维护版；
ml:mainline的缩写：最新稳定版；
```
**5.安装内核（长期维护版本）**
```
yum --disablerepo=\* --enablerepo=elrepo-kernel install kernel-lt.x86_64 -y
```
**6.查看现有工具包**
```
rpm -qa | grep kernel
```
```
[root@k8s-master ~]# rpm -qa | grep kernel
kernel-3.10.0-1160.el7.x86_64
kernel-tools-libs-3.10.0-1160.el7.x86_64
kernel-lt-5.4.88-1.el7.elrepo.x86_64
kernel-tools-3.10.0-1160.el7.x86_64
kernel-3.10.0-1160.11.1.el7.x86_64
```
**7.卸载旧版本工具包**
```
yum remove kernel-tools-libs.x86_64 kernel-tools.x86_64  -y
```
**8.安装新版本工具包**
```
yum --disablerepo=\* --enablerepo=elrepo-kernel install kernel-lt-tools.x86_64 -y
```
```
[root@k8s-master ~]# rpm -qa | grep kernel
kernel-3.10.0-1160.el7.x86_64
kernel-lt-5.4.88-1.el7.elrepo.x86_64
kernel-3.10.0-1160.11.1.el7.x86_64
kernel-ml-tools-libs-5.10.6-1.el7.elrepo.x86_64
```
**9.查看内核启动顺序**
```
awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```
```
[root@k8s-master ~]# awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (5.4.88-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (3.10.0-1160.11.1.el7.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-a0274de3233e40f6a6b1bd50d702c580) 7 (Core)
```
说明：默认启动的顺序是从0开始，新内核是从头插入。

**10.查看当前启动顺序**
```
grub2-editenv list
```
```
[root@k8s-master ~]# grub2-editenv list
saved_entry=CentOS Linux (3.10.0-1160.11.1.el7.x86_64) 7 (Core)
```
**11.设置默认启动**
```
grub2-set-default 0
```
```
[root@k8s-master ~]# grub2-editenv list
saved_entry=0
```
**12.reboot重启验证**
```
[root@k8s-master ~]# uname -r
5.4.88-1.el7.elrepo.x86_64
```
**参考：**[https://www.cnblogs.com/ding2016/p/10429640.html](https://www.cnblogs.com/ding2016/p/10429640.html)
