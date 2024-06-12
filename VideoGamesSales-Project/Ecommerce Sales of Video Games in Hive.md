# Ecommerce Sales of Video Games in Hive
## Creating the Client Database
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/6494701f-fe06-4377-a0f3-08e267009bf2)

## Existing MySQL Table
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/18c3a56f-159d-4586-992e-4a9fe8bc18d2)
## Creating a Hive Table before importing Data from MySQL and we are using Year of Release as Partition
```
create table video_games_table (Name string, Platform string, Genre string, Publisher string, NA_Sales double, EU_Sales double, JP_Sales double, Other_Sales double, Global_Sales double, Critical_Score int, Critic_Count int, User_Score int, User_Count int, Developer string, Rating string) partitioned by (Year_of_Release int) row format delimited fields terminated by ',';
```
## Importing MySQL Table Data into Hive Table
```
sqoop import --connect jdbc:mysql://localhost/video_games --username root --password cloudera --table video_games_table --hive-import -m 1
```
![image (10)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/94227b4a-c7ae-4ece-bfb3-70ab0b8882bf)

## Analysis and Creation of External Tables
### Total Global Sales in a year
```
SELECT Year_of_Release, (sum(Global_Sales)) as Total_Sales FROM video_games_table group by Year_of_Release order by Year_of_Release;
create external table ext0(Year_of_Release int, Global_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext0.txt';
insert overwrite table ext0 SELECT Year_of_Release, (sum(Global_Sales)) as Total_Sales FROM video_games_table group by Year_of_Release order by Year_of_Release;
```
### Top 10 Platforms and their Global Sales
```
SELECT Platform, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Platform order by Total_Sales desc limit 10;
create external table ext1(Platform string, Total_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext1.txt';
insert overwrite table ext1 SELECT Platform, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Platform order by Total_Sales desc limit 10;
```
### ### Top 10 Publisher and their Global Sales
```
SELECT Publisher, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Publisher order by Total_Sales desc limit 10;
create external table ext2(Publisher string, Total_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext2.txt';
insert overwrite table ext2 SELECT Publisher, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Publisher order by Total_Sales desc limit 10;
```
### Top 10 Genre and their Global Sales
```
SELECT Genre, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Genre order by Total_Sales desc limit 10;
create external table ext3(Genre string, Total_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext3.txt';
insert overwrite table ext1 SELECT Genre, (sum(NA_Sales)+sum(EU_Sales)+sum(JP_Sales)+sum(Other_Sales)) as Total_Sales FROM video_games_table GROUP BY Genre order by Total_Sales desc limit 10;
```
### Comparision of Sales in USA and Global Sales
```
select sum(NA_Sales) as Total_USA_Sales,sum(Global_Sales) as Total_Global_Sales from video_games_table where Year_of_Release>=2000 AND Year_of_Release<=2006;
create external table ext4 (USA_Sales float, Global_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext4.txt';
insert overwrite table ext4 select sum(NA_Sales) as Total_USA_Sales,sum(Global_Sales) as Total_Global_Sales from video_games_table where Year_of_Release>=2000 AND Year_of_Release<=2006;
```
### Most Popular Genre in USA
```
select Genre,sum(NA_Sales) as USA_Sales from video_games_table group by Genre order by USA_Sales desc limit 3;
create external table ext5(Genre string, USA_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext5.txt';
insert overwrite table ext5 select Genre,sum(NA_Sales) as USA_Sales from video_games_table group by Genre order by USA_Sales desc limit 3;
```
### Most popular Publisher in Japan
```
select Publisher,sum(JP_Sales) as Japan_Sales from  video_games_table group by Publisher order by Japan_Sales desc limit 1;
create external table ext6(Publisher string, JP_Sales float) row format delimited fields terminated by ',' location '/user/cloudera/VideoGames/ext6.txt';
insert overwrite table ext6 select Publisher,sum(JP_Sales) as Japan_Sales from  video_games_table group by Publisher order by Japan_Sales desc limit 1;
```
### External Files in the form of part files
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/4786ff26-4a2b-4343-a776-3d9d679c4665)

## Exporting External Tables to Client Database inside MySQL.
### Creating Blank External Table
```
create table ext0(Year_of_Release int, Global_Sales float);
create table ext1(Platform varchar(100), Total_Sales float);
create table ext2(Publisher varchar(100), Total_Sales float);
create table ext3(Genre varchar(100), Total_Sales float);
create table ext4(USA_Sales float, Global_Sales float);
create table ext5(Genre varchar(100), USA_Sales float);
create table ext6(Publisher varchar(100), JP_Sales float);
```
![image (1)](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/35912490-6d96-40e3-9911-f61fbbba00cb)
### Transfering Data from External Table to Client Database
```
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext0 --export-dir '/user/cloudera/VideoGames/ext0.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext1 --export-dir '/user/cloudera/VideoGames/ext1.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext2 --export-dir '/user/cloudera/VideoGames/ext2.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext3 --export-dir '/user/cloudera/VideoGames/ext3.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext4 --export-dir '/user/cloudera/VideoGames/ext4.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext5 --export-dir '/user/cloudera/VideoGames/ext5.txt/000000_0' --input-fields-terminated-by ','
sqoop export --connect jdbc:mysql://localhost/video_games_output --username root --password cloudera --table ext6 --export-dir '/user/cloudera/VideoGames/ext6.txt/000000_0' --input-fields-terminated-by ','
```
### Sqoop Jobs for transferring External Tables to Client DB Running
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/6c0a0b13-0bb8-450d-ac71-c61ec5be100f)

### External Tables imported in Client Database
![image](https://github.com/abirbhattacharya82/IBM-Big-Data-Training-Projects/assets/70687014/3a2008d1-3712-47b6-baed-25d68c2fe185)
