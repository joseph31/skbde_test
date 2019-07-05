# README.md

## 개요 
    * 과정명: SK ICT Big Data Engineering 통합실습  
    * 일시: 20190703-0705
    * 강사: Helry Park (hwpark at wiken.co.kr) 
    * Group 4: 정명훈(리더, SK planet), 전평재(SK planet), 양진욱(SK planet)
    * 파일 구성
        BDE_1_InstallCM.md
        BDE_2_InstallSqoopSparkImpalaKafka.md
        BDE_3_TestCluster.md
        BDE_4_YelpDataAnalysis.md
        yelp_create_table_query
        
    
## 목차 및 내용  
    * BDE_1_InstallCM.md
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

    * BDE_2_InstallSqoopSparkImpalaKafka.md
        ## Install Sqoop, Spark, and Kafka (and else)
            [Install Spark2]
            [Install Sqoop, Impala, Kafka]

    * BDE_3_TestCluster.md
        ## Let’s test out our cluster 
            [0. scp SQL files]
            [1. Create user “training” in linux and in hdfs]
            [2. In MySQL create the sample tables that will be used for the rest of the test]
            [3. Extract tables authors and posts from the database and create Hive tables]
            [4. Create and run a Hive/Impala query]
            [5. Export the data from above query to MySQL]

    * BDE_4_YelpDataAnalysis.md
         [Intro & Objectives]
         [Step 1: Data Preparation]
         [Step 2: Uploading Data to HDFS For Storage and Analysis]
         [Step 3: Adding RCONGUI JSON SerDe]
         [Step 4: Creating Tables in HIVE]
         [Analysis]
            1. bussiness json파일을 hive schema 없이 테이블로 넣기
               1) spark >> json 스트링에서 parquet에서 사용불가한 문자인 '-'를 '_'으로 변환하여 pqrquet 파일로 쓰기
               2) hdfs 에서 file 경로 복사
               3) impala에서 테이블 처리 : impala에서는 parquet 파일을 기반으로 impala table을 생성할 수 있음
            2. Yelp data 분석하기
               1) 공통적으로 사용할 view 생성
               2) Which cities have the highest number of restaurants?
               3) Top 15 sub-categories of restaurants
               4) Distribution of ratings vs. categories
               5) What ratings do the majority of restaurants have?
               6) Rating distribution in restaurant reviews
            
    * yelp_create_table_query
        ## 테이블 생성쿼리
        
        
 (end of file) 
