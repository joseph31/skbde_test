# BDE_1_InstallCM.md
    20190703-DataEngineering
    ToC
        ## System pre-configuration checks
        ## Path B install using CM 5.15x (part 1)
        ## Install a Cluster & deploy CDH (part 2)
            Step 1: Configure a Repository
            Step 2: Install JDK
            Step 3: Install Cloudera Manager Server
            Step 4: Install Databases
            Step 5: Set up the Cloudera Manager Database
            Step 6: Install CDH and Other Software
            Step 7: Set Up a Cluster

## Intro 
### [Basic Info] 
    What: Big Data Engineering Hands-on
    Who: group 4 - 정명훈(리더), 전평재, 양진욱
    Which ones: 1) CM & related env. installation 2) Yelp data analysis 3) Spark 

### [IP addresses by host]
    
    IP addresses (host: 1~5, public & private)
    52.78.148.50	172.31.0.116
    52.78.3.56      172.31.3.209
    52.78.52.113	172.31.12.210
    52.78.66.117	172.31.8.176
    54.180.166.152	172.31.2.67
  
~~previous IPs</br>
host 1: 13.209.93.133    172.31.15.117<br/>
host 2: 15.164.136.136   172.31.0.89<br/>
host 3: 15.164.161.233   172.31.0.71<br/>
host 4: 15.164.172.180   172.31.8.189<br/>
host 5: 15.164.189.170   172.31.12.75~~<br/>
~~(참고: CM설치시 JDK 1.7 설치/충돌로~새 IP 부여받음. 아래 일부내용에는 이 IP가 남아 있을 수 있음)~~<br/>

  
## System pre-configuration checks

### [Connect to each server]
    (by terminal)
        # chmod 400 key.pem 
        # ssh -i /path/to/key.pem centos@13.209.93.133 
    (by secureCRT)

### [Update yum] 
    # sudo yum -y update
    
    cf. 아래는 필요 시 실행 
    # sudo yum install -y wget
    # sudo yum install -y unzip 

### [Change the run-level to multi-user text mode]
    # sudo systemctl get-default
    multi-user.target (* 아닌 경우 아래 실행) 
    # sudo systemctl set-default multi-user.target 
      
### [Disable SE linux]
    # sestatus 
    SELinux status:                 disabled  (* 아닌 경우 아래 실행)
    
    # sudo vi /etc/selinux/config 
    ...
    SELINUXTYPE=targeted (* =disabled 로 수정) 
    
    cf. Set SELinux in permissive mode (effectively disabling it)
    setenforce 0
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

    cf. 
    vi /etc/sysconfig/selinux
    SELINUX=enforcing 을 SELINUX=disabled 로 변경후 저장한다.
    :%s/SELINUX=enforcing/SELINUX=disabled/s
    reboot
    disabled
      
### [Disable firewall]
    (* 본 실습에서는 이미 firewall이 disable되어 있음) 
    방화벽 중지: # sudo systemctl stop firewalld
    Failed to stop firewalld.service: Unit firewalld.service not loaded.
    방화벽 자동시작 해제(재부팅시 켜지지 않음): # systemctl disable firewalld

### [Check vm.swappiness & update permanently]
    # sudo vi /etc/sysctl.conf
    ...
    vm.swappiness=1 (* 없는경우 추가) 
    
    cf. 아래는 일시적인 방법
    # cat /proc/sys/vm/swappiness
    1 (* 아닌 경우 아래 실행) 
    # sudo sysctl -w vm.swappiness=1
          
### [Disable transparent hugepage support permanently]
    * THP: Transparent Huge Page 
    # cat /sys/kernel/mm/transparent_hugepage/enabled 
    [always] madvise never  (* THP 활성화 상태) 
    always madvise [never]  (* THP 비활성화 상태)
    
    # cat /proc/meminfo | grep Huge
    AnonHugePages:      6144 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB

    # sudo vi /etc/rc.d/rc.local
    ... (* 아래 항목 추가)
    echo "never" > /sys/kernel/mm/transparent_hugepage/enabled
    echo "never" > /sys/kernel/mm/transparent_hugepage/defrag
    
    cf. 아래 'Action.#' 참고(명훈님) 
            Action.1
            mkdir /etc/tuned/transparent_hugepage 

            vi /etc/tuned/transparent_hugepage/tuned.conf 
            [main]
            include=throughput-performance
            [vm]
            transparent_hugepages=never 

            chmod +x /etc/tuned/transparent_hugepage/tuned.conf 

            tuned-adm profile transparent_hugepage 

            vi /etc/sysconfig/grup



            Action.2 
            /bin/echo never > /sys/kernel/mm/transparent_hugepage/enabled

            Action.3
            vi /etc/sysconfig/mic/default.conf  <== X



            Action.4 
            https://www.cloudera.com/documentation/enterprise/5-16-x/topics/cdh_admin_performance.html#cdh_performance__section_hw3_sdf_jq

            echo never > /sys/kernel/mm/transparent_hugepage/enabled
            echo never > /sys/kernel/mm/transparent_hugepage/defrag
            chmod +x /etc/rc.d/rc.local

            cat /etc/default/grub

            vi /etc/default/grub
            :%s/GRUB_CMDLINE_LINUX="console=tty0 crashkernel=auto console=ttyS0,115200"/GRUB_CMDLINE_LINUX="console=tty0 crashkernel=auto console=ttyS0,115200 transparent_hugepage=never"/c

            grub2-mkconfig -o /boot/grub2/grub.cfg



            echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag
            echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled

            sh -c "echo 'never' > /sys/kernel/mm/transparent_hugepage/defrag"
            sh -c "echo 'never' > /sys/kernel/mm/transparent_hugepage/enabled" 


            cat /sys/kernel/mm/transparent_hugepage/defrag
            cat /sys/kernel/mm/transparent_hugepage/enabled  



            [이슈] defrag 가 reboot 하고 나면 
            [root@ip-172-31-15-117 ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
            [always] madvise never
            [root@ip-172-31-15-117 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled  
            always madvise [never]


            echo never > /sys/kernel/mm/transparent_hugepage/defrag
            vi /etc/default/grub

            grub2-mkconfig -o /boot/grub2/grub.cfg

            [Solution]
            vi /etc/rc.d/rc.local
            echo never > /sys/kernel/mm/transparent_hugepage/enabled
            echo never > /sys/kernel/mm/transparent_hugepage/defrag
      
### [Check to see that nscd service is running]
    nscd.service - Name Service Cache Daemon 
    # sudo yum install -y nscd
    # sudo systemctl enable nscd
    # sudo systemctl start nscd
    # sudo systemctl status nscd

### [Check to see that ntp service is running - disable chrony as necessary]
    * ntpd.service - Network Time Service 
    # sudo systemctl status ntpd
    # sudo yum install ntp
    # sudo systemctl enable ntpd
    # sudo systemctl start ntpd
    # sudo systemctl status ntpd
    ...
    Active: active (running) since ... 
    
    * chronyd.service - NTP client/server
    # sudo ssystemctl status chronyd
    ...
    Active: inactive (dead)
      
### [Disable IPv6]
    # sudo sysctl net.ipv6.conf.all.disable_ipv6
    net.ipv6.conf.all.disable_ipv6 = 0              (* disable_ipv6 = 0: active
    # sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1    (* disable_ipv6 = 1: inactive) 
    net.ipv6.conf.all.disable_ipv6 = 0 
    
    # sudo vi /etc/sysctl.conf
    ...
    vm.swappiness = 1
    ... 
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    
    # sysctl -p
    vm.swappiness = 1
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
      
### [desc 10.] 
    During the instavllation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. 
    In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server 
    or setup a private/public key access. 
    Whichever method you choose, make sure you test access with ssh before proceeding.
      
### [Show that forward and reverse host lookups are correctly resolved] 
    In this lab, we will use /etc/hosts Files setting to accomplish this 
    Add the necessary information to the /etc/hosts files 
    Check to make sure that File lookup has priority 
    Use getent to make sure you are getting proper host name and ip address 
            
### [Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file]
    # vi /etc/hosts (* 작성)
    172.31.15.117    cm.skplanet.com        cm
    172.31.0.89      master1.skplanet.com   master1
    172.31.0.71      worker1.skplanet.com   worker1
    172.31.8.189     worker2.skplanet.com   worker2
    172.31.12.75     worker3.skplanet.com   worker3
    # getent hosts (* 확인) 
   
    # hostnamectl set-hostname worker3.skplanet.com
    # vi /etc/sysconfig/network 
    HOSTNAME=worker3.skplanet.com


## Path B install using CM 5.15x (part 1)
    The Full Rundown is here: 
    https://www.cloudera.com/documentation/enterprise/5-15-x/topics/install_cm_cdh.html 
    
    Step 1: Configure a Repository
    Step 2: Install JDK
    Step 3: Install Cloudera Manager Server
    Step 4: Install Databases
    Step 5: Set up the Cloudera Manager Database
    Step 6: Install CDH and Other Software
    Step 7: Set Up a Cluster

### [Step 1: Configure CM Repository]
    version = 5.15.2: https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.15.2/
    Documentation: https://www.cloudera.com/documentation/enterprise/5-15-x/topics/configure_cm_repo.html#cm_repo
    
    # sudo yum install -y wget
    # sudo wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/
    # sudo rpm --import https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
    
### [Step 2: Install JDK]
    version: 1.8
    [localhost]# scp  -i ./key.pem ./jdk-8u211-linux-x64.tar.gz centos@13.209.93.133:/home/centos
    # tar -zxvf jdk-8u202-linux-x64.tar.gz
    # mkdir /app
    # mv jdk1.8.0_202 /app/
    # vi /etc/profile   (* 작성) 
    JAVA_HOME=/app/jdk1.8.0_202
    PATH=$PATH:$JAVA_HOME/bin
    source /etc/profile
    # java -version     (* 확인) 
    java version "1.8.0_211"
          
### [Step 3: Install CM] 
    # sudo yum install -y cloudera-manager-daemons cloudera-manager-agent
    
    cf. troubleshooting 
        [error]
        [root@cm ~]#  yum install cloudera-manager-daemons cloudera-manager-agent
        ...
        Error: Package: cloudera-manager-agent-5.16.2-1.cm5162.p0.7.el6.x86_64 (cloudera-manager)
                   Requires: libpython2.6.so.1.0()(64bit)
         You could try using --skip-broken to work around the problem
         You could try running: rpm -Va --nofiles --nodigest

        [action.1]
        wget https://www.python.org/ftp/python/2.6.9/Python-2.6.9.tgz

        [action.2]
        rpm -Uvh https://archive.fedoraproject.org/pub/archive/epel/5/x86_64/python26-2.6.8-2.el5.x86_64.rpm

        [원인]
        6 버전으로 다운 받아서 생긴 이슈  
        wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo -P /etc/yum.repos.d/

### [Step 4: Install & configure Databases(MySQL)]
    # rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
    # yum install -y mysql-community-server
    # systemctl start mysqld
    # grep 'temporary password' /var/log/mysqld.log
    [root@cm ~]# grep 'temporary password' /var/log/mysqld.log
    2019-07-03T05:03:31.890985Z 1 [Note] A temporary password is generated for root@localhost: lNWoGSk0Bp(b
    
    SET PASSWORD = PASSWORD('Admin123!');
    FLUSH PRIVILEGES;
    mysql> show grants;
    +---------------------------------------------------------------------+
    | Grants for root@localhost                                           |
    +---------------------------------------------------------------------+
    | GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
    | GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
    +---------------------------------------------------------------------+
    2 rows in set (0.00 sec)

    # vi /etc/my.cnf
    validate-password=off
    # systemctl restart mysqld

#### [Setting mysqld]
    
    # vi /etc/my.cnf 

    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock
    transaction-isolation = READ-COMMITTED
    Disabling symbolic-links is recommended to prevent assorted security risks;
    to do so, uncomment this line:
    symbolic-links = 0

    key_buffer_size = 32M
    max_allowed_packet = 32M
    thread_stack = 256K
    thread_cache_size = 64
    query_cache_limit = 8M
    query_cache_size = 64M
    query_cache_type = 1

    max_connections = 550
    #expire_logs_days = 10
    #max_binlog_size = 100M

    #log_bin should be on a disk with enough free space.
    #Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
    #system and chown the specified folder to the mysql user.
    log_bin=/var/lib/mysql/mysql_binary_log

    #In later versions of MySQL, if you enable the binary log and do not set
    #a server_id, MySQL will not start. The server_id must be unique within
    #the replicating group.
    server_id=1
    binlog_format = mixed

    read_buffer_size = 2M
    read_rnd_buffer_size = 16M
    sort_buffer_size = 8M
    join_buffer_size = 8M

    #InnoDB settings
    innodb_file_per_table = 1
    innodb_flush_log_at_trx_commit  = 2
    innodb_log_buffer_size = 64M
    innodb_buffer_pool_size = 4G
    innodb_thread_concurrency = 8
    innodb_flush_method = O_DIRECT
    innodb_log_file_size = 512M

#### [mysqld_safe]
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid

    sql_mode=STRICT_ALL_TABLES
    validate-password=off

    * mysql connector
    # wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
    # tar zxvf mysql-connector-java-5.1.47.tar.gz 
    # mkdir -p /usr/share/java/
    # cd mysql-connector-java-5.1.47
    # cp mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar

#### [create DB]
    https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_mysql.html#cmig_topic_5_5
    CREATE DATABASE scm DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE amon DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE rman DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE hue DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE metastore DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE sentry DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE hnavue DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE navms DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    CREATE DATABASE oozie DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    
    password : Hadoop123!

    GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON hnavue.* TO 'hnavue'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'Hadoop123!';

    Hadoop123!

    ERROR 1044 (42000): Access denied for user 'root'@'%' to database 'scm'
    ALTER USER 'root'@'%' IDENTIFIED BY 'Admin123!';
    CREATE USER 'scm'@'%' IDENTIFIED BY 'Hadoop123!';
    GRANT ALL PRIVILEGES ON scm.* TO 'scm'@'%';

    flush privileges;

    update user set authentication_string=password('Admin123!') where user='root';
    create  user 'root'@'%' identified with mysql_native_password by 'Admin123!';

    CREATE USER 'root'@'%' IDENTIFIED BY 'Admin123!'  ;
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';


## Install a Cluster & deploy CDH (part 2)

### [Step 5: Set up the Cloudera Manager Database]
    # /usr/share/cmf/schema/scm_prepare_database.sh mysql scm scm
    # /usr/share/cmf/schema/scm_prepare_database.sh -h cm.skplanet.com mysql metastore hive 
    # /usr/share/cmf/schema/scm_prepare_database.sh -h cm.skplanet.com mysql rman rman 
    # /usr/share/cmf/schema/scm_prepare_database.sh -h cm.skplanet.com mysql hue hue 

### [Step 6: Install CDH and Other Software]
    # systemctl start cloudera-scm-server
    # systemctl stop cloudera-scm-server
    # tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
    # vi /var/log/cloudera-scm-server/cloudera-scm-server.log
    172.31.15.117    cm.skplanet.com        cm
    172.31.0.89      master1.skplanet.com   master1
    172.31.0.71      worker1.skplanet.com   worker1
    172.31.8.189     worker2.skplanet.com   worker2
    172.31.12.75     worker3.skplanet.com   worker3

    cm.skplanet.com
    master1.skplanet.com
    worker1.skplanet.com
    worker2.skplanet.com
    worker3.skplanet.com

    [root@cm yum.repos.d]# vi cloudera-manager.repo 
    [cloudera-manager]
    #Packages for Cloudera Manager, Version 5, on RedHat or CentOS 7 x86_64
    name=Cloudera Manager
    baseurl=https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5/
    gpgkey =https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera
    gpgcheck = 1


    echo '
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCz+BzjVT79k1zGDLaZS9D0iNqHlMJV4gvVB+9a0s4QKWVQFEAEbyAFl8MasI+TtjJWk4ISRyw8iPit0/3OJA/wpOhCHQkh+eXIOZeyNJouiyLrQd3q8ArZhDX2lrnRBsfndIwVbBw0n0zlLOWDglX7W8cZnarnrGt5Jirpij17vhRD3sakyAs/7oYbtlMSAXvG1cjB+pozKWzM7vG15+uchqvkOtjMnVJx3qnNzvV93wyoo1s+JG4v60jvezWWaXRZoyxpMrAHE2+COHg2nZ5aQUbuGib5arAiwi4WyPSoVI5DaMZnwPcHPxv6RijsGoP20LVTb+XnsIKViKC/gkQT root@dichdptx-ambari-dev01.sktx.ss' >> /root/.ssh/authorized_keys



    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1zmB/NDVVD7gpg9p0cJIDx+sd1XPjGBhIptrTlM08i4lwXObR7zt/wOLVXdjvP5GE6cWf3OwyQzy8kL/T410YYkpeiolJAW7BVA00lZkI0yBGKVG/vW9/WkvQLg94Xtj93vHGkLnWgyIsjdxIlUeb8eFRxjhoExRvUIKZgINDtBZkqJT2vQJeXXPHcAMcN5aNkL2yp6icG2MPvnBiPqHc4sTNu6EkL3l9+x+S53qWaKrB9QF9/JVr5jsmdRfzXew5mAF5Cify0gktJEwg0jSImUjaH5SrKHqc+IpTxApttKRiRAFS1fEEhZR8FaqtOmP+URM4gnZNGQbVAdEc+KSl root@cm.skplanet.com


(end of file)

