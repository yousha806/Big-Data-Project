# Ecommerce Sales of Video Games in Pig
### Creating a CSV from the client Database and Putting it in HDFS
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/d538cfaf-8078-4056-b9e6-18147a364394)
The file is already there in HDFS <br>
So we are creating Pig relations to load them.
```
videogames= load '/user/cloudera/VideoGames/Video_Games.csv' using PigStorage(',') as (Name:chararray, Platform:chararray, Year_of_Release:int, Genre:chararray, Publisher:chararray, NA_Sales:double, EU_Sales:double, JP_Sales:double, Other_Sales:double, Global_Sales:double, Critical_Score:int, Critic_Count:int, User_Score:int, User_Count:int, Developer:chararray, Rating:chararray);
```
Viewing the Pig relation
```
dump videogames
```
![image (2)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/5fa697ec-5c9e-4d0c-af9d-8f9c70e594db)
### Total Sales in each Year
```
grunt> res1= foreach videogames generate Year_of_Release,Global_Sales;
grunt> grouped_by_year = GROUP res1 BY Year_of_Release;
grunt> sales_sum_by_year = FOREACH grouped_by_year GENERATE group AS Year_of_Release, SUM(res1.Global_Sales) AS total_sales;
grunt> dump sales_sum_by_year;
grunt> STORE sales_sum_by_year INTO '/user/cloudera/VideoGames/PigOutput1' USING PigStorage(',');
```
#### Final Relation
![image (4)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/ec797ad2-5e5c-47c0-ba56-3bd4e0412fd1)
#### Storing it as a Part File
![image (5)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/13ab3102-f50a-4556-8049-93647925be50)

### Total Sales region wise
```
grunt> res2= foreach videogames generate Year_of_Release,NA_Sales,EU_Sales,JP_Sales;
grunt> grunt> sales_sum_by_year = FOREACH grouped_by_year GENERATE
>>     SUM(res2.EU_Sales) AS total_EU_sales,
>>     SUM(res2.JP_Sales) AS total_JP_sales,
>>     SUM(res2.NA_Sales) AS total_NA_sales;
grunt> STORE sales_sum_by_year INTO '/user/cloudera/VideoGames/pigOutput2' USING PigStorage(',');
```
#### Final Relation
![image (7)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/f3d7870d-edef-4281-aaaa-bae34af14cdd)
#### Storing it as a Part File
![image (6)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/555c0553-e1c7-4571-80b5-32a54b81514b)

