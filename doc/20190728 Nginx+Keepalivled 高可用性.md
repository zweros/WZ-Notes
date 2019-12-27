---
title: Nginx 高可用 
date: 2019-07-28
---

## 更换 yum 源 ##

> 解决通过 yum 命令下载速度太慢的问题

- 进入 yum 源目录

  ```shell
  cd /etc/yum.repos.d/
  ```

- 先将 yum 源文件备份

  ```shell
  mv CentOS-Base.repo CentOS-Base.repo.backup
  ```

- 下载 阿里云的 yum 源文件

  ```shell
  # 163
  wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
  
  #阿里云 
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
  
  wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
  ```

- 执行 yum  makecache，生成缓存

  ```shell
  yum makecache
  ```

- 更新 yum 源

  ```shell
  yum -y update
  ```

<br/>

解决执行 `yum makecache` 出现 `[Errno 14] PYCURL ERROR 6 - "Couldn't resolve host 'mirrors.aliyun.com'" `的问题

**原因：**DNS 配置错误，重新配置网络适配器的 ifcfg-ech0 的 DNS 服务器地址即可

使用 vim 命令编辑 etc/sysconfig/network-scripts/ifcfg-eth0 文件

```shell
DNS1=233.5.5.5
DNS2=233.6.6.6
```

## Nginx+ keepalived 高可用性 ##

### 前期准备 ###

安装 Nginx 和 keepalived ，这里仅提供 keepalived 的安装步骤。

在两台虚拟机上安装  Nginx 和 keepalived ，**注意两台机器上的 Nginx 的配置文件相同** 

| IP              | 节点   |
| --------------- | ------ |
| 192.168.170.139 | master |
| 192.168.170.149 | backup |

### keepalived 的安装与使用 ###

- 下载 keepalived 依赖库

  ```shell
  yum install -y curl gcc openssl-devel libnl3-devel net-snmp-devel
  ```

- 下载 keepalived 

  这里提供 yun 方式安装，简单快速，比源码安装简单

  ```shell
  yum install -y keepalived
  ```

- keepalived 的使用

  ```shell
  service keepalived start
  service keepalived status
  service keepalived stop
  # 设置自启
  chkconfig keepalived on
  ```

### keepalived 的配置 ###

主要是对 `/etc/keepalived/keepalived.conf` 文件的修改配置

- master 节点

```shell
global_defs {
   notification_email {
     acassen@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_01   #相当于 mysql 的 server_id ,唯一标识
}

vrrp_instance VI_1 {  #实例名称，与 backup 节点保持一致
    state MASTER      #表示主节点
    interface eth1    #使用的网卡
    virtual_router_id 51   #与 backup 从节点保持一致，防止裂脑
    priority 150      #优先级，master 的优先级高于 backup ，一般以 50 作为步长
    advert_int 1    
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {  #虚拟 IP 
        192.168.170.3/24
    }
}

# 其他配置省略
```

- backup 节点

```shell
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_02
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.170.3/24   
    }
}

# ip addr | grep "192.168.3.2/24"  这个ip只有在master挂掉的时候才会有
```



## 参考 ##

[修改 yum 源](https://blog.csdn.net/inslow/article/details/54177191)

[Nginx 高可用性1](https://blog.csdn.net/bbwangj/article/details/80346428)

[Nginx 高可用性2](https://klionsec.github.io/2017/12/23/keepalived-nginx)

