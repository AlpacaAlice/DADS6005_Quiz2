# DADS6005_Quiz2
KsqlDB Quiz

# Member
1. Nithida Phookriangkrai &nbsp; ID: 6420422004
2. Napasakon Monbut &nbsp;&emsp;&ensp; ID: 6420422009
3. Thunpitcha Sattabun &nbsp;&ensp;&nbsp; ID: 6420422010
4. Wisarut Sarai &nbsp;&emsp;&emsp;&emsp;&emsp;&nbsp; ID: 6420422012
5. Natchapat Youngchoay&nbsp; ID: 6420422013

# Video
- Watch full flow as youtube link below


[![DADS6005_Quiz2](http://img.youtube.com/vi/3-rtg6LyY80/0.jpg)](https://www.youtube.com/watch?v=3-rtg6LyY80 "DADS6005 Quiz2")


# Objective

- Design & implement “data-streaming and real-time analytics” system
- Real-time data cleansing and analytics


# Dataset

- Food choices College students' food and cooking preferences ([Reference](https://www.kaggle.com/datasets/borapajo/food-choices?select=food_coded.csv))

# Diagram
<img src="images/Diagram.png"/>


# Procedure
- Please see the [Reference](https://docs.ksqldb.io/en/latest/tutorials/etl/) follow the link
1. Run docker
    ```batch
    docker-compose up -d
    ```
    
2. After docker build complete, run file <create_table.py> to create table inside Postgres
    ```python
    python create_table.py
    ```
    - You can check Postgres with command
        ```batch
        docker exec -it postgres /bin/bash
        ```
        ```batch
        psql -U postgres-user customers
        ```

3. Run file <import_foodcoded.py> to import data from <food_coded.csv>
    ```python
    python import_foodcoded.py
    ```

4. Start Ksql to create connector
    ```batch
    docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
    ```
    - Enter following queries to create a source connector
      ```sql
      SET 'auto.offset.reset' = 'earliest';
      ```
      ```sql
      CREATE SOURCE CONNECTOR customers_reader WITH (
          'connector.class' = 'io.debezium.connector.postgresql.PostgresConnector',
          'database.hostname' = 'postgres',
          'database.port' = '5432',
          'database.user' = 'postgres-user',
          'database.password' = 'postgres-pw',
          'database.dbname' = 'customers',
          'database.server.name' = 'localhost',
          'table.whitelist' = 'public.foodcoded',
          'transforms' = 'unwrap',
          'transforms.unwrap.type' = 'io.debezium.transforms.ExtractNewRecordState',
          'transforms.unwrap.drop.tombstones' = 'false',
          'transforms.unwrap.delete.handling.mode' = 'rewrite'
      );
      ```
    - Create a Stream topic1 for recieve data
      ```sql
      CREATE STREAM foodcoded WITH (
          kafka_topic = 'localhost.public.foodcoded',
          value_format = 'avro'
      );
      ```
    - Create a Stream topic2 for clean data follow <foodcoded_clean.sql>
      
      
    - Create a Stream topic3 for analyze data follow <foodcoded_analyze.sql>
    
    
    - Enter following queries to create a sink connector
      ```sql
      CREATE SINK CONNECTOR data_writer WITH (
          'connector.class' = 'io.confluent.connect.elasticsearch.ElasticsearchSinkConnector',
          'connection.url' = 'http://elastic:9200',
          'type.name' = 'kafka-connect',
          'topics' = 'foodcoded_analyze',
          'key.ignore' = 'true',
          'name' = 'data_writer',
          'tasks.max' = '1'
      );
      ```
      
5. Analyzed data from Elasticsearch can be check via command
    ```batch
    curl -XGET 'http://localhost:9200/foodcoded_analyze/_search?pretty'
    ```
    
6. Resul will be as below
    ```JSON
      {
        "took" : 816,
        "timed_out" : false,
        "_shards" : {
          "total" : 1,
          "successful" : 1,
          "skipped" : 0,
          "failed" : 0
        },
        "hits" : {
          "total" : {
            "value" : 123,
            "relation" : "eq"
          },
          "max_score" : 1.0,
          "hits" : [
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+3",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 2.4,
                "GRADE" : "C",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : null,
                "CALORIES_DAY_DESC" : null,
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 2,
                "COOK_DESC" : "A couple of times a week",
                "CUISINE" : null,
                "CUISINE_DESC" : null,
                "DIET_CURRENT_CODED" : 1,
                "DIET_CURRENT_CODED_DESC" : "healthy/balanced/moderated/",
                "EATING_OUT" : 3,
                "EATING_OUT_DESC" : "2-3 times",
                "EMPLOYMENT" : 3,
                "EMPLOYMENT_DESC" : "no",
                "ETHNIC_FOOD" : 1,
                "ETHNIC_FOOD_DESC" : "very unlikely",
                "EXERCISE" : 1,
                "EXERCISE_DESC" : "Everyday",
                "FAV_CUISINE" : "Arabic cuisine",
                "FAV_CUISINE_CODED" : 3,
                "FAV_CUISINE_CODED_DESC" : "Arabic/Turkish",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "rice  and chicken ",
                "FOOD_CHILDHOOD_SPLIT" : "rice  and chicken ",
                "FRUIT_DAY" : 5,
                "FRUIT_DAY_DESC" : "very likely",
                "GREEK_FOOD" : 5,
                "GREEK_FOOD_DESC" : "very likely",
                "HEALTHY_FEELING" : 2,
                "INCOME" : 5,
                "INCOME_DESC" : "$70,001 to $100,000",
                "INDIAN_FOOD" : 5,
                "INDIAN_FOOD_DESC" : "very likely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 5,
                "NUTRITIONAL_CHECK_DESC" : "on everything",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 2,
                "PAY_MEAL_OUT_DESC" : "$5.01 to $10.00",
                "PERSIAN_FOOD" : 5,
                "PERSIAN_FOOD_DESC" : "very likely",
                "SELF_PERCEPTION_WEIGHT" : 3,
                "SELF_PERCEPTION_WEIGHT_DESC" : "just right",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 1,
                "THAI_FOOD_DESC" : "very unlikely",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 187
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+18",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.3,
                "GRADE" : "B",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 5,
                "COOK_DESC" : "Never, I really do not know my way around a kitchen",
                "CUISINE" : 1,
                "CUISINE_DESC" : "American",
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 4,
                "EATING_OUT_DESC" : "3-5 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 4,
                "ETHNIC_FOOD_DESC" : "likely",
                "EXERCISE" : 1,
                "EXERCISE_DESC" : "Everyday",
                "FAV_CUISINE" : "Mexican",
                "FAV_CUISINE_CODED" : 2,
                "FAV_CUISINE_CODED_DESC" : "Spanish/mexican",
                "FAV_FOOD" : 3,
                "FAV_FOOD_DESC" : "both bought at store and cooked at home",
                "FOOD_CHILDHOOD" : "pizza, chicken fingers",
                "FOOD_CHILDHOOD_SPLIT" : "pizza",
                "FRUIT_DAY" : 2,
                "FRUIT_DAY_DESC" : "unlikely",
                "GREEK_FOOD" : 2,
                "GREEK_FOOD_DESC" : "unlikely",
                "HEALTHY_FEELING" : 5,
                "INCOME" : 6,
                "INCOME_DESC" : "higher than $100,000",
                "INDIAN_FOOD" : 1,
                "INDIAN_FOOD_DESC" : "very unlikely",
                "ITALIAN_FOOD" : 4,
                "ITALIAN_FOOD_DESC" : "likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 2,
                "NUTRITIONAL_CHECK_DESC" : "on certain products only",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 2,
                "PAY_MEAL_OUT_DESC" : "$5.01 to $10.00",
                "PERSIAN_FOOD" : 1,
                "PERSIAN_FOOD_DESC" : "very unlikely",
                "SELF_PERCEPTION_WEIGHT" : 6,
                "SELF_PERCEPTION_WEIGHT_DESC" : "i dont think myself in these terms",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 1,
                "THAI_FOOD_DESC" : "very unlikely",
                "VEGGIES_DAY" : 3,
                "VEGGIES_DAY_DESC" : "neutral",
                "VITAMINS" : 2,
                "VITAMINS_DESC" : "No",
                "WEIGHT" : 175
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+24",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.7,
                "GRADE" : "B+",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 2,
                "CALORIES_DAY_DESC" : "it is not at all important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 3,
                "COOK_DESC" : "Whenever I can, but that is not very often",
                "CUISINE" : 1,
                "CUISINE_DESC" : "American",
                "DIET_CURRENT_CODED" : 1,
                "DIET_CURRENT_CODED_DESC" : "healthy/balanced/moderated/",
                "EATING_OUT" : 2,
                "EATING_OUT_DESC" : "1-2 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 2,
                "ETHNIC_FOOD_DESC" : "unlikely",
                "EXERCISE" : 1,
                "EXERCISE_DESC" : "Everyday",
                "FAV_CUISINE" : "Italian food ",
                "FAV_CUISINE_CODED" : 1,
                "FAV_CUISINE_CODED_DESC" : "Italian/French/greek",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "Chicken Parm, Pizza ",
                "FOOD_CHILDHOOD_SPLIT" : "Chicken Parm",
                "FRUIT_DAY" : 3,
                "FRUIT_DAY_DESC" : "neutral",
                "GREEK_FOOD" : 1,
                "GREEK_FOOD_DESC" : "very unlikely",
                "HEALTHY_FEELING" : 9,
                "INCOME" : 5,
                "INCOME_DESC" : "$70,001 to $100,000",
                "INDIAN_FOOD" : 1,
                "INDIAN_FOOD_DESC" : "very unlikely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 2,
                "MARITAL_STATUS_DESC" : "In a relationship",
                "NUTRITIONAL_CHECK" : 2,
                "NUTRITIONAL_CHECK_DESC" : "on certain products only",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 4,
                "PAY_MEAL_OUT_DESC" : "$20.01 to $30.00",
                "PERSIAN_FOOD" : 1,
                "PERSIAN_FOOD_DESC" : "very unlikely",
                "SELF_PERCEPTION_WEIGHT" : 2,
                "SELF_PERCEPTION_WEIGHT_DESC" : "very fit",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 2,
                "THAI_FOOD_DESC" : "unlikely",
                "VEGGIES_DAY" : 3,
                "VEGGIES_DAY_DESC" : "neutral",
                "VITAMINS" : 2,
                "VITAMINS_DESC" : "No",
                "WEIGHT" : 160
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+25",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.0,
                "GRADE" : "B",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 4,
                "CALORIES_DAY_DESC" : "it is very important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 4,
                "COOK_DESC" : "I only help a little during holidays",
                "CUISINE" : 1,
                "CUISINE_DESC" : "American",
                "DIET_CURRENT_CODED" : 1,
                "DIET_CURRENT_CODED_DESC" : "healthy/balanced/moderated/",
                "EATING_OUT" : 2,
                "EATING_OUT_DESC" : "1-2 times",
                "EMPLOYMENT" : 3,
                "EMPLOYMENT_DESC" : "no",
                "ETHNIC_FOOD" : 3,
                "ETHNIC_FOOD_DESC" : "neutral",
                "EXERCISE" : 1,
                "EXERCISE_DESC" : "Everyday",
                "FAV_CUISINE" : "Mexican ",
                "FAV_CUISINE_CODED" : 2,
                "FAV_CUISINE_CODED_DESC" : "Spanish/mexican",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "Steak",
                "FOOD_CHILDHOOD_SPLIT" : "Steak",
                "FRUIT_DAY" : 5,
                "FRUIT_DAY_DESC" : "very likely",
                "GREEK_FOOD" : 3,
                "GREEK_FOOD_DESC" : "neutral",
                "HEALTHY_FEELING" : 9,
                "INCOME" : 6,
                "INCOME_DESC" : "higher than $100,000",
                "INDIAN_FOOD" : 3,
                "INDIAN_FOOD_DESC" : "neutral",
                "ITALIAN_FOOD" : 4,
                "ITALIAN_FOOD_DESC" : "likely",
                "MARITAL_STATUS" : 2,
                "MARITAL_STATUS_DESC" : "In a relationship",
                "NUTRITIONAL_CHECK" : 4,
                "NUTRITIONAL_CHECK_DESC" : "on most products",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 6,
                "PAY_MEAL_OUT_DESC" : "more than $40.01",
                "PERSIAN_FOOD" : 2,
                "PERSIAN_FOOD_DESC" : "unlikely",
                "SELF_PERCEPTION_WEIGHT" : 2,
                "SELF_PERCEPTION_WEIGHT_DESC" : "very fit",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 3,
                "THAI_FOOD_DESC" : "neutral",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 175
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+28",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 4.0,
                "GRADE" : "A",
                "GENDER" : 1,
                "GENDER_DESC" : "Female",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 3,
                "COOK_DESC" : "Whenever I can, but that is not very often",
                "CUISINE" : 1,
                "CUISINE_DESC" : "American",
                "DIET_CURRENT_CODED" : 1,
                "DIET_CURRENT_CODED_DESC" : "healthy/balanced/moderated/",
                "EATING_OUT" : 3,
                "EATING_OUT_DESC" : "2-3 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 4,
                "ETHNIC_FOOD_DESC" : "likely",
                "EXERCISE" : 2,
                "EXERCISE_DESC" : "Twice or three times per week",
                "FAV_CUISINE" : "mexican",
                "FAV_CUISINE_CODED" : 2,
                "FAV_CUISINE_CODED_DESC" : "Spanish/mexican",
                "FAV_FOOD" : 2,
                "FAV_FOOD_DESC" : "store bought",
                "FOOD_CHILDHOOD" : "french fries, waffles, chocolate",
                "FOOD_CHILDHOOD_SPLIT" : "french fries",
                "FRUIT_DAY" : 3,
                "FRUIT_DAY_DESC" : "neutral",
                "GREEK_FOOD" : 2,
                "GREEK_FOOD_DESC" : "unlikely",
                "HEALTHY_FEELING" : 7,
                "INCOME" : 5,
                "INCOME_DESC" : "$70,001 to $100,000",
                "INDIAN_FOOD" : 3,
                "INDIAN_FOOD_DESC" : "neutral",
                "ITALIAN_FOOD" : 4,
                "ITALIAN_FOOD_DESC" : "likely",
                "MARITAL_STATUS" : 2,
                "MARITAL_STATUS_DESC" : "In a relationship",
                "NUTRITIONAL_CHECK" : 4,
                "NUTRITIONAL_CHECK_DESC" : "on most products",
                "ON_OFF_CAMPUS" : 3,
                "ON_OFF_CAMPUS_DESC" : "Live with my parents and commute",
                "PARENTS_COOK" : 3,
                "PARENTS_COOK_DESC" : "1-2 times a week",
                "PAY_MEAL_OUT" : 3,
                "PAY_MEAL_OUT_DESC" : "$10.01 to $20.00",
                "PERSIAN_FOOD" : 2,
                "PERSIAN_FOOD_DESC" : "unlikely",
                "SELF_PERCEPTION_WEIGHT" : 3,
                "SELF_PERCEPTION_WEIGHT_DESC" : "just right",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 3,
                "THAI_FOOD_DESC" : "neutral",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 115
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+26",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.2,
                "GRADE" : "B",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 2,
                "CALORIES_DAY_DESC" : "it is not at all important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 2,
                "COOK_DESC" : "A couple of times a week",
                "CUISINE" : 2,
                "CUISINE_DESC" : "Mexican.Spanish",
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 2,
                "EATING_OUT_DESC" : "1-2 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 2,
                "ETHNIC_FOOD_DESC" : "unlikely",
                "EXERCISE" : 2,
                "EXERCISE_DESC" : "Twice or three times per week",
                "FAV_CUISINE" : "Italian/German",
                "FAV_CUISINE_CODED" : 1,
                "FAV_CUISINE_CODED_DESC" : "Italian/French/greek",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "Deer Steak, Buttered Pasta, Garlic Pasta",
                "FOOD_CHILDHOOD_SPLIT" : "Deer Steak",
                "FRUIT_DAY" : 3,
                "FRUIT_DAY_DESC" : "neutral",
                "GREEK_FOOD" : 1,
                "GREEK_FOOD_DESC" : "very unlikely",
                "HEALTHY_FEELING" : 4,
                "INCOME" : 5,
                "INCOME_DESC" : "$70,001 to $100,000",
                "INDIAN_FOOD" : 1,
                "INDIAN_FOOD_DESC" : "very unlikely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 5,
                "NUTRITIONAL_CHECK_DESC" : "on everything",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 3,
                "PAY_MEAL_OUT_DESC" : "$10.01 to $20.00",
                "PERSIAN_FOOD" : 2,
                "PERSIAN_FOOD_DESC" : "unlikely",
                "SELF_PERCEPTION_WEIGHT" : 3,
                "SELF_PERCEPTION_WEIGHT_DESC" : "just right",
                "SPORTS" : 2,
                "SPORTS_DESC" : "No",
                "THAI_FOOD" : 1,
                "THAI_FOOD_DESC" : "very unlikely",
                "VEGGIES_DAY" : 2,
                "VEGGIES_DAY_DESC" : "unlikely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 180
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+29",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 4.0,
                "GRADE" : "A",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 2,
                "COOK_DESC" : "A couple of times a week",
                "CUISINE" : 1,
                "CUISINE_DESC" : "American",
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 2,
                "EATING_OUT_DESC" : "1-2 times",
                "EMPLOYMENT" : 3,
                "EMPLOYMENT_DESC" : "no",
                "ETHNIC_FOOD" : 5,
                "ETHNIC_FOOD_DESC" : "very likely",
                "EXERCISE" : 2,
                "EXERCISE_DESC" : "Twice or three times per week",
                "FAV_CUISINE" : "italian",
                "FAV_CUISINE_CODED" : 1,
                "FAV_CUISINE_CODED_DESC" : "Italian/French/greek",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "chicken and biscuits",
                "FOOD_CHILDHOOD_SPLIT" : "chicken and biscuits",
                "FRUIT_DAY" : 5,
                "FRUIT_DAY_DESC" : "very likely",
                "GREEK_FOOD" : 5,
                "GREEK_FOOD_DESC" : "very likely",
                "HEALTHY_FEELING" : 5,
                "INCOME" : 4,
                "INCOME_DESC" : "$50,001 to $70,000",
                "INDIAN_FOOD" : 4,
                "INDIAN_FOOD_DESC" : "likely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 4,
                "NUTRITIONAL_CHECK_DESC" : "on most products",
                "ON_OFF_CAMPUS" : 4,
                "ON_OFF_CAMPUS_DESC" : "Own my own house",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 3,
                "PAY_MEAL_OUT_DESC" : "$10.01 to $20.00",
                "PERSIAN_FOOD" : 3,
                "PERSIAN_FOOD_DESC" : "neutral",
                "SELF_PERCEPTION_WEIGHT" : 4,
                "SELF_PERCEPTION_WEIGHT_DESC" : "slightly overweight",
                "SPORTS" : 2,
                "SPORTS_DESC" : "No",
                "THAI_FOOD" : 4,
                "THAI_FOOD_DESC" : "likely",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 205
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+30",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.4,
                "GRADE" : "B",
                "GENDER" : 2,
                "GENDER_DESC" : "Male",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 5,
                "COOK_DESC" : "Never, I really do not know my way around a kitchen",
                "CUISINE" : null,
                "CUISINE_DESC" : null,
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 3,
                "EATING_OUT_DESC" : "2-3 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 5,
                "ETHNIC_FOOD_DESC" : "very likely",
                "EXERCISE" : null,
                "EXERCISE_DESC" : null,
                "FAV_CUISINE" : "Spanish",
                "FAV_CUISINE_CODED" : 2,
                "FAV_CUISINE_CODED_DESC" : "Spanish/mexican",
                "FAV_FOOD" : null,
                "FAV_FOOD_DESC" : null,
                "FOOD_CHILDHOOD" : "Spaghetti, Chicken, Won Tons",
                "FOOD_CHILDHOOD_SPLIT" : "Spaghetti",
                "FRUIT_DAY" : 4,
                "FRUIT_DAY_DESC" : "likely",
                "GREEK_FOOD" : 5,
                "GREEK_FOOD_DESC" : "very likely",
                "HEALTHY_FEELING" : 5,
                "INCOME" : 5,
                "INCOME_DESC" : "$70,001 to $100,000",
                "INDIAN_FOOD" : 5,
                "INDIAN_FOOD_DESC" : "very likely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 4,
                "NUTRITIONAL_CHECK_DESC" : "on most products",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 3,
                "PARENTS_COOK_DESC" : "1-2 times a week",
                "PAY_MEAL_OUT" : 4,
                "PAY_MEAL_OUT_DESC" : "$20.01 to $30.00",
                "PERSIAN_FOOD" : 5,
                "PERSIAN_FOOD_DESC" : "very likely",
                "SELF_PERCEPTION_WEIGHT" : 4,
                "SELF_PERCEPTION_WEIGHT_DESC" : "slightly overweight",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 5,
                "THAI_FOOD_DESC" : "very likely",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : null
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+31",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 2.8,
                "GRADE" : "C+",
                "GENDER" : 1,
                "GENDER_DESC" : "Female",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 4,
                "COOK_DESC" : "I only help a little during holidays",
                "CUISINE" : 2,
                "CUISINE_DESC" : "Mexican.Spanish",
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 2,
                "EATING_OUT_DESC" : "1-2 times",
                "EMPLOYMENT" : 3,
                "EMPLOYMENT_DESC" : "no",
                "ETHNIC_FOOD" : 4,
                "ETHNIC_FOOD_DESC" : "likely",
                "EXERCISE" : 3,
                "EXERCISE_DESC" : "Once a week",
                "FAV_CUISINE" : "Italian ",
                "FAV_CUISINE_CODED" : 1,
                "FAV_CUISINE_CODED_DESC" : "Italian/French/greek",
                "FAV_FOOD" : 1,
                "FAV_FOOD_DESC" : "cooked at home",
                "FOOD_CHILDHOOD" : "Chicken Nuggets, Mac and Cheese, and pasta",
                "FOOD_CHILDHOOD_SPLIT" : "Chicken Nuggets",
                "FRUIT_DAY" : 3,
                "FRUIT_DAY_DESC" : "neutral",
                "GREEK_FOOD" : 5,
                "GREEK_FOOD_DESC" : "very likely",
                "HEALTHY_FEELING" : 7,
                "INCOME" : 3,
                "INCOME_DESC" : "$30,001 to $50,000",
                "INDIAN_FOOD" : 2,
                "INDIAN_FOOD_DESC" : "unlikely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 4,
                "NUTRITIONAL_CHECK_DESC" : "on most products",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 1,
                "PARENTS_COOK_DESC" : "Almost everyday",
                "PAY_MEAL_OUT" : 3,
                "PAY_MEAL_OUT_DESC" : "$10.01 to $20.00",
                "PERSIAN_FOOD" : 2,
                "PERSIAN_FOOD_DESC" : "unlikely",
                "SELF_PERCEPTION_WEIGHT" : 3,
                "SELF_PERCEPTION_WEIGHT_DESC" : "just right",
                "SPORTS" : 2,
                "SPORTS_DESC" : "No",
                "THAI_FOOD" : 1,
                "THAI_FOOD_DESC" : "very unlikely",
                "VEGGIES_DAY" : 3,
                "VEGGIES_DAY_DESC" : "neutral",
                "VITAMINS" : 1,
                "VITAMINS_DESC" : "Yes",
                "WEIGHT" : 128
              }
            },
            {
              "_index" : "foodcoded_analyze",
              "_type" : "kafka-connect",
              "_id" : "foodcoded_analyze+0+34",
              "_score" : 1.0,
              "_source" : {
                "GPA" : 3.7,
                "GRADE" : "B+",
                "GENDER" : 1,
                "GENDER_DESC" : "Female",
                "CALORIES_DAY" : 3,
                "CALORIES_DAY_DESC" : "it is moderately important",
                "COMFORT_FOOD_REASONS_CODED" : null,
                "COMFORT_FOOD_REASONS_CODED_DESC" : null,
                "COOK" : 3,
                "COOK_DESC" : "Whenever I can, but that is not very often",
                "CUISINE" : null,
                "CUISINE_DESC" : null,
                "DIET_CURRENT_CODED" : 2,
                "DIET_CURRENT_CODED_DESC" : "unhealthy/cheap/too much/random/",
                "EATING_OUT" : 4,
                "EATING_OUT_DESC" : "3-5 times",
                "EMPLOYMENT" : 2,
                "EMPLOYMENT_DESC" : "yes part time",
                "ETHNIC_FOOD" : 4,
                "ETHNIC_FOOD_DESC" : "likely",
                "EXERCISE" : 1,
                "EXERCISE_DESC" : "Everyday",
                "FAV_CUISINE" : "Italian or Chinese ",
                "FAV_CUISINE_CODED" : 1,
                "FAV_CUISINE_CODED_DESC" : "Italian/French/greek",
                "FAV_FOOD" : 3,
                "FAV_FOOD_DESC" : "both bought at store and cooked at home",
                "FOOD_CHILDHOOD" : "pizza, pasta, grilled cheese ",
                "FOOD_CHILDHOOD_SPLIT" : "pizza",
                "FRUIT_DAY" : 5,
                "FRUIT_DAY_DESC" : "very likely",
                "GREEK_FOOD" : 3,
                "GREEK_FOOD_DESC" : "neutral",
                "HEALTHY_FEELING" : 7,
                "INCOME" : 6,
                "INCOME_DESC" : "higher than $100,000",
                "INDIAN_FOOD" : 2,
                "INDIAN_FOOD_DESC" : "unlikely",
                "ITALIAN_FOOD" : 5,
                "ITALIAN_FOOD_DESC" : "very likely",
                "MARITAL_STATUS" : 1,
                "MARITAL_STATUS_DESC" : "Single",
                "NUTRITIONAL_CHECK" : 3,
                "NUTRITIONAL_CHECK_DESC" : "very rarely",
                "ON_OFF_CAMPUS" : 1,
                "ON_OFF_CAMPUS_DESC" : "On campus",
                "PARENTS_COOK" : 2,
                "PARENTS_COOK_DESC" : "2-3 times a week",
                "PAY_MEAL_OUT" : 3,
                "PAY_MEAL_OUT_DESC" : "$10.01 to $20.00",
                "PERSIAN_FOOD" : 1,
                "PERSIAN_FOOD_DESC" : "very unlikely",
                "SELF_PERCEPTION_WEIGHT" : 4,
                "SELF_PERCEPTION_WEIGHT_DESC" : "slightly overweight",
                "SPORTS" : 1,
                "SPORTS_DESC" : "Yes",
                "THAI_FOOD" : 2,
                "THAI_FOOD_DESC" : "unlikely",
                "VEGGIES_DAY" : 5,
                "VEGGIES_DAY_DESC" : "very likely",
                "VITAMINS" : 2,
                "VITAMINS_DESC" : "No",
                "WEIGHT" : 150
              }
            }
          ]
        }
      }
    ```
