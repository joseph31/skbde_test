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
      * (by terminal)  
      * (by secureCRT)
  1. Update yum 
      * #yum update 
  2. Change the run-level to multi-user text mode
      * #systemctl get-default
      * #systemctl set-default multi-user.target 
  3. Disable SE linux
      * #sestatus 
  4. Disable firewall
      * 방화벽 중지: #systemctl stop firewalld
      * 방화벽 자동시작 해제 (재부팅시 켜지지 않음): #systemctl disable firewalld
  5. Check vm.swappiness & update permanently as necessary (= set value = 1)
      * #sysctl vm.swappiness
      * #vi /etc/sysctl.conf --> vm.swappiness = 1
      * sysctl -w vm.swappiness=1
  6. Disable transparent hugepage support permanently
  7. Check to see that nscd service is running
  8. Check to see that ntp service is running
  9. Disable IPv6
  10. During the installation process, Cloudera Manager Server will need to remotely access each of the remaining nodes. In order to facilitate this, you may either set up an admin user and password to be used by Cloudera Manager Server or setup a private/public key access. Whichever method you choose, make sure you test access with ssh before proceeding.
  11. Show that forward and reverse host lookups are correctly resolved
  12. Change the hostname of each of the nodes to match the FQDN that you entered in the /etc/hosts file.
    

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



