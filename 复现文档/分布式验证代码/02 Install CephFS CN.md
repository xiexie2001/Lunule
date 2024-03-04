# 安装和配置 CephFS 系统  

参考文献：[分布式存储之ceph软件安装及使用](https://blog.csdn.net/MssGuo/article/details/122280657)

1、创建服务器后，在每个服务器写入服务器集群各主机的IP

执行以下指令，并修改文件为对应内容：

```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# vim /etc/hosts
```

`/etc/hosts` 文件内容

```
::1     localhost       localhost.localdomain   localhost6      localhost6.localdomain6
127.0.0.1       localhost       localhost.localdomain   localhost4      localhost4.localdomain4


10.0.0.184      mgr     mgr
10.0.0.186      MDS1    MDS1
10.0.0.177      MDS2    MDS2
10.0.0.174      MDS3    MDS3
10.0.0.178      MDS4    MDS4
10.0.0.188      MDS5    MDS5
10.0.0.175      OSD1    OSD1
10.0.0.182      OSD2    OSD2
10.0.0.185      OSD3    OSD3
10.0.0.183      OSD4    OSD4
10.0.0.179      OSD5    OSD5
10.0.0.187      OSD6    OSD6
10.0.0.189      OSD7    OSD7
10.0.0.180      OSD8    OSD8
10.0.0.181      OSD9    OSD9
10.0.0.176      OSD10   OSD10
```

2、执行系统设置，包括关闭防火墙和对齐系统时间

```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# systemctl stop firewalld
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# systemctl disable firewalld
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# setenforce 0
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# yum install ntp -y
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# systemctl enable ntpd
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# systemctl start ntpd;
```

3、配置和安装必要的软件

（1）安装epel

```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# yum install epel-release -y
```

（2）配置ceph软件资源，并安装

```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# vim /etc/yum.repos.d/ceph.repo
```

`/etc/yum.repos.d/ceph.repo` 文件内容

```
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/x86_64/
enabled=1
gpgcheck=0
priority=1

[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/noarch/
enabled=1
gpgcheck=0
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-mimic/el7/SRPMS
enabled=1
gpgcheck=0
priority=1
```

3、配置服务器之间的免密码登录

参考文献：[配置ceph集群节点间的SSH免密登录快速方法](https://blog.csdn.net/SL_World/article/details/84190710)

（1）制作公钥并发送到所有服务器

```
[root@mgr ~]# ssh-keygen -t rsa
[root@mgr ~]# sudo chmod 700 .ssh
[root@mgr ~]# cd .ssh/
[root@mgr .ssh]# cat id_rsa.pub >> authorized_keys
rsync -avP ./* {mgr, MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}}:/root/.ssh/
```

（2）测试服务器公钥配置情况

```
[root@mgr ~]# cd /root/.ssh && cat  authorized_keys
[root@mgr .ssh]# ssh root@{mgr, MDS{1,2,3,4,5}, OSD{1,2,3,4,5,6,7,8,9,10}}
```

4、创建ceph系统

（1）安装ceph-deploy配置软件
```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9}} ~]# yum install ceph-deploy -y
```

（2）创建ceph目录

```
[root@mgr ~]# mkdir /etc/ceph
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# yum install -y python-setuptools
```

（3）在ceph系统中加入新节点（mgr、MDS、OSD等节点类似）
```
[root@mgr ~]# ceph-deploy new mgr
```

指令输出

```
[ceph_deploy.conf][DEBUG ] found configuration file at: /root/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy new mgr
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  func                          : <function new at 0x7f4ef3e6ded8>
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f4ef37f0c20>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  ssh_copykey                   : True
[ceph_deploy.cli][INFO  ]  mon                           : ['mgr']
[ceph_deploy.cli][INFO  ]  public_network                : None
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  cluster_network               : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  fsid                          : None
[ceph_deploy.new][DEBUG ] Creating new cluster named ceph
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[mgr][DEBUG ] connected to host: mgr 
[mgr][DEBUG ] detect platform information from remote host
[mgr][DEBUG ] detect machine type
[mgr][DEBUG ] find the location of an executable
[mgr][INFO  ] Running command: /usr/sbin/ip link show
[mgr][INFO  ] Running command: /usr/sbin/ip addr show
[mgr][DEBUG ] IP addresses found: [u'10.0.0.184']
[ceph_deploy.new][DEBUG ] Resolving host mgr
[ceph_deploy.new][DEBUG ] Monitor mgr at 10.0.0.184
[ceph_deploy.new][DEBUG ] Monitor initial members are ['mgr']
[ceph_deploy.new][DEBUG ] Monitor addrs are ['10.0.0.184']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
```

（4）查看 `/etc/ceph` 目录下文件

```
[root@mgr ceph]# ll
```

指令输出

```
total 16
-rw-r--r-- 1 root root  191 Feb 17 23:47 ceph.conf
-rw-r--r-- 1 root root 2882 Feb 17 23:47 ceph-deploy-ceph.log
-rw------- 1 root root   73 Feb 17 23:47 ceph.mon.keyring
-rw-r--r-- 1 root root   92 Jan 31  2020 rbdmap
-rw------- 1 root root    0 Feb 17 14:10 tmp2v1CBL
-rw------- 1 root root    0 Feb 17 16:11 tmpbRslhZ
-rw------- 1 root root    0 Feb 17 14:16 tmpgdleNS
-rw------- 1 root root    0 Feb 17 14:14 tmpjnpZN4
-rw------- 1 root root    0 Feb 17 16:10 tmpzUmO7M
```

（5）安装Ceph

```
[root@{mgr,MDS{1,2,3,4,5},OSD{1,2,3,4,5,6,7,8,9,10}} ~]# yum install ceph ceph-radosgw -y
```

（6）使用 `ceph -v` 检查安装的ceph系统版本信息

指令输出

```
ceph version 13.2.10 (564bdc4ae87418a232fc901524470e1a0f76d641) mimic (stable)
```

（7）OSD10是ceph系统的客户端，安装客户端程序

```
yum -y install ceph-common
```

（8）修改ceph配置文件 `ceph.conf`

在文件末尾插入新行，内容为 `public_network = 10.0.0.0/24`

```
[root@mgr ~]# cd /etc/ceph/
[root@mgr ceph]# vim ceph.conf
```

`/etc/ceph/ceph.conf` 文件内容

```
[global]
fsid = 9186e112-9343-4fa6-8bca-36f87a9ded25
mon_initial_members = mgr
mon_host = 10.0.0.184
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

（9）设置ceph系统的监视器 mon

```
[root@mgr ceph]# ceph-deploy mon create-initial
```

（10）将管理员信息发送给所有服务器

```
[root@mgr ceph]# ceph-deploy admin mgr MDS1 MDS2 MDS3 MDS4 MDS5 OSD1 OSD2 OSD3 OSD4 OSD5 OSD6 OSD7 OSD8 OSD9 OSD10
```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_OK
 
  services:
    mon: 1 daemons, quorum mgr
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   0B used, 0B / 0B avail
    pgs:     
```

（12）设置多个mon节点（奇数个，便于投票）

```
[root@mgr ceph]# ceph-deploy mon add MDS1
[root@mgr ceph]# ceph-deploy mon add MDS2
[root@mgr ceph]# ceph-deploy mon add OSD1
[root@mgr ceph]# ceph-deploy mon add OSD2
```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_WARN
            clock skew detected on mon.MDS1
 
  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

（13）创建manager节点
```
[root@mgr ceph]# ceph-deploy mgr create mgr
```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_OK
 
  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: mgr(active)
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

（14）创建OSD节点

格式化节点硬盘区块
```
[root@mgr ceph]# ceph-deploy disk zap OSD{1,2,3,4,5,6,7,8,9} /dev/vdb
```

如果出现创建失败的情况，尝试使用指令 `fdisk /dev/vdb` 重新创建硬盘分区

（选择 `n` 创建新分区表 -> 接下来4个默认首选项 ->键入 `w` 保存新分区表）

例如出现以下报错信息时，可以尝试该方法解决：
```
OSD4 cannot work by this method, we cannot understand its reason. But the same procedures on other Server can work properly.
```

创建OSD区块

```
[root@mgr ceph]# ceph-deploy osd create osd1 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd2 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd3 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd4 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd5 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd6 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd7 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd8 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd9 --data /dev/vdb
```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_OK
 
  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: mgr(active)
    osd: 9 osds: 9 up, 9 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   9.1 GiB used, 351 GiB / 360 GiB avail
    pgs: 
```

（15）创建数据池

```
[root@mgr ceph]# ceph osd pool create test_pool 16
```

使用指令 `ceph osd lspools` 查看数据池信息

指令输出

```
1 test_pool
```

使用指令 `ceph osd pool get test_pool pg_num` 查看数据池test_pool的数据块数量

指令输出

```
pg_num: 16
```

使用指令 `ceph -s` 查看ceph系统状态

Output

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_WARN
            too few PGs per OSD (5 < min 30)
 
  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: mgr(active)
    osd: 9 osds: 9 up, 9 in
 
  data:
    pools:   1 pools, 16 pgs
    objects: 0  objects, 0 B
    usage:   9.1 GiB used, 351 GiB / 360 GiB avail
    pgs:     16 active+clean
```

（16）删除已有数据池

```
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# vim ceph.conf
```

在第5行添加 `mon_allow_pool_delete=true`

将ceph.conf发送到所有集群节点，并重启监视器服务

```
[root@mgr ceph]# ceph-deploy --overwrite-conf admin mgr MDS1 MDS2 MDS3 MDS4 OSD1 OSD2 OSD3 OSD4 OSD5 OSD6 OSD7 OSD8 OSD9 OSD10
[root@{'All mon Node'} ceph]# systemctl restart ceph-mon.target
```

删除数据池

```
[root@mgr ceph]# ceph osd pool delete test_pool test_pool --yes-i-really-really-mean-it
```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_OK

  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: mgr(active)
    osd: 9 osds: 9 up, 9 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   9.1 GiB used, 351 GiB / 360 GiB avail
    pgs:
```

（17）创建MDS节点

```
[root@mgr ceph]# ceph-deploy --overwrite-conf mds create MDS1 MDS2 MDS3 MDS4 MDS5
```

使用指令 `ceph mds stat` 查看MDS节点状态

指令输出

```, 5 up:standby```

使用指令 `ceph -s` 查看ceph系统状态

指令输出

```
  cluster:
    id:     9186e112-9343-4fa6-8bca-36f87a9ded25
    health: HEALTH_OK

  services:
    mon: 5 daemons, quorum OSD1,MDS2,OSD2,mgr,MDS1
    mgr: mgr(active)
    osd: 9 osds: 9 up, 9 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   9.1 GiB used, 351 GiB / 360 GiB avail
    pgs:
```

（18）创建CephFS文件系统

```
[root@mgr ceph]# ceph osd pool  create ceph_data 9
[root@mgr ceph]# ceph osd pool  create ceph_metadata 5
[root@mgr ceph]# ceph fs new cephfs ceph_metadata ceph_data
new fs with metadata pool 4 and data pool 2
```

使用指令 `ceph fs ls` 查看CephFS文件系统状态

指令输出

```
name: cephfs, metadata pool: ceph_metadata, data pools: [ceph_data ]
```

（19）配置客户端节点

查看 `ceph.client.admin.keyring` 文件，并保存 `key`

```
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# cat ceph.client.admin.keyring
```

`/etc/ceph/ceph.client.admin.keyring` 文件内容

```
[client.admin]
        key = AQC0MdJlnkXkJxAAjrtKve4saaoimWHtRVq3mw==
        caps mds = "allow *"
        caps mgr = "allow *"
        caps mon = "allow *"
        caps osd = "allow *"
```

将 mgr 节点的`/etc/ceph/ceph.client.admin.keyring`的`key` 保存到 OSD10 节点的 `/etc/ceph/admin.key`

```
[root@OSD10 ~]# cd /etc/ceph
[root@OSD10 ceph]# echo 'AQC0MdJlnkXkJxAAjrtKve4saaoimWHtRVq3mw==' >>admin.key
[root@OSD10 ceph]# cat admin.key 
AQAGn9JhQ3AUOxAAHz+xz6M8B86uoj+gV/ElIw==
[root@OSD10 ceph]# mkdir /cephfs_data
[root@OSD10 ceph]# mount.ceph  mgr:6789:/ /cephfs_data/ -o name=admin,secretfile=/etc/ceph/admin.key
```

使用指令 `df -h` 查看节点的文件系统（硬盘）状态

指令输出

```
Filesystem         Size  Used Avail Use% Mounted on
devtmpfs           1.8G     0  1.8G   0% /dev
tmpfs              1.8G     0  1.8G   0% /dev/shm
tmpfs              1.8G  544K  1.8G   1% /run
tmpfs              1.8G     0  1.8G   0% /sys/fs/cgroup
/dev/vda1          148G  2.5G  139G   2% /
tmpfs              365M     0  365M   0% /run/user/0
10.0.0.184:6789:/  111G     0  111G   0% /cephfs_data
```