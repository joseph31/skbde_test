# BDE_4_YelpDataAnalysis.md
    20190705-DataEngineering (Con't)
    Title: Yelp Restaurants Big Data Analysis Project Tutorial
        
        
## Yelp Restaurants Big Data Analysis Project Tutorial
 
 
### [Intro & Objectives]
    Dataset: 2017 Yelp challange data set 
        1 TAR file - 2.28 GB compressed
        6 JSON files - 5.79 GB uncompressed
        (The included files are: business, reviews, user, check-in, tip and photos)
        
    Objectives
        Download and extract the data set from Yelp
        Upload JSON files to HDFS using Ambari
        Install Rcongui SerDe
        Create tables and queries using HiveQL
        Export results and visualize them in Tableau


### [Step 1: Data Preparation]
    (TBD)
    
### [Step 2: Uploading Data to HDFS For Storage and Analysis]
    hdfs dfs -mkdir /user/training/yelp
    hdfs dfs -mkdir /user/training/yelp/business 
    hdfs dfs -mkdir /user/training/yelp/review 
    hdfs dfs -mkdir /user/training/yelp/users 
    hdfs dfs -mkdir /user/training/yelp/tip 
    hdfs dfs -mkdir /user/training/yelp/checkin 

    cd /home/training/dataset
    hdfs dfs -put business.json /user/training/yelp/business/
    hdfs dfs -put checkin.json /user/training/yelp/checkin/
    hdfs dfs -put review.json /user/training/yelp/review/
    hdfs dfs -put tip.json /user/training/yelp/tip/
    hdfs dfs -put user.json /user/training/yelp/users/
    #hdfs dfs -put photos.json /user/training/yelp/photos/
    
### [Step 3: Adding RCONGUI JSON SerDe]
    (this step may not be necessary with current hive version) 

    wget -O json-serde-1.3.8-jar-with-dependencies.jar  http://www.congiu.net/hive-json-serde/1.3.8/cdh5/json-serde-1.3.8-jar-with-dependencies.jar

    wget -O json-udf-1.3.8-jar-with-dependencies.jar http://www.congiu.net/hive-json-serde/1.3.8/cdh5/json-udf-1.3.8-jar-with-dependencies.jar

    ADD JAR json-serde-1.3.8-jar-with-dependencies.jar; 
    ADD JAR json-udf-1.3.8-jar-with-dependencies.jar; 

    set hive.cli.print.header=true;


### [Step 4: Creating Tables in HIVE]

    * 테이블 생성쿼리

        CREATE EXTERNAL TABLE business4 (
        address string,
        business_id string,
        categories array<string>,
        city string,
        hours struct<friday:string, monday:string, saturday:string, sunday:string, thursday:string,
        tuesday:string, wednesday:string>,
        is_open int,
        latitude double, 
        longitude double,
        name string,
        neighborhood string,
        postal_code string,
        review_count int,
        stars double,
        state string,
        Attributes struct<
        Accepts_Insurance:boolean,
        Ages_Allowed:string,
        Alcohol:string,
        Bike_Parking:boolean,
        Business_Accepts_Bitcoin:boolean,
        Business_Accepts_Credit_Cards:boolean,
        By_Appointment_Only:boolean,
        Byob:boolean,
        BYOB_Corkage:string,
        Caters:boolean,
        Coat_Check:boolean,
        Corkage:boolean,
        Dogs_Allowed:boolean, 
        Drive_Thru:boolean,
        Good_For_Dancing:boolean,
        Good_For_Kids:boolean,
        Happy_Hour:boolean,
        Has_TV:boolean,
        Noise_Level:string,
        Open24Hours:boolean,
        Outdoor_Seating:boolean,
        Restaurants_Attire:string,
        Restaurants_Counter_Service:boolean, 
        Restaurants_Delivery:boolean,
        Restaurants_Good_For_Groups:boolean,
        Restaurants_Reservations:boolean,
        Restaurants_Table_Service:boolean,
        Restaurants_Take_Out:boolean,
        Smoking:string,
        WheelchairAccessible:boolean, 
        WiFi:string,
        Ambience:struct<
        Casual:boolean,
        Classy:boolean,
        Divey:boolean,
        Hipster:boolean,
        Intimate:boolean,
        Romantic:boolean,
        Touristy:boolean,
        Trendy:boolean,
        Upscale:boolean>,
        BestNights:struct<
        Friday1:boolean,
        Monday1:boolean,
        Saturday1:boolean, 
        Sunday1:boolean,
        Thursday1:boolean,
        Tuesday1:boolean,
        Wednesday1:boolean>,
        BusinessParking:struct<
        Garage:boolean,
        Lot:boolean,
        Street:boolean,
        Valet:boolean,
        Validated:boolean>,
        DietaryRestrictions:struct<
        Dairy_Free:boolean,
        Gluten_Free:boolean,
        Halal:boolean,
        Kosher:boolean,
        Soy_Free:boolean,
        Vegan:boolean,
        Vegetarian:boolean>,
        GoodForMeal:struct<
        Breakfast:boolean,
        Brunch:boolean,
        Dessert:boolean, 
        Dinner:boolean,
        Latenight:boolean,
        Lunch:boolean>,
        HairSpecializesIn:struct<
        Africanamerican:boolean,
        Asian:boolean,
        Coloring:boolean,
        Curly:boolean,
        Extensions:boolean,
        Kids:boolean,
        Perms:boolean,
        Straightperms:boolean>,
        Music:struct<
        BackgroundMusic:boolean,
        Dj:boolean,
        Jukebox:boolean,
        Karaoke:boolean,
        Live:boolean, 
        NoMusic:boolean,
        Video:boolean>,
        restaurantspricerange2:int>)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/business'; 


        CREATE TABLE restaurants
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/business/restaurants'
        AS
        SELECT * FROM exploded WHERE cat_exploded="Restaurants"; 

        CREATE EXTERNAL TABLE review (
        business_id string,
        cool int,
        review_date string,
        funny int,
        review_id string,
        stars int,
        text string,
        useful int,
        user_id string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/review'; 

        CREATE TABLE review_filtered
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE 
        LOCATION '/user/training/yelp/review_filtered'
        AS
        SELECT re.business_id, r.stars, r.user_id
        FROM review r JOIN restaurants re
        ON r.business_id = re.business_id;

        CREATE EXTERNAL TABLE users (
        average_stars double,
        compliment_cool int,
        compliment_cute int,
        compliment_funny int,
        compliment_hot int,
        compliment_list int,
        compliment_more int,
        compliment_note int,
        compliment_photos int,
        compliment_plain int,
        compliment_profile int,
        compliment_writer int,
        cool int,
        elite array<int>,
        fans int,
        friends array<string>,
        funny int,
        name string,
        review_count int,
        useful int,
        user_id string,
        yelping_since string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        LOCATION '/user/training/yelp/users'; 

        CREATE TABLE elite_users
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/users/elite'
        AS
        SELECT * FROM users LATERAL VIEW explode(elite) c AS elite_year; 

        CREATE EXTERNAL TABLE tip (
        text string,
        date_tip string,
        likes int,
        business_id string,
        user_id string)
        ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
        STORED AS TEXTFILE
        LOCATION '/user/training/yelp/tip'; 

(end of file) 
