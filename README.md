# skbde_test
## Basic Info
* Big Data Engineering Hands-on
* group 4 
* Installing CDH using CM - hands-on from scratch 

## (title)
  1. (title)
      * desc) 
        - dd
      * command) # cmd 
  
## System pre-configuration checks
  0. connect to each server (public IP)
      * (by terminal) (e.g. #ssh -i /path/to/keyname.pem@13.209.93.133)   
      * (by secureCRT)
  1. Update yum 
      * #yum update 
  2. Change the run-level to multi-user text mode
      * #systemctl get-default
      * #systemctl set-default multi-user.target 
  3. Disable SE linux
      * #sestatus 
      * (추가설명) 
  4. Disable firewall
      * (본 실습에서는 이미 firewall이 disable되어 있음) 
      * 방화벽 중지: #systemctl stop firewalld
      * 방화벽 자동시작 해제 (재부팅시 켜지지 않음): #systemctl disable firewalld
  5. Check vm.swappiness & update permanently as necessary (= set value = 1)
      * #sysctl vm.swappiness
      * #vi /etc/sysctl.conf --> vm.swappiness = 1
        * sysctl -w vm.swappiness=1
  6. Disable transparent hugepage support permanently
      * (too long) 
  7. Check to see that nscd service is running
      * nscd: Name Service Cache Daemon 
      * #yum install nscd
      * #systemctl enable nscd
      * #systemctl start nscd
      * #systemctl status nscd
  8. Check to see that ntp service is running - disable chrony as necessary 
      * ntp: The Network Time Protocol 
      * #yum install ntp
      * #vi /etc/ntp.conf
      * #systemctl start ntpd
      * #systemctl status ntpd
      * #systemctl enable ntpd
      * #systemctl status chronyd  : inactive 상태임
  9. Disable IPv6
      * ref: https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_IPv6_%EB%B9%84%ED%99%9C%EC%84%B1%ED%99%94
      * [root@ip-172-31-15-117 ~]# sysctl net.ipv6.conf.all.disable_ipv6
      * net.ipv6.conf.all.disable_ipv6 = 0            → disable_ipv6 이 0이므로 활성화 상태임
      * #sysctl -w net.ipv6.conf.all.disable_ipv6=1   → disable_ipv6 이 1이므로 비활성화됨
      * #vi /etc/sysctl.conf
        * net.ipv6.conf.all.disable_ipv6 = 1
        * net.ipv6.conf.default.disable_ipv6 = 1
      * #sysctl -p
  10. During the installation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. 
  In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server or setup a private/public key access. Whichever method you choose, make sure you test access with ssh before proceeding.
      * (별도 설명?) 
  11. Show that forward and reverse host lookups are correctly resolved
o In this lab, we will use /etc/hosts Files setting to accomplish this 
o Add the necessary information to the /etc/hosts files 
o Check to make sure that File lookup has priority 
o Use getent to make sure you are getting proper host name and ip address 
  12. Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file.
      * vi /etc/hosts
172.31.15.117    cm.skplanet.com        cm
172.31.0.89      master1.skplanet.com   master1
172.31.0.71      worker1.skplanet.com   worker1
172.31.8.189     worker2.skplanet.com   worker2
172.31.12.75     worker3.skplanet.com   worker3

hostnamectl set-hostname worker3.skplanet.com

vi /etc/sysconfig/network 
HOSTNAME=worker3.skplanet.com

## Path B install using CM 5.15x
* desc
  * desc

## Install & configure MariaDB(MySQL)
* desc
  * desc

## MySQL installation - plan two detail
* desc
  * desc

## Install a cluster & deploy CDH
* desc
  * desc

## Install Sqoop, Spark, and Kafka
* desc
  * desc

## Test cluster 
* desc
  * desc
  
## Setup cluster to begin data analysis 
* desc
  * desc

## cf. Table of Conents
* System pre-configuration checks
* Path B install using CM 5.15x
* Install & configure MariaDB(MySQL)
* MySQL installation - plan two detail
* Install a cluster & deploy CDH
* Install Sqoop, Spark, and Kafka
* Test cluster 
* Setup cluster to begin data analysis 



