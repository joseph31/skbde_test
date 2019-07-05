# 20190704-DataEngineering (Con't) 


## Let’s test out our cluster 
    
### [1. Create user “training” in linux and in hdfs]

    useradd training -G hadoop 
    passwd training 
    hdfs dfs -mkdir /user/training
    hdfs dfs -chown training:hadoop /user/training

### [2. In MySQL create the sample tables that will be used for the rest of the test]
    mysql -u root -p
    CREATE DATABASE test DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
    GRANT ALL ON *.* TO 'training'@'%' IDENTIFIED BY 'Admin123!';

    mysql -u training -p
    source authors-23-04-2019-02-34-beta.sql
    source posts23-04-2019-02-44.sql

### [3. Extract tables authors and posts from the database and create Hive tables]

#### a. Use Sqoop to import the data from authors and posts 

        #check 
        beeline -u jdbc:hive2://cm.skplanet.com:10000 -n hive

        authors, posts

        sqoop import --connect jdbc:mysql://cm.skplanet.com:3306/test \
        --username training  \
        -P training   \
        --table authors     \
        --target-dir /user/training   \
        --fields-terminated-by ","    \
        --hive-import    \
        --create-hive-table    \
        --hive-table authors

#### b. For both tables, you will import the data in tab delimited text format 

#### c. The imported data should be saved in training’s HDFS home directory 
        i. Create authors and posts directories in your HDFS home directory 
        ii. Save the imported data in each 

#### d. In Hive, create 2 tables: authors and posts. 
They will contain the data that you imported from Sqoop in above step. 

#### e. You are free to use whatever database in Hive. 

#### f. Create authors as an external table. 

#### g. Create posts as a managed table.
