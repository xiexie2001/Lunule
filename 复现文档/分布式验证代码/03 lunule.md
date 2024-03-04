# 执行的指令

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
[root@MDS1 ~]# yum install wget -y
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

# 遇到的问题

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
