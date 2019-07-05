# BDE_2_InstallSqoopSparkImpalaKafka.md
    20190704-DataEngineering (Con't) 


## Install Sqoop, Spark, Impala and Kafka

### [scp SQL files]
    authors-23-04-2019-02-34-beta.sql.zip
    posts23-04-2019-02-44.sql.zip

    # scp  -i ./SKT.pem ./authors-23-04-2019-02-34-beta.sql.zip centos@13.209.93.133:/home/centos
    # scp  -i ./SKT.pem ./posts23-04-2019-02-44.sql.zip centos@13.209.93.133:/home/centos
    [root@cm centos]# ls -al *.sql
    -rwxrwxrwx 1 root root   892780 Apr 23 02:34 authors-23-04-2019-02-34-beta.sql
    -rwxrwxrwx 1 root root 52680682 Apr 23 02:44 posts23-04-2019-02-44.sql
    # mysql -u root -p
    CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON test.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';
    # mysql -u training -p
    # source authors-23-04-2019-02-34-beta.sql
    # source posts23-04-2019-02-44.sql

    
### [Install Sqoop]
    sqoop import --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --table authors  --driver com.mysql.jdbc.Driver --target-dir /user/training/authors --hive-import --hive-table test.authors 
    sqoop import --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --table posts  --driver com.mysql.jdbc.Driver --target-dir /user/training/posts --hive-import --hive-table test.posts

    CREATE External TABLE `authors`(
      `id` int,
      `first_name` string,
      `last_name` string,
      `email` string,
      `birthdate` string,
      `added` string)
    COMMENT 'Imported by sqoop on 2019/07/04 05:42:59'
    ROW FORMAT SERDE
      'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
    WITH SERDEPROPERTIES (
      'field.delim'='\u0001',
      'line.delim'='\n',
      'serialization.format'='\u0001')
    STORED AS INPUTFORMAT
      'org.apache.hadoop.mapred.TextInputFormat'
    OUTPUTFORMAT
      'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
    LOCATION
      'hdfs://master1.skplanet.com:8020/user/training/authors'
    TBLPROPERTIES (
      'COLUMN_STATS_ACCURATE'='true',
      'numFiles'='4',
      'totalSize'='760827',
      'transient_lastDdlTime'='1562218982')

### [Install Impala] 
    1) shell실행
     명령어 : impala-shell 
    2) 접속후 하이브 테이블 메타정보 갱신
      INVALIDATE METADATA 
    3) 테이블 데이터 확인
     show databases;
     use test;
     select * from authors limit 10;
     
### [Export Sqoop]
 
#### hive query to hdfs 
        INSERT OVERWRITE DIRECTORY '/user/training/results'
        select a.id, a.fname, a.Lname, count(a.pid)
        from (
            select authors.id as id, authors.first_name as fname, authors.last_name as Lname, posts.id as pid
          from authors, posts
          where authors.id = posts.author_id
        ) a
        group by a.id, a.fname, a.Lname;

##### create mysql table (test.results)
        create table test.results (
          id varchar(100)
          , fname varchar(100)
          , Lname varchar(100)
          , num_posts int
        );

#### hdfs to mysql (sqoop)
        sqoop export --connect jdbc:mysql://cm:3306/test --username training --password Hadoop123! --driver com.mysql.jdbc.Driver --table test.results --export-dir /user/training/results --input-fields-terminated-by '\0001'

    
### [Install Spark2]
    link: https://www.cloudera.com/documentation/spark2/latest/topics/spark2.html
    CDS Powered by Apache Spark Overview
    CDS(custom service descriptor)

    1. To download the CDS Powered by Apache Spark service descriptor, in the Version Information table in CDS Versions Available for Download, click the service descriptor link for the version you want to install.

    Available CDS Versions
    Version	        Custom Service Descriptor	        Parcel Repository
    2.4 Release 2	SPARK2_ON_YARN-2.4.0.cloudera2.jar	http://archive.cloudera.com/spark2/parcels/2.4.0.cloudera2/

    2. Log on to the Cloudera Manager Server host, and copy the CDS Powered by Apache Spark service descriptor in the location configured for service descriptor files.
    * scp  -i ./SKT.pem ./SPARK2_ON_YARN-2.4.0.cloudera2.jar centos@13.209.93.133:/home/centos

    3. Set the file ownership of the service descriptor to cloudera-scm:cloudera-scm with permission 644.

    [root@cm csd]# pwd
    /opt/cloudera/csd
    [root@cm csd]# cp /home/centos/SPARK2_ON_YARN-2.4.0.cloudera2.jar ./
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 root root 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# chown cloudera-scm:cloudera-scm SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# 
    [root@cm csd]# chmod 644 SPARK2_ON_YARN-2.4.0.cloudera2.jar 
    [root@cm csd]# ll
    total 20
    -rw-r--r-- 1 cloudera-scm cloudera-scm 19066 Jul  4 00:49 SPARK2_ON_YARN-2.4.0.cloudera2.jar
    [root@cm csd]# 

    4. Restart the Cloudera Manager Server with the following command:
    command: systemctl restart cloudera-scm-server

    5. Download the CDS Powered by Apache Spark parcel, distribute the parcel to the hosts in your cluster, and activate the parcel. See Managing Parcels.
    cm 메뉴 옆에 있는 Parcel menu 에서 spark2 에 대한 설정 진행 

### [Create user “training” in linux and in hdfs]
    # useradd training -G hadoop 
    # passwd training 
    # hdfs dfs -mkdir /user/training
    # hdfs dfs -chown training:hadoop /user/training
    
### []
    # mysql -u root -p
    CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';

    # mysql -u training -p
    source authors-23-04-2019-02-34-beta.sql
    source posts23-04-2019-02-44.sql

### [Extract tables authors and posts from the database and create Hive tables]
    a. Use Sqoop to import the data from authors and posts 

    check 
    # beeline -u jdbc:hive2://cm.skplanet.com:10000 -n hive

    authors, posts
    # sqoop import --connect jdbc:mysql://cm.skplanet.com:3306/test \
    --username training  \
    -P training   \
    --table authors     \
    --target-dir /user/training   \
    --fields-terminated-by ","    \
    --hive-import    \
    --create-hive-table    \
    --hive-table authors

    b. For both tables, you will import the data in tab delimited text format 
    c. The imported data should be saved in training’s HDFS home directory 
        i. Create authors and posts directories in your HDFS home directory 
        ii. Save the imported data in each 
    d. In Hive, create 2 tables: authors and posts. They will contain the data that 
    you imported from Sqoop in above step. 
    e. You are free to use whatever database in Hive. 
    f. Create authors as an external table. 
    g. Create posts as a managed table. 

### [Install Kafka]
    CM에서 인스톨
    
## Let's test out our cluster 
    * (TBD)
  
## Setup your cluster to begin data analysis 
    * (TBD)


