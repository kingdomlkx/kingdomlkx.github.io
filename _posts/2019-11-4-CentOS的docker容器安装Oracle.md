---
title: docker容器安装Oracle
auther: 雁引愁心去
layout: post
order: 12
---

本周上课老师开始上Oracle数据库课,然后花了一个上午的时间讲如何安装Oracle......  
说实话这是我见过的最难安装的软件......  
下面开始整体:

### 一. 获得Oracle ###
1. [Oracle中文网站](http://oracle.cn)   
前往官网下载Oracle,选择开发人员下载,我们选择12c的版本.Oracle一般人员下载和使用都是免费的,但如果是企业使用是需要购买的,不然被发现就会受到法院传票......下载之前需要注册Oracle账号.正常注册就好.

2. 选择 __Oracle Database 12c Release 2__,找到 __Linux x86-64__ 直接点下载就好.

### 二. 准备CentOS容器 ###
我们把他安装在docker中的CentOS容器中,优点有很多,显著的几点就是内存耗费少,运行速度快,使用方便.不需要费时费力的打开虚拟机.   

#### 1.  拉取镜像 #####  
执行命令 __docker pull centos__

#### 2. 创建容器 ####   
        这一步很重要必须,有一些必要的参数.    
        docker run -itd   \  
          --name cent7    \         容器的名字   
          -h cent7  \               centos的主机名  
          --privileged  \           !! 使container内的root拥有真正的root权限  
          -v /tmp/.X11-unix:/tmp/.X11-unix  \  !! 共享本地unix端口  
          -e DISPLAY=:0   \                    !! 修改环境变量DISPLAY  
          67fa59  \                 镜像编号
          /bin/bash                 shell   

因为docker是命令行工具,所以需要创建容器时使用一些参数使得centos中的GUI软件可以显示窗口界面.

#### 3. 进入容器   
__docker attach cent7__   

#### 4. 安装网络工具   
__yum install net-tools__

#### 5. 查看网络状态   
__ifconfig__

#### 6. 修改root密码   
__passwd root__

#### 7. `# yum install xclock`   
执行 `# xclock`   查看是否出现clock

### 三. 准备安装环境 ###
#### 1. 创建用户和组 ####  
        $ su  
        # groupadd oinstall   
        # groupadd dba  
        # useradd -m -g oinstall -G dba oracle  
        # passwd oracle   

#### 2. 配置内存参数 ####  
        # touch /etc/sysctl.d/97-oracledatabase-sysctl.conf
        # cat >> /etc/sysctl.d/97-oracledatabase-sysctl.conf << EOF  
            fs.aio-max-nr = 1048576  
            fs.file-max = 6815744    
            kernel.shmall = 2097152   
            kernel.shmmax = 4294967295   
            kernel.shmmni = 4096    
            kernel.sem = 250 32000 100 128    
            net.ipv4.ip_local_port_range = 9000 65500   
            net.core.rmem_default = 4194304 12c 262144  
            net.core.rmem_max = 4194304    
            net.core.wmem_default = 262144    
            net.core.wmem_max = 1048586
            EOF
        # sysctl --system

#### 3. 配置资源限制   
        # cat >> /etc/security/limits.conf << EOF             
            oracle  soft  nproc  2047   
            oracle  hard  nproc  16384   
            oracle  soft  nofile  1024   
            oracle  hard  nofile  65536   
            oracle  soft  stack   3145728
            oracle  hard  stack   3145728
            EOF

#### 4. 修改本地hosts文件  
__`# vi /etc/hosts`__   
加入一条记录 __`ip(container的ip地址)     cent7(container的主机名)`__

#### 5. 创建安装目录   
        # mkdir -p /u01/app/oracle
        # chown -R oracle:oinstall /u01/app/
        # chmod -R 755 /u01/app/oracle/
        # chmod -R 755 /u01/        

#### 6. 配置变量环境   
        # su - oracle     
        # cat >> /home/oracle/.bash_profile << EOF
            export EDITOR=vim
            #oracle的BASE目录定义  
            export ORACLE_BASE=/u01/app/oracle
            #oracle的HOME目录定义  
            export ORACLE_HOME=\$ORACLE_BASE/product/12.2.0.1/db_1  
            export ORACLE_SID=ORCL
            #重新定义系统环境变量
            export PATH=\$PATH:\$HOME/.local/bin:\$HOME/bin:\$ORACLE_HOME/bin
            #简体中文版`
            #export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"  
            export NLS_LANG="SIMPLIFIED CHINESE_CHINA".UTF8
            #定义语系
            export LANG=zh_CN.UTF-8
            #export LANG=en_US   
            #export LC_ALL=en_US
            export LD_LIBRARY_PATH=/lib:/usr/lib:\$ORACLE_HOME/lib   
            export CLASSPATH=\$ORACLE_HOME/JRE:\$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib   
            #alias sqlplus="rlwrap sqlplus"   
            #alias rman="rlwrap rman"    
            #权限（反码）
            umask 022    
            EOF
        # source .bash_profile    

#### 7. centos容器中可能存在的一些选项
    1. 关闭SELINUX  
      __# vi /etc/selinux/config__     
        添加一条`SELINUX=disabled`     
        _如果没有SELINUX,则不需要此操作_   
    2. 关闭防火墙      
      __`# systemctl stop firewalld.service`__      
      __`# systemctl disable firewalld.service`__     
      _没有firewalld则不需要此操作_      
    3. 关闭透明大页     
        1. 查看       
          __`# cat /sys/kernel/mm/transparent_hugepage/enabled`__       
          如果出现'[always]'代表启用,出现'[never]'代表禁用     
        2. 出现'[always]'时执行以下命令:       
          __`# vi /etc/default/grub`__        
          找到`GRUB_CMDLINE_LINUX=`这一行,在 __quiet__ 后添加`transparent_hugepage=never`  
          `# grub2-mkconfig -o /boot/grub2/grub.cfg`     
        3. 重启   

#### 8. 安装软件包    
          yum -y install binutils compat-libcap1 compat-libstdc++-33 gcc gcc-c++ glibc glibc-devel ksh libaio libaio-devel libgcc libstdc++ libstdc++-devel libXi libXtst make sysstat unixODBC unixODBC-devel   

### 四. 传输软件包 ###
在本地主机上配置ftp服务器,用centos容器进行ftp登陆来get软件包.   
解压至`/u01/app/`    
`$ unzip linuxx64_12201_database.zip /u01/app/`
注意用oracle用户解压,不然会出现权限问题.   
可能需要下载 __unzip__ ,直接`yum install unzip `即可.

### 五. 安装软件包 ###
        # su - oracle
        $ export DISPLAY=:0.0
        $ export LANG=en_US 或者 export LANG=zh_CN.UTF-8
        $ cd /u01/app/database
        $ ./runInstaller

            1. 如果出现内存问题,即第二个没有passed.
            在本地主机中执行下列命令.
            # dd if=/dev/zero of=/swapfile bs=1024 count=512k
            # mkswap /swapfile
            # swapon /swapfile
            # chown oracle:oinstall /swapfile
            # chmod 0600 /swapfile

            2. 如果出现display问题,即第三个没有passed
            在container中执行下列命令:
            $ su
            # yum install xdpyinfo
            # xdpyinfo | grep "name of display"
            > name of display:    :0.0      #取后面的`:0.0`

            # su - oracle
            $ export DISPLAY=:0.0

            $ ./runInstaller

            3. 如果必要性检查中出现Swap Size不够的情况,
            在本地主机中执行下列命令:
            # dd if=/dev/zero of=/swapfiles bs=1024 count=8192000
            # mkswap /swapfiles
            # swapon /swapfiles

        跳出窗口时,窗口显示了以下两条命令,在container中以root用户执行:
        # /u01/app/oraInventory/orainstRoot.sh
        # /u01/app/oracle/product/12.2.0.1/db_1/root.sh

        最后 finish.

        添加路径:
        # vi /etc/profile
        export PATH=$PATH:/u01/app/oracle/product/12.2.0/dbhome_1/bin

### 五.创建监听服务 ###
        $ netca  
        # 参数一律默认即可    
        $ ps -ef |grep ora_

### 六.创建数据库 ###
        $ dbca  
        # 选择gernel purpose，与 高级模式(advance)  在SID和全局名称中，选择你在.bash_profile中创建的SID.    
        # 数据库的字符集选项中选择ZHS16GBK, 其它默认OK,完成.
### 七.oracle的启动服务配置 ###
        $ vi /etc/oratab  
        #把最后的N改成Y (默认是N)    
        orcl:/u01/app/oracle/product/12.2.0.1/db_1:Y

        cat >> /etc/rc.local << EOF  
            # Oracle service start    
            su - oracle -c "lsnrctl start"    
            su - oracle -c "dbstart"    
            EOF

### 八.安装flash-player播放器插件 ###
        # rpm -ivh flash-player-npapi-26.0.0.131-release.x86_64.rpm
        这个rpm包可能找不到,我在这里上传一个.
        链接:https://pan.baidu.com/s/16B7ST1XrIRbQ3H5SgWv21g 提取码:19fn
        >SQL alter user sys identified by waterman

### 九.安装完数据库密码过期问题 ###
        >SQL alter profile default limit password_life_time unlimited;
        >SQL alter profile default limit failed_login_attempts unlimited;

### 十.取消段延迟特性 ###
        >SQL alter system set deferred_segment_creation=false scope=both;

### 十一.设置大小写忽略（12c已废弃该参数，但兼容） ###
        >SQL alter system set sec_case_sensitive_logon=false scope=both;

### 十二.实例启动关闭 ###
        sqlplus / as sysdba
        startup
        shutdown immediate
        select status from v$instance     # 查看实例状态
