# Ecommerce Sales of Video Games in Hbase
There is an existing MySQL Table already with the video games data. But there are no Primary or Unique value columns there. So we will get an error regarding row-key. To avoid that we are adding a column named id which will have unique values.
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/6dd64f84-b6ff-4f37-842b-4b5f90c15b9c)

### Creating the Table
```
create 'video_games_table','cf1'
```
### Importing Data from MySQL to Hbase
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/ed6bfdd0-ac43-4e5c-aad7-21c6da995597)

### Importing Hbase table to Hive for analysis
```
create external table video_games_hbase(id int,Name string, Platform string, Year_of_Release int, Genre string, Publisher string, NA_Sales double, EU_Sales double, JP_Sales double, Other_Sales double, Global_Sales double, Critical_Score int, Critic_Count int, User_Score int, User_Count int, Developer string, Rating string ) stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping"=":key,cf1:Name, cf1:Platform, cf1:Year_of_Release, cf1:Genre, cf1:Publisher, cf1:NA_Sales, cf1:EU_Sales, cf1:JP_Sales, cf1:Other_Sales, cf1:Global_Sales,cf1: Critical_Score, cf1:Critic_Count, cf1:User_Score, cf1:User_Count, cf1:Developer, cf1:Rating") TBLPROPERTIES("hbase.table.name"="video_games_table","hbase.mapred.output.outputtable"="video_games_table");
```
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/e7e2fcc2-56e4-400d-8aea-b08a65171f4a)

### Total Sales through the years and storing it in an external table
```
select Year_of_Release, sum(Global_Sales) from video_games_hbase group by Year_of_Release order by Year_of_Release limit 10;
create external table ext0_hbase(Year_of_Release int, Global_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext0_hbase.txt';
insert overwrite table ext0_hbase SELECT Year_of_Release, (sum(Global_Sales)) as Total_Sales FROM video_games_hbase group by Year_of_Release order by Year_of_Release;
```
### Region wise Sales through the years and storing it in an external table
```
select Year_of_Release, sum(NA_Sales), sum(EU_Sales), sum(JP_Sales) from video_games_hbase group by Year_of_Release order by Year_of_Release limit 10;
create external table ext1_hbase(Platform string, Total_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext1_hbase.txt';
insert overwrite table ext1_hbase SELECT Platform, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_hbase GROUP BY Platform order by Total_Sales desc limit 10;
```
### Preparing Empty Table in the Client Database
```
create table ext0_hbase(Year_of_Release int, Global_Sales float);
create table ext1_hbase(Platform varchar(100), Total_Sales float);
```
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/4a3ce4b0-ad8d-4002-a657-eab155f61638)

### Sqoop Pipeline to Transfer Data from Hive to Client Database
```
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext0_hbase --export-dir '/user/cloudera/VideoGames/ext0_hbase.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext1_hbase --export-dir '/user/cloudera/VideoGames/ext1_hbase.txt/000000_0' --input-fields-terminated-by ','
```
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/f9939c5c-81c5-4880-bc0b-9e18a01e7edf)
