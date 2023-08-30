# ansible-playbook

# 功能说明

## 操作系统配置

操作系统层面主要完成防火墙、selinux等的自动关闭，但是生产过程中应该还需要修改/etc/security/limits.conf中的用户资源限制（本次并没有去做这个配置的修改，后续会考虑加进去）、自动配置免密、自动配置/etc/hosts、自动配置/etc/ansible/hosts。

## 相关软件环境的配置

软件环境的配置主要完成自动配置：

1、JAVA_HOME\PATH等环境变量，并添加至/etc/profile；

2、ntp时钟同步；

3、本地yum源；（需要在ambari-server节点连接iso镜像文件至/dev/cdrom，如默认连接该设备的路径在其他地方，需要修改autoDeployAmbari.sh脚本中的相关路径）

4、自动安装MySQL、创建hive、hue、ambari等数据库以及初始化MySQL相关账号的密码。（密码的定义在ambari.yml中，可自行修改）

# 使用须知

## 环境

**操作系统：centos7.9**

**Python3：3.6.8**

**MySQL: 5.7.30**

# 部署方式

## 离线部署

该自动化工具为离线部署，即在不依赖外部互联网的情况下完成部署。



由于github的限制，并没有上传jdk、hdp、mysql相关的包。在ambari.yml中有定义相关包的版本，可以自行获取或者修改相关版本并放置于相关目录。



用法上需要人工配置以下几处：
1）下载这个包之后解压上传至准备作为ambari-server节点的服务器中的/opt/目录，结构如下：

[root@hdp3-node1 ansible-playbook-ambari]# ll
总用量 152
-rw-r--r--. 1 root root   4959 8月  26 17:01 ambari.yml
drwxr-xr-x. 2 root root   4096 8月  28 10:26 ansible-rpm
-rw-r--r--. 1 root root   2823 8月  28 13:52 autoDeployAmbari.sh
drwxr-xr-x. 9 root root    108 8月  28 10:26 autoDeployFiles
drwxr-xr-x. 2 root root   4096 8月  28 16:13 component
[root@hdp3-node1 ansible-playbook-ambari]# pwd
/opt/ansible-playbook/ansible-playbook-ambari

2）人工配置autoDeployFiles/Scripts/hostlist.txt，将所有作为ambari-server\ambari-agent节点的服务器主机名与密码填写好，用于制作ambari-server节点的单向免密ssh远程root用户访问ambari-agent节点。
例如：
hdp3-node1 123456
hdp3-node2 123456
hdp3-node3 123456

3）人工配置autoDeployFiles/Scripts/temphosts.txt，填写好所有ambari-server\ambari-agent节点的ip、hostname映射关系
例如：
192.168.0.1 hdp3-node1
192.168.0.2 hdp3-node2
192.168.0.3 hdp3-node3

完成以上步骤，即可执行部署，以下是我本地的运行过程，仅供参考。



另外说明一下：部署完ambari-server并且成功启动之后，在安装HDP hue服务的过程中，会提示缺失python2-psycopg2依赖包，hidataplus有提供，大家可以去百度网盘获取。



[root@hdp3-node1 ansible-playbook-ambari]# sh autoDeployAmbari.sh 
已加载插件：fastestmirror
正在检查 /opt/ansible-playbook/ansible-playbook-ambari/autoDeployFiles/Scripts//../rpmPackages/vsftpd-3.0.2-28.el7.x86_64.rpm: vsftpd-3.0.2-28.el7.x86_64
/opt/ansible-playbook/ansible-playbook-ambari/autoDeployFiles/Scripts//../rpmPackages/vsftpd-3.0.2-28.el7.x86_64.rpm 将被安装
正在解决依赖关系
--> 正在检查事务
---> 软件包 vsftpd.x86_64.0.3.0.2-28.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

============================================================================================================================

 Package                                     架构                                        版本                                              源                                                                大小

正在安装:
 vsftpd                                      x86_64                                      3.0.2-28.el7                                      /vsftpd-3.0.2-28.el7.x86_64                                      353 k

事务概要

安装  1 软件包

总计：353 k
安装大小：353 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : vsftpd-3.0.2-28.el7.x86_64                                                                                                                                                                    1/1 
  验证中      : vsftpd-3.0.2-28.el7.x86_64                                                                                                                                                                    1/1 

已安装:
  vsftpd.x86_64 0:3.0.2-28.el7                                                                                                                                                                                    

完毕！
创建本地iso挂载目录
mount: /dev/sr0 写保护，将以只读方式挂载
挂载iso至/var/ftp/centos7
创建默认yum源备份目录
备份默认yum源
已加载插件：fastestmirror
正在清理软件源： centos
Cleaning up list of fastest mirrors
已加载插件：fastestmirror
Determining fastest mirrors
centos                                                                                                                                                                                     | 3.6 kB  00:00:00     
(1/4): centos/group_gz                                                                                                                                                                     | 153 kB  00:00:00     
(2/4): centos/primary_db                                                                                                                                                                   | 3.3 MB  00:00:00     
(3/4): centos/filelists_db                                                                                                                                                                 | 3.3 MB  00:00:00     
(4/4): centos/other_db                                                                                                                                                                     | 1.3 MB  00:00:00     
元数据缓存已建立
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 expect.x86_64.0.5.45-14.el7_1 将被 安装
--> 正在处理依赖关系 libtcl8.5.so()(64bit)，它被软件包 expect-5.45-14.el7_1.x86_64 需要
--> 正在检查事务
---> 软件包 tcl.x86_64.1.8.5.13-8.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

============================================================================================================================

 Package                                         架构                                            版本                                                       源                                               大小

正在安装:
 expect                                          x86_64                                          5.45-14.el7_1                                              centos                                          262 k
为依赖而安装:
 tcl                                             x86_64                                          1:8.5.13-8.el7                                             centos                                          1.9 M

事务概要

安装  1 软件包 (+1 依赖软件包)

总下载量：2.1 M
安装大小：4.9 M
Downloading packages:
(1/2): expect-5.45-14.el7_1.x86_64.rpm                                                                                                                                                     | 262 kB  00:00:00     

(2/2): tcl-8.5.13-8.el7.x86_64.rpm                                                                                                                                                         | 1.9 MB  00:00:00     

总计                                                                                                                                                                               15 MB/s | 2.1 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 1:tcl-8.5.13-8.el7.x86_64                                                                                                                                                                     1/2 
  正在安装    : expect-5.45-14.el7_1.x86_64                                                                                                                                                                   2/2 
  验证中      : 1:tcl-8.5.13-8.el7.x86_64                                                                                                                                                                     1/2 
  验证中      : expect-5.45-14.el7_1.x86_64                                                                                                                                                                   2/2 

已安装:
  expect.x86_64 0:5.45-14.el7_1                                                                                                                                                                                   

作为依赖被安装:
  tcl.x86_64 1:8.5.13-8.el7                                                                                                                                                                                       

完毕！
Generating public/private rsa key pair.
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:kTlaw1pHxUGUOEchmSBr9bWcDUx4d9g6Wl1AU1tNsII root@hdp3-node1
The key's randomart image is:
+---[RSA 2048]----+
|      . o.o&@==B=|
|       = =*=*=ooB|
|      o X E+=.o=.|
|     . = =   .+ .|
|      o S    o . |
|            .    |
|                 |
|                 |
|                 |
+----[SHA256]-----+
spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp3-node1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'hdp3-node1 (192.168.0.97)' can't be established.
ECDSA key fingerprint is SHA256:n4xebIj0+xNnB/e08ts437M2qDk40wgr53GacZvBjqI.
ECDSA key fingerprint is MD5:f7:1f:7d:d4:3c:98:06:a7:c5:84:7f:34:d1:9c:80:68.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@hdp3-node1's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@hdp3-node1'"
and check to make sure that only the key(s) you wanted were added.

spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp3-node2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'hdp3-node2 (192.168.0.98)' can't be established.
ECDSA key fingerprint is SHA256:n4xebIj0+xNnB/e08ts437M2qDk40wgr53GacZvBjqI.
ECDSA key fingerprint is MD5:f7:1f:7d:d4:3c:98:06:a7:c5:84:7f:34:d1:9c:80:68.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@hdp3-node2's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@hdp3-node2'"
and check to make sure that only the key(s) you wanted were added.

spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@hdp3-node3
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'hdp3-node3 (192.168.0.99)' can't be established.
ECDSA key fingerprint is SHA256:n4xebIj0+xNnB/e08ts437M2qDk40wgr53GacZvBjqI.
ECDSA key fingerprint is MD5:f7:1f:7d:d4:3c:98:06:a7:c5:84:7f:34:d1:9c:80:68.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@hdp3-node3's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@hdp3-node3'"
and check to make sure that only the key(s) you wanted were added.

hosts                                                                                                                                                                           100%  230   387.2KB/s   00:00    
hosts                                                                                                                                                                           100%  230   899.8KB/s   00:00    

PLAY [master] *********************************************************

TASK [创建相关目录] *********************************************************************
changed: [hdp3-node1]

TASK [解压ambari-server包] ******************************************************
changed: [hdp3-node1]

TASK [创建目录/var/www/html] ***************************************************************
changed: [hdp3-node1]

TASK [解压HDPUTILS包] *********************************
changed: [hdp3-node1]

TASK [配置本地操作系统与ambari的yum源] *********************************************************************
changed: [hdp3-node1]

TASK [安装httpd服务] ******************************************
changed: [hdp3-node1]

TASK [安装ntp时钟同步服务] ***************************************************
changed: [hdp3-node1]

TASK [配置时钟同步源] *********************************************
changed: [hdp3-node1]

PLAY [all_node] *********************************************************

TASK [关闭防火墙并禁止开机启动、关闭selinux] ******************************
changed: [hdp3-node1]
changed: [hdp3-node3]
changed: [hdp3-node2]

TASK [创建JDK所需临时目录] *********************************************
ok: [hdp3-node1]
ok: [hdp3-node3]
changed: [hdp3-node2]

TASK [创建存放rpm包临时目录] *********************************************
ok: [hdp3-node1]
ok: [hdp3-node3]
changed: [hdp3-node2]

TASK [拷贝JDK至相关目录] *********************
ok: [hdp3-node3]
changed: [hdp3-node1]
changed: [hdp3-node2]

TASK [创建相关JAVA_HOME目录] ******************************************
changed: [hdp3-node1]
changed: [hdp3-node2]
ok: [hdp3-node3]

TASK [解压jdk包] ************************************************
ok: [hdp3-node3]
changed: [hdp3-node1]
changed: [hdp3-node2]

TASK [设置/etc/profile的JAVA_HOME与PATH变量] ************************************
changed: [hdp3-node1]
changed: [hdp3-node3]
changed: [hdp3-node2]

TASK [copy] ******************************************************
ok: [hdp3-node1]
changed: [hdp3-node2]
ok: [hdp3-node3]

TASK [安装vsftpd包] ***************************************
ok: [hdp3-node1]
ok: [hdp3-node3]
changed: [hdp3-node2]

TASK [创建临时目录] ************************
ok: [hdp3-node1]
changed: [hdp3-node2]
ok: [hdp3-node3]

TASK [拷贝配置脚本和文件] *********************************
ok: [hdp3-node1]
changed: [hdp3-node2]
ok: [hdp3-node3]

TASK [配置本地操作系统与ambari的yum源] ***************************************
changed: [hdp3-node3]
changed: [hdp3-node2]
changed: [hdp3-node1]

TASK [拷贝rpm包] ************************************************************
ok: [hdp3-node1]
ok: [hdp3-node3]
changed: [hdp3-node2]

TASK [安装libtirpc包] ************************************
ok: [hdp3-node3]
changed: [hdp3-node1]
changed: [hdp3-node2]

TASK [安装libtirpc-devel包] ******************************************************
ok: [hdp3-node3]
changed: [hdp3-node1]
changed: [hdp3-node2]

TASK [安装Python3] ***************************************
ok: [hdp3-node3]
changed: [hdp3-node1]
changed: [hdp3-node2]

PLAY [master] ******************************************************************

TASK [卸载可能与 mysql 冲突的包] ************************************************
ok: [hdp3-node1] => (item=mysql-community*)

TASK [删除 /data/mysql] ***************************************************
ok: [hdp3-node1]

TASK [安装 mysql 及其依赖] ******************************************************
[WARNING]: Consider using the yum module rather than running 'yum'.  If you need to use command because yum is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in
ansible.cfg to get rid of this message.
changed: [hdp3-node1]

TASK [创建 /data/mysql] ************************************************************************
changed: [hdp3-node1] => (item=/data/mysql)
changed: [hdp3-node1] => (item=/data/mysql/data)
changed: [hdp3-node1] => (item=/data/mysql/binlog)
changed: [hdp3-node1] => (item=/data/mysql/log)
changed: [hdp3-node1] => (item=/data/mysql/tmp)

TASK [删除 /var/lib/mysql/ib_logfile0] *********************************************************
ok: [hdp3-node1]

TASK [删除 /var/lib/mysql/ib_logfile1] ************************************************************
ok: [hdp3-node1]

TASK [判断 /etc/my.cnf 是否存在] ***************************************************************
ok: [hdp3-node1]

TASK [备份 /etc/my.cnf] ************************************************************
changed: [hdp3-node1]

TASK [复制新 my.cnf 到 /etc 下] ************************************************************
changed: [hdp3-node1]

TASK [修改 /etc/my.cnf 权限为 644] ***************************************************
ok: [hdp3-node1]

TASK [启动 mysql server] ************************************************************
changed: [hdp3-node1]

TASK [获取 mysql 初始密码] ******************************************************
changed: [hdp3-node1]

TASK [保存 mysql 初始密码] ***************************************************************
changed: [hdp3-node1]

TASK [设置 mysql root 用户密码] ************************************************************
changed: [hdp3-node1]

TASK [启动 mysql server] *********************************************************************
changed: [hdp3-node1]

TASK [安装python-mysql模块] ***************************************************
changed: [hdp3-node1]

TASK [创建 ambari 数据库] ************************************************************
changed: [hdp3-node1]

TASK [创建 hive 数据库] ***************************************************************
changed: [hdp3-node1]

TASK [创建 hue 数据库] ******************************************************
changed: [hdp3-node1]

TASK [创建 hive 数据库] ************************************************
ok: [hdp3-node1]

TASK [创建 navms 数据库] ******************************************************
changed: [hdp3-node1]

TASK [开放 ambari 库下所有表的权限给 ambari 用户] *********************************************
[WARNING]: Module did not set no_log for update_password
changed: [hdp3-node1]

TASK [开放 hive 库下所有表的权限给 hive 用户] ************************************************
changed: [hdp3-node1]

TASK [开放 hue 库下所有表的权限给 hue 用户] *********************************************
changed: [hdp3-node1]

TASK [安装Ambari Server服务] ***************************************************************
changed: [hdp3-node1]

TASK [创建mysql driver的目录/usr/share/java] ***************************************************************
changed: [hdp3-node1]

TASK [复制mysql-driver包至/usr/share/java] *********************************************************
changed: [hdp3-node1]

TASK [初始化Ambar server元数据] ************************************************************
changed: [hdp3-node1]

TASK [设置Ambari Server] ******************************************
changed: [hdp3-node1]

TASK [安装ambari agent] ***************************************************************
changed: [hdp3-node1]

PLAY [slave] *********************************************************************

TASK [创建配置文件所需目录] ******************************************************************
changed: [hdp3-node2]
ok: [hdp3-node3]

TASK [安装ntp服务] ******************************************************
ok: [hdp3-node3]
changed: [hdp3-node2]

TASK [拷贝配置文件] ******************************************
changed: [hdp3-node2]
changed: [hdp3-node3]

TASK [配置slave节点同步ntp server] ************************************************************************************
[WARNING]: Consider using the replace, lineinfile or template module rather than running 'sed'.  If you need to use command because replace, lineinfile or template is insufficient you can add 'warn: false' to
this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [hdp3-node2]
changed: [hdp3-node3]

TASK [生效配置并检查] ************************************************************************************
changed: [hdp3-node2]
changed: [hdp3-node3]

PLAY [master] ************************************************************

TASK [修改ambari连接MySQL的参数] ************************************************************
changed: [hdp3-node1]

TASK [启动ambari] *********************************************************************
changed: [hdp3-node1]

PLAY RECAP *********************************************************
hdp3-node1                 : ok=56   changed=42   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hdp3-node2                 : ok=21   changed=21   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
hdp3-node3                 : ok=21   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@hdp3-node1 ansible-playbook-ambari]#
