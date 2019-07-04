## System pre-configuration checks
  0. connect to each server (public IP)
      * (by terminal) (e.g. #ssh -i /path/to/keyname.pem centos@13.209.93.133)   
      * (by secureCRT)
  1. Update yum 
      * #yum update 
      
      
      
      
20190704-DataEngineering


### [sql file scp]
* files
    * authors-23-04-2019-02-34-beta.sql.zip
    * posts23-04-2019-02-44.sql.zip
* commands
    * scp  -i ./SKT.pem ./authors-23-04-2019-02-34-beta.sql.zip centos@13.209.93.133:/home/centos
    * scp  -i ./SKT.pem ./posts23-04-2019-02-44.sql.zip centos@13.209.93.133:/home/centos
    * [root@cm centos]# ls -al *.sql
        * -rwxrwxrwx 1 root root   892780 Apr 23 02:34 authors-23-04-2019-02-34-beta.sql
        * -rwxrwxrwx 1 root root 52680682 Apr 23 02:44 posts23-04-2019-02-44.sql
    * mysql -u root -p
        * CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
        * GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';
    * mysql -u training -p
    * source authors-23-04-2019-02-34-beta.sql
    * source posts23-04-2019-02-44.sql

### [Install spark2]
* link: https://www.cloudera.com/documentation/spark2/latest/topics/spark2.html
    * CDS Powered by Apache Spark Overview
    * CDS(custom service descriptor)

1. To download the CDS Powered by Apache Spark service descriptor, in the Version Information table in CDS Versions Available for Download, click the service descriptor link for the version you want to install.

Available CDS Versions
Version	        Custom Service Descriptor	        Parcel Repository
2.4 Release 2	SPARK2_ON_YARN-2.4.0.cloudera2.jar	http://archive.cloudera.com/spark2/parcels/2.4.0.cloudera2/

2. Log on to the Cloudera Manager Server host, and copy the CDS Powered by Apache Spark service descriptor in the location configured for service descriptor files.
    * scp  -i ./SKT.pem ./SPARK2_ON_YARN-2.4.0.cloudera2.jar centos@13.209.93.133:/home/centos

3. Set the file ownership of the service descriptor to cloudera-scm:cloudera-scm with permission 644.

    * [root@cm csd]# pwd
        * /opt/cloudera/csd
    * [root@cm csd]# cp /home/centos/SPARK2_ON_YARN-2.4.0.cloudera2.jar ./
    * [root@cm csd]# ll
        * total 20
        * -rw-r--r-- 1 root root 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    * [root@cm csd]# chown cloudera-scm:cloudera-scm SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    * [root@cm csd]# ll
        * total 20
        * -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    * [root@cm csd]# 
    * [root@cm csd]# chmod 644 SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    * [root@cm csd]# ll
        * total 20
        * -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    * [root@cm csd]# 

4. Restart the Cloudera Manager Server with the following command:
  * command: systemctl restart cloudera-scm-server

5. Download the CDS Powered by Apache Spark parcel, distribute the parcel to the hosts in your cluster, and activate the parcel. See Managing Parcels.
   * cm 메뉴 옆에 있는 Parcel menu 에서 spark2 에 대한 설정 진행 


## Let’s test out our cluster 

1. Create user “training” in linux and in hdfs. 
    * useradd training -G hadoop 
    * passwd training 
    * hdfs dfs -mkdir /user/training
    * hdfs dfs -chown training:hadoop /user/training
    
2. (추가바람) 
    * mysql -u root -p
        * CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
        * GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';

    * mysql -u training -p
        * source authors-23-04-2019-02-34-beta.sql
        * source posts23-04-2019-02-44.sql

3. Extract tables authors and posts from the database and create Hive tables. 

    * a. Use Sqoop to import the data from authors and posts 
        * beeline -u jdbc:hive2://cm.skplanet.com:10000 -n hive
            * authors, posts
        * sqoop import --connect jdbc:mysql://cm.skplanet.com:3306/test \
          --username training  \
          -P training   \
          --table authors     \
          --target-dir /user/training   \
          --fields-terminated-by ","    \
          --hive-import    \
          --create-hive-table    \
          --hive-table authors

    * b. For both tables, you will import the data in tab delimited text format 
    * c. The imported data should be saved in training’s HDFS home directory 
        * i. Create authors and posts directories in your HDFS home directory 
        * ii. Save the imported data in each 
    * d. In Hive, create 2 tables: authors and posts. They will contain the data that you imported from Sqoop in above step. 
    * e. You are free to use whatever database in Hive. 
    * f. Create authors as an external table. 
    * g. Create posts as a managed table. 
