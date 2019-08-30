---
layout: post
title:  fastdfs部署
categories: fastdfs
description: fastdfs在linux上的部署
keywords: fastdfs
---

  本章记录linux下对fastdfs的部署以及遇到的问题

## FASTDFS 介绍
   FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
   FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务
   更多详细的可以百度下
   
## 部署

   ### 准备
   
   搭建所需要的工具如下图
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/installation-package.jpg)
   
   FastDfs下载地址
    
   链接: https://pan.baidu.com/s/1TNU9nZSbupEkYGAjZf7Gig 提取码: w71k
   
   nginx 还可以到下面地址中完成下载。
   
   http://nginx.org/download/
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/nginx-download.jpg)
   
   完成上面所有的工具下载后，我们准备进行下一步。
   
   ### 安装
   
   1. libfastcommon 安装
   
   将 libfastcommon-master.zip  上传到服务器，执行解压命令
     
   > unzip libfastcommon-master.zip 
   
   解压后进入目录，找到图中shell脚本文件，我们需要执行的就是此文件
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/libfastcommon.jpg)
  
  然后执行下面命令，安装libfastcommon
   
  > ./make.sh
    ./make.sh install
    
  2. FastDfs安装
  
   将 fastdfs-5.11.zip  上传到服务器，执行解压命令
         
   > unzip fastdfs-5.11.zip 
     
   到解压目录下执行下面命令，安装fastdfs
        
   > ./make.sh
     ./make.sh install
     
   安装好之后，执行命令切换到该目录下
   
   >cd /etc/fdfs/
  
   会发现有目录下有这几个文件
   
   >-rw-r--r-- 1 root root  1461 Jun  8 21:56 client.conf.sample
    -rw-r--r-- 1 root root  7927 Jun  8 21:56 storage.conf.sample
    -rw-r--r-- 1 root root  7389 Jun  8 21:56 tracker.conf.sample  
    
   把这3个文件重新拷贝一份
   
   >cp client.conf.sample client.conf
    cp storage.conf.sample storage.conf
    cp tracker.conf.sample tracker.conf
    
  3. tracker 配置安装
  
  在/usr/fastdfs/ 下新建tracker目录
  
  >mkdir fastdfs_tracker
  
  配置tracker.conf
  >vim /etc/fdfs/tracker.conf
  
  打开后配置
  
    1.disabled=false #默认开启 
    2.port=22122 #默认端口号 
    3.base_path=/usr/muyou/dev/fastdfs/fastdfs_tracker #刚刚创建的目录 
    4.http.server_port=6666 #默认端口是8080
    
  保存修改，并执行启动命令  
  
  >service fdfs_trackerd start
  
  或
  
  >systemctl start fdfs_trackerd
  
  或
  
  >fdfs_trackerd /etc/fdfs/tracker.conf start
  
  成功后会看到
  
  [root@localhost fdfs]# service fdfs_trackerd start
  Starting fdfs_trackerd (via systemctl):                    [  确定  ]
  
  然后进入刚才创建饿fdfs_tracker目录，会发先下面多了两个目录data和logs
  
  为了避免每次开机后都执行上面命令启动服务，我们需要将其加入开机启动
  
  加上权限
  
  >chmod +x /etc/rc.d/rc.local
  
  编辑文件
  
  >vi /etc/rc.d/rc.local
  
  在末尾加上 service fdfs_trackerd start，如下图
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/rd.local.jpg)
  
  保存后执行命令查看端口
  
  >netstat -unltp|grep fdfs
  
  出现如下图，说明tracker启动成功
  
  ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/tracker-listen.jpg)
  
   3. storage 配置安装
   
   在/usr/fastdfs/ 下新建两个目录
   
   > mkdir fastdfs_storage
     mkdir fastdfs_storage_data
   
   分别用于存储storage日志和文件
   
   修改storage.conf 配置文件
   
   >vim /etc/fdfs/storage.conf
   
   正常情况单机修改如下几点就可以了
   
       1.disabled=false 
       2.group_name=group1 #组名，根据实际情况修改 
       3.port=23000  #设置storage的端口号，默认是23000，同一个组的storage端口号必须一致 
       4.base_path=/usr/fastdfs/fastdfs_storage  #设置storage数据文件和日志目录 
       5.store_path_count=1 #存储路径个数，需要和store_path个数匹配 
       6.base_path0=/usr/fastdfs/fastdfs_storage_data #实际文件存储路径 
       7.tracker_server=192.168.192.128:22122 #我CentOS7的ip地址 
       8.http.server_port=8888 #设置 http 端口号
   
   上面的ip地址如果是虚拟机，是虚拟机的ip地址，并且虚拟机网络配置需要设置为NAT模式，不然后面启动或配置nginx时候会遇到问题
   
   保存过后启动storage
   
   >service fdfs_storaged start
     
   或
     
   >systemctl start fdfs_storaged
     
   或
     
   >fdfs_storaged /etc/fdfs/storage.conf start
   
   同样在末尾加上 service fdfs_storaged start，如下图
     
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/rd.local.jpg)
   
     
   执行命令查看storage是否启动成功,出现如下图中，说明storage启动成功
     
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/storage-listen.jpg)
   
   执行
   
   >fdfs_monitor /etc/fdfs/storage.conf
   
   如果出现ip_addr = Active, 则表明storage服务器已经登记到tracker服务器
   
   4.配置client
   
   >vim /etc/fdfs/client.conf
   
       base_path=/usr/fastdfs/fastdfs_tracker  # 数据和日志文件存储根目录        
       tracker_server=192.168.192.128:22122    # tracker服务器IP和端口，有多个按行添加         
       http.tracker_server_port=9999           # 服务端IP和端口号
   
   接下来上传图片到服务器，我们先用rz命令上传一张图片到服务器，然后执行下面命令
   
   >/usr/bin/fdfs_upload_file  /etc/fdfs/client.conf  /root/fdfstest.png
   
   如果成功会返回图片路径
   
   group1/M00/00/00/wKjAgF1mhn-ABQZTAABWhlcgp00820.jpg
   
   可以在fastdfs_storage_data文件夹下看到刚才上传的图片
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/image-path.jpg)
   
   data下有256个1级目录，每级目录下又有256个2级子目录，总共65536个文件，新写的文件会以hash的方式被路由到其中某个子目录下，然后将文件数据直接作为一个本地文件存储到该目录中
   
   然后用 ip地址直接访问 
   
   http://192.168.192.128:9999/group1/M00/00/00/wKjAgF1mhn-ABQZTAABWhlcgp00820.jpg
   
   但是访问不了为什么呢，因为 早在4.05的时候，就remove embed HTTP support
   
   HTTP请求不能访问文件的原因
   
   我们在使用FastDFS部署一个分布式文件系统的时候，通过FastDFS的客户端API来进行文件的上传、下载、删除等操作。同时通过FastDFS的HTTP服务器来提供HTTP服务。但是FastDFS的HTTP服务较为简单，无法提供负载均衡等高性能的服务，所以FastDFS的开发者——淘宝的架构师余庆同学，为我们提供了Nginx上使用的FastDFS模块（也可以叫FastDFS的Nginx模块）。 
   
   FastDFS通过Tracker服务器,将文件放在Storage服务器存储,但是同组之间的服务器需要复制文件,有延迟的问题.假设Tracker服务器将文件上传到了192.168.128.131,文件ID已经返回客户端,这时,后台会将这个文件复制到192.168.128.131,如果复制没有完成,客户端就用这个ID在192.168.128.131取文件,肯定会出现错误。这个fastdfs-nginx-module可以重定向连接到源服务器取文件,避免客户端由于复制延迟的问题,出现错误。 
   
   正是这样，FastDFS需要结合nginx，所以取消原来对HTTP的直接支持。
   
   5. FastDFS的nginx模块安装
   
   在安装nginx之前要安装nginx所需的依赖lib
   
   >yum -y install pcre pcre-devel  
    yum -y install zlib zlib-devel  
    yum -y install openssl openssl-devel
    
   安装nginx并添加fastdfs-nginx-module
    
   解压nginx,和fastdfs-nginx-module
   >tar -zxvf nginx-1.12.0.tar.gz
    unzip fastdfs-nginx-module-master.zip
   
   然后进入nginx安装目录，添加fastdfs-nginx-module： 
   
   >./configure --prefix=/usr/local/nginx --add-module=/usr/software/fastdfs-nginx-module-master/src
   
   如果没有错误，开始安装
   >make
    make install
    
   修改nginx.conf
   
   >vim /usr/local/nginx/nginx.conf
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/nginx.jpg)
   
   如图，只需要改 listen,server_name 添加location知道上面上传的图片地址
   
   然后进入FastDFS安装时的解压过的目录，将http.conf和mime.types拷贝到/etc/fdfs目录下
   
   >cd /usr/fastdfs-5.11/conf/
   >cp http.conf /etc/fdfs/
   >cp mime.types /etc/fdfs/
   
   还需要把fastdfs-nginx-module src下的mod_fastdfs.conf 拷贝到fdfs目录下
   
   >cp /usr/software/fastdfs-nginx-module-master/src/mod_fastdfs.conf  /etc/fdfs/
   
   对刚刚拷贝的mod_fastdfs.conf文件进行修改
   
   >vim /etc/fdfs/mod_fastdfs.conf
   
    base_path=/usr/fastdfs/fastdfs_storage  #保存日志目录
    tracker_server=192.168.192.128:22122 #tracker服务器的IP地址以及端口号
    storage_server_port=23000 #storage服务器的端口号
    url_have_group_name = true #文件 url 中是否有 group 名
    store_path0=/usr/fastdfs/fastdfs_storage_data   #存储路径
    group_count = 3 #设置组的个数，事实上这次只使用了group1
    
   文件最后添加group
   
       [group1]
       group_name=group1
       storage_server_port=23000
       store_path_count=1
       store_path0=/usr/fastdfs/fastdfs_storage_data
       store_path1=/usr/fastdfs/fastdfs_storage_data
        
       [group2]
       group_name=group2
       storage_server_port=23000
       store_path_count=1
       store_path0=/usr/fastdfs/fastdfs_storage_data
        
       [group3]
       group_name=group3
       storage_server_port=23000
       store_path_count=1
       store_path0=/usr/fastdfs/fastdfs_storage_data
       
   创建M00至storage存储目录的符号连接：  
   
   >ln  -s  /usr/fastdfs/fastdfs_storage_data/data/ /usr/fastdfs/fastdfs_storage_data/data/M00
   
   启动nginx:
   
   >/usr/local/nginx/sbin/nginx
   
   启动后浏览  http://192.168.192.128:9999/
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/nginx-index.jpg)
   
   出现上图页面说明nginx部署成功，如果不显示需要添加该端口允许防火墙访问。
   
   这时候我们就可以浏览图片了 http://192.168.192.128:9999/group1/M00/00/00/wKjAgF1mhn-ABQZTAABWhlcgp00820.jpg
   
   ![INNER JOIN](https://chinakarl.github.io/images/posts/fastdfs/fdfs-test-image.jpg)
   
   图片展示出来了，好了大功告成了。