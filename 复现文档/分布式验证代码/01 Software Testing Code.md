# 论文复现：Lunule: An Agile and Judicious Metadata Load Balancer for CephFS

## 1 购买和准备服务器环境

### 步骤 1：购买实验服务器，如阿里云、华为云、腾讯云提供的云服务器

本项目复现需要16个服务器节点，包含1个manager节点、5个MDS节点、9个OSD节点、1个Client节点

### 步骤 2：进行服务器集群各节点基本设置

+ 所有节点都要执行
+ 此处以mgr节点为例

```
[root@mgr ~]# cat >> /etc/hosts << EOF

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
EOF

[root@mgr ~]# systemctl stop firewalld
[root@mgr ~]# systemctl disable firewalld
[root@mgr ~]# setenforce 0
[root@mgr ~]# yum install ntp -y
[root@mgr ~]# systemctl enable ntpd
[root@mgr ~]# systemctl start ntpd;
```

## 2 安装 CephFS

您可以参考这个文档了解安装的具体过程：

[CephFS的安装](./02%20Install%20CephFS%20CN.md)

### 步骤 1：安装必要的软件包

+ 所有节点都要执行
+ 此处以 mgr 节点为例

```
[root@mgr ~]# yum install epel-release -y
[root@mgr ~]# cat > /etc/yum.repos.d/ceph.repo <<EOF

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
EOF

[root@mgr ~]# 
```

### 步骤 2：设置服务器之间的免密登录

+ mgr节点执行

```
[root@mgr ~]# ssh-keygen -t rsa
[root@mgr ~]# sudo chmod 700 .ssh
[root@mgr .ssh]# cd .ssh/
[root@mgr .ssh]# cat id_rsa.pub >> authorized_keys
[root@mgr .ssh]# rsync -avP ./* MDS1:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* MDS2:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* MDS3:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* MDS4:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* MDS5:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD1:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD2:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD3:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD4:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD5:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD6:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD7:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD8:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD9:/root/.ssh/
[root@mgr .ssh]# rsync -avP ./* OSD10:/root/.ssh/
```

验证配置情况

+ 所有节点都可以执行
+ 此处以mgr节点为例

```
[root@mgr ~]# cat /root/.ssh/authorized_keys
[root@mgr ~]# ssh root@MDS1
[root@mgr ~]# ssh root@MDS2
[root@mgr ~]# ssh root@MDS3
[root@mgr ~]# ssh root@MDS4
[root@mgr ~]# ssh root@MDS5
[root@mgr ~]# ssh root@OSD1
[root@mgr ~]# ssh root@OSD2
[root@mgr ~]# ssh root@OSD3
[root@mgr ~]# ssh root@OSD4
[root@mgr ~]# ssh root@OSD5
[root@mgr ~]# ssh root@OSD6
[root@mgr ~]# ssh root@OSD7
[root@mgr ~]# ssh root@OSD8
[root@mgr ~]# ssh root@OSD9
[root@mgr ~]# ssh root@OSD10
[root@mgr ~]# ssh root@mgr
```

### 步骤 3：安装Ceph并配置和创建节点  

+ 所有节点都要执行
+ 此处以mgr节点为例

```
[root@mgr ~]# yum install ceph-deploy -y
[root@mgr ~]# mkdir /etc/ceph
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# yum install -y python-setuptools
[root@mgr ceph]# ceph-deploy new mgr
[root@mgr ceph]# yum install ceph ceph-radosgw -y
[root@mgr ceph]# ceph -v
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# echo "public_network = 10.0.0.0/24" >> ceph.conf
```

+ OSD10节点是CephFS文件系统的客户端，需要继续执行以下指令

```
[root@OSD10 ~]# yum -y install ceph-common
```

### 步骤 4：设置Mon监视节点

+ mgr节点执行

```
[root@mgr ceph]# ceph-deploy mon create-initial
[root@mgr ceph]# ceph-deploy admin mgr MDS1 MDS2 MDS3 MDS4 MDS5 OSD1 OSD2 OSD3 OSD4 OSD5 OSD6 OSD7 OSD8 OSD9 OSD10
[root@mgr ceph]# ceph -s
[root@mgr ceph]# ceph-deploy mon add MDS1
[root@mgr ceph]# ceph-deploy mon add MDS2
[root@mgr ceph]# ceph-deploy mon add OSD1
[root@mgr ceph]# ceph-deploy mon add OSD2
[root@mgr ceph]# ceph -s
```

### 步骤 5：创建MGR管理节点

+ mgr节点执行

```
[root@mgr ceph]# ceph-deploy mgr create mgr
[root@mgr ceph]# ceph -s
```

### 步骤 6：创建OSD节点

+ mgr节点执行

```
[root@mgr ceph]# ceph-deploy disk zap OSD1 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD2 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD3 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD4 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD5 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD6 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD7 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD8 /dev/vdb
[root@mgr ceph]# ceph-deploy disk zap OSD9 /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd1 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd2 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd3 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd4 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd5 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd6 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd7 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd8 --data /dev/vdb
[root@mgr ceph]# ceph-deploy osd create osd9 --data /dev/vdb
[root@mgr ceph]# ceph -s
```

### 步骤 7：创建数据池

+ mgr节点执行

```
[root@mgr ceph]# ceph osd pool create test_pool 16
[root@mgr ceph]# ceph osd lspools
[root@mgr ceph]# ceph osd pool get test_pool pg_num
[root@mgr ceph]# ceph -s
```

### 步骤 8：删除数据池

+ mgr节点执行

```
[root@mgr ~]# cd /etc/ceph
[root@mgr ceph]# echo "mon_allow_pool_delete=true" >> ceph.conf
[root@mgr ceph]# ceph osd pool delete test_pool test_pool --yes-i-really-really-mean-it
[root@mgr ceph]# ceph -s
```

### 步骤 9：创建MDS节点

+ mgr节点执行

```
[root@mgr ceph]# ceph-deploy --overwrite-conf mds create MDS1 MDS2 MDS3 MDS4 MDS5
[root@mgr ceph]# ceph mds stat
[root@mgr ceph]# ceph -s
```

### 步骤 10：创建CephFS文件系统

+ mgr节点执行

```
[root@mgr ceph]# ceph osd pool  create ceph_data 9
[root@mgr ceph]# ceph osd pool  create ceph_metadata 5
[root@mgr ceph]# ceph fs new cephfs ceph_metadata ceph_data
new fs with metadata pool 4 and data pool 2
[root@mgr ceph]# ceph fs ls
```

### 步骤 11：配置客户端节点

+ 在 mgr 节点查看`key`并在 OSD10 节点保存至 `admin.key`

  + mgr节点执行
  ```
  [root@mgr ~]# cd /etc/ceph
  [root@mgr ceph]# cat ceph.client.admin.keyring
  ```

  + OSD10节点执行
  ```
  [root@OSD10 ~]# cd /etc/ceph
  [root@OSD10 ceph]# echo 'AQC0MdJlnkXkJxAAjrtKve4saaoimWHtRVq3mw==' >>admin.key
  [root@OSD10 ceph]# cat admin.key 
  AQAGn9JhQ3AUOxAAHz+xz6M8B86uoj+gV/ElIw==
  [root@OSD10 ceph]# mkdir /cephfs_data
  [root@OSD10 ceph]# mount.ceph mgr:6789:/ /cephfs_data/ -o name=admin,secretfile=/etc/ceph/admin.key
  ```

### 步骤 12：查看文件系统状态

```
[root@mgr ~]# df -h
```

## 3 编译安装Lunule

您可以参考这个文件了解安装的具体过程：

[Lunule的安装](./03%20lunule.md)

### 执行的指令

+ MDS1-5节点执行
+ 此处以MDS1节点为例

```
[root@MDS1 ~]# yum install curl -y
[root@MDS1 ~]# yum install wget -y
[root@MDS1 ~]# yum install zlib -y
[root@MDS1 ~]# yum install openssl -y
[root@MDS1 ~]# yum install expat -y
[root@MDS1 ~]# yum install libiconv -y
[root@MDS1 ~]# yum install curl-devel -y
[root@MDS1 ~]# yum install expat-devel -y
[root@MDS1 ~]# yum install gettext-devel -y
[root@MDS1 ~]# yum install openssl-devel -y
[root@MDS1 ~]# yum install zlib-devel -y
[root@MDS1 ~]# yum install git-core -y
[root@MDS1 ~]# git clone https://github.com/mdbal-lunule/lunule.git
[root@MDS1 ~]# yum install bzip2-devel.x86_64 -y
[root@MDS1 ~]# wget ftp://ftp.pbone.net/mirror/archive.fedoraproject.org/epel/7.2020-04-20/x86_64/Packages/p/python34-Cython-0.28.5-1.el7.x86_64.rpm
[root@MDS1 ~]# yum install -y python34-Cython-0.28.5-1.el7.x86_64.rpm
[root@MDS1 ~]# cd ./lunule
[root@MDS1 lunule]# ./install-deps.sh
[root@MDS1 lunule]# ./do_cmake.sh
[root@MDS1 build]# cd ./build
[root@MDS1 build]# make -j 16
[root@MDS1 build]# make install -j 16
[root@MDS1 build]# systemctl restart ceph-mds.target
```

### 遇到的问题

+ ERROR1：执行 `./install-deps.sh` 时，报错 `ERROR: No matching distribution found for setuptools<36,>=0.8`

  该问题是Python环境安装的问题。
  
  在查看本地所有的python运行环境，
  并设置多个国内可用的pypi镜像站以确保可以正确使用的情况下，
  该报错仍然无法解决。
  在查看 `./install-deps.sh` 脚本文件后，
  怀疑该问题与执行中使用venv创建了虚拟环境相关，
  运行时创建了虚拟环境并使用pip以安装相关依赖包，
  安装时无法访问pypi官方站从而导致的环境安装问题，
  不影响后续实验。

+ ERROR2：执行 `make` 时，报错 `c++: internal compiler error: killed (program cc1plus)`

  该问题是内存不足的问题。

  在实验中由于购买的服务器内存资源不够执行源代码的编译和安装，
  在执行到一些语句时会出现类似问题。
  一方面可以通过更换内存资源更多的服务器以解决，
  另一方面可以执行以下代码创建swap交换分区解决：

  ```
  [root@MDS1 ~]# sudo dd if=/dev/zero of=/swapfile bs=8M count=1024
  [root@MDS1 ~]# sudo mkswap /swapfile
  [root@MDS1 ~]# sudo swapon /swapfile
  ```

+ ERROR3：执行 `make` 时，报错

  ```
  make[2]: *** [src/test/CMakeFiles/ceph_test_librgw_file_marker.dir/librgw_file_marker.cc.o] Error 1
  make[1]: *** [src/test/CMakeFiles/ceph_test_librgw_file_marker.dir/all] Error 2
  make: *** [all] Error 2
  ```

  该问题为执行make和make install时遇到的两个TEST函数报错，
  不影响make、make install的执行。

  可以忽略该报错或尝试删除以下两个相关函数：

  `\src\test\librgw_file_marker.cc`

  ```
  TEST(LibRGW, CLEANUP) {
    int rc;

    if (do_marker1) {
      cleanup_queue.push_back(
        obj_rec{bucket_name, bucket_fh, fs->root_fh, get_rgwfh(fs->root_fh)});
    }

    for (auto& elt : cleanup_queue) {
      if (elt.fh) {
        rc = rgw_fh_rele(fs, elt.fh, 0 /* flags */);
        ASSERT_EQ(rc, 0);
      }
    }
    cleanup_queue.clear();
  }
  ```

  `\src\test\librgw_file_nfsns.cc`

  ```
  TEST(LibRGW, CLEANUP) {
    int rc;

    if (do_marker1) {
      cleanup_queue.push_back(
        obj_rec{bucket_name, bucket_fh, fs->root_fh, get_rgwfh(fs->root_fh)});
    }

    for (auto& elt : cleanup_queue) {
      if (elt.fh) {
        rc = rgw_fh_rele(fs, elt.fh, 0 /* flags */);
        ASSERT_EQ(rc, 0);
      }
    }
    cleanup_queue.clear();
  }
  ```

+ ERROR4：执行 `make install` 时，报错`undefined reference to 'pthread_create'`

  该问题是 `pthread` 库不是 Linux 系统默认的库引起的报错。

  因此需要在编译连接时使用静态库 `libpthread.a`，
  即在使用 `pthread_create()` 创建线程及调用 `pthread_atfork()` 函数建立 `fork` 处理程序时，
  在编译中需要添加 `-lpthread` 参数。
  这种情况类似于使用 `<math.h>`时，需在编译过程添加 `-m` 参数。

  例如：在加了头文件 `#include <pthread.h>` 之后执行 `pthread.c` 文件，
  需要使用如下命令：`gcc pthread.c -o thread -lpthread`

## 上传和解压ILSVRC2012数据集

### 步骤 1：从以下页面下载ILSVRC2012数据集

[IMAGENET Download](https://image-net.org/challenges/LSVRC/2012/2012-downloads.php)

+ 开发套件（Development Kit）

  + [Devlopment kit for Task1 & Task2](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_devkit_t12.tar.gz)

  + [Devlopment kit for Task3](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_devkit_t3.tar.gz)

+ 图像（Image）

  + [Task1和2的训练数据集(138GB)](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_img_train.tar)

  + [Task3的训练数据集(728MB)](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_img_train_t3.tar)

  + [验证数据集(6.3GB)](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_img_val.tar)

  + [测试数据集(13GB)](https://image-net.org/data/ILSVRC/2012/ILSVRC2012_img_test_v10102019.tar)

### 步骤 2：上传数据集到CephFS客户端OSD10节点

方法一：使用WinSCP等软件，将本地文件上传到服务器

方法二：使用wget等下载指令在服务器下载步骤1所述文件

### 步骤 3：在客户端解压文件

（1）解压测试集

```
[root@OSD10 Dataset]# tar -xvf ./ILSVRC2012_img_test_v10102019.tar -C ./ILSVRC2012_img_test
```

（2）解压验证集

```
[root@OSD10 Dataset]# tar -xvf ./ILSVRC2012_img_val.tar -C ./ILSVRC2012_img_val
```

（3）解压训练集

```
[root@OSD10 Dataset]# tar -xvf ./ILSVRC2012_img_train.tar -C ./ILSVRC2012_img_train
[root@OSD10 Dataset]# cat > "unzip.py" << EOF

import glob
import os

filelist = glob.glob("./ILSVRC2012_img_train/*.tar")

for f in filelist:
    os.system("mkdir ."+f.split(".")[1])
    os.system("tar -xvf "+f+" -C ."+f.split('.')[1])
    os.system("rm "+f)

EOF

[root@OSD10 Dataset]# python3 unzip.py
```

## 配置环境并运行img2rec.py进行测试

+ CephFS的客户端 OSD10 节点执行

### 具体流程

本部分需要根据上一步骤上传的数据集的具体位置，
对最终执行的代码进行一定的调整。

```
[root@OSD10 ~]# python3 -m pip install scikit-build  contextvars numpy mxnet opencv-python
[root@OSD10 ~]# git clone https://github.com/apache/mxnet.git
[root@OSD10 ~]# python3 ./mxnet/tools/im2rec.py --list --recursive {your_ceph_client_path}/record {your_ceph_client_path}/Dataset
```

e.g.

```
[root@OSD10 ~]# python3 ./mxnet/tools/im2rec.py --list --recursive /mnt/ILSVRC2012/record /mnt/ILSVRC2012/Dataset
```

### 遇到的问题

+ opencv-python安装出错  

  原因：最新opencv版本不支持当前安装使用的Python（如opencv 4.6不支持Python 3.6）

  解决方法：安装老版本的opencv-python
    ```
    [root@OSD10 ~]# python3 -m pip install opencv-python==4.3.0.38
    ```

+ 执行时numpy报错，不支持numpy.bool

  原因：numpy更新版本后，删除了numpy.bool属性

  解决方法：使用不高于1.20版本的numpy

    ```
    [root@OSD10 ~]# python3 -m pip install numpy==1.19.0
    ```

+ ImportError: libGL.so.1: cannot open shared object file: No such file or directory

  原因：缺少opencv-python依赖项libGL.so.1

  解决方法：
    + 方法1：`yum install libGL.so.1`
    + 方法2：安装无依赖opencv即opencv-headless
      ```
      [root@OSD10 ~]# python3 -m pip uninstall opencv-python
      [root@OSD10 ~]# python3 -m pip install opencv-python-headless
      ```
