---
title: "11.Linux 系统软件源收集"
slug: 11-linux-repository-sources
published: 2026-05-20
description: "Linux 系统软件源收集，记录 CentOS 等系统常用镜像源与仓库配置示例。"
tags: ["Linux", "软件源", "CentOS"]
category: Linux
draft: false
---
# centos源

## 阿里云

```shell
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#released updates 
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
 
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

## 腾讯云

```
echo "Ping991027;" | docker login --username aliyun9714359486 --password-stdin registry.cn-shenzhen.aliyuncs.com
```

```
https://restapi.amap.com/v5/ip/location?key=ea473540ff0df4fc28f9bc4249f8f8a8&ip=163.125.179.172&type=4
```

```shell
[os]
name=Qcloud centos os - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/os/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7

[updates]
name=Qcloud centos updates - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/updates/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7

[centosplus]
name=Qcloud centosplus - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/centosplus/$basearch/
enabled=0
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7

[cr]
name=Qcloud centos cr - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/cr/$basearch/
enabled=0
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=Qcloud centos extras - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7

[fasttrack]
name=Qcloud centos fasttrack - $basearch
baseurl=http://mirrors.cloud.tencent.com/centos/$releasever/fasttrack/$basearch/
enabled=0
gpgcheck=1
gpgkey=http://mirrors.cloud.tencent.com/centos/RPM-GPG-KEY-CentOS-7
```

## 清华源

```shell
cd  /etc/yum.repos.d/
mkdir -p /etc/yum.repos.d/bak
cp /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
sed -e "s|^mirrorlist=|#mirrorlist=|g" \
    -e "s|^#baseurl=http://mirror.centos.org/centos/\$releasever|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/7.9.2009|g" \
    -e "s|^#baseurl=http://mirror.centos.org/\$contentdir/\$releasever|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos-vault/7.9.2009|g" \
    /etc/yum.repos.d/CentOS-*.repo
```

`https://mirrors.tuna.tsinghua.edu.cn/help/centos-vault/`

# openEuler源

## 清华源

```shell
sed -i 's@repo.openeuler.org@mirrors.tuna.tsinghua.edu.cn/openeuler@' /etc/yum.repos.d/openEuler.repo 
```

# openstack源

## train版本

```shell
[openstack-train]
name=openEuler-os
baseurl=https://mirrors.cloud.tencent.com/centos/7.9.2009/cloud/x86_64/openstack-train/
enabled=1
gpgcheck=0

[ceph-nautilus]
name=ceph-nautilus
baseurl=https://mirrors.cloud.tencent.com/centos/7.9.2009/storage/x86_64/ceph-nautilus/
enabled=1
gpgcheck=0


[nfs-ganesha-28]
name=nfs-ganesha-28
baseurl=https://mirrors.cloud.tencent.com/centos/7.9.2009/storage/x86_64/nfs-ganesha-28/
enabled=1
gpgcheck=0
```



```bash

```


