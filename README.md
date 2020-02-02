# Viaplay-Technical-Test

## A quick explanation over the code

An ETL project which analyzes customers' behavior and content using the whatson and the 
stream dataset provided by [Viaplay](https://viaplay.se/ "Viaplay's Homepage").

The project is done with [Apache Maven](https://maven.apache.org/ "Apache Maven Homepage") and 
[Spark](https://spark.apache.org/docs/latest/ "Apache Spark Latest Doc Page") using the Java API. 

There is only one class called Logic which handles all the ETL logic.

The code does the following procedures:

1. Loads the necessary libraries.

2. Instantiates a *SparkSession* class.

3. Declares Schemas for the data that are expected from the CSVs using the class *StructType*.  

4. Creates DataFrames from the CSV files using the class *Dataset<Row>* and applies the schemas from the previous step.

5. Registers the DataFrames from the previous step as temporary tables.

6. For each task given, executes a Spark SQL query and saves the resulting tables in a DataFrame.

7. Prints the schemas for the DataFrames created in the previous step (for tesing purposes).

8. Exports the DataFrames in CSV format and saves them on disc for further analysis.

9. The Spark session ends

## Dataset Documentation

For the first task the resulting CSV file has the following schema:

```
root
 |-- dt: date (nullable = true)
 |-- time: string (nullable = true)
 |-- device_name: string (nullable = true)
 |-- house_number: string (nullable = true)
 |-- user_id: string (nullable = true)
 |-- country_code: string (nullable = true)
 |-- program_title: string (nullable = true)
 |-- season: string (nullable = true)
 |-- season_episode: string (nullable = true)
 |-- genre: string (nullable = true)
 |-- product_type: string (nullable = true)
 |-- broadcast_right_start_date: date (nullable = true)
 |-- broadcast_right_end_date: date (nullable = true)
```
The dt,house_number and the broadcast_rights columns are coming from the whatson dataframe and the rest are coming from the stream dataframe. Inside the SQL query starting "bottom-up", we begin by filtering out the whatson table. We have two nested *SELECT* statements and one self join using the whatson table which provide us with the most recent data. Once we get the fresh data and do some *GROUP BY* statements, we join the produced whatson table with the stream table on the house_number where the program_type is TVOD or EST.  

For the second task the schema is the following:
```
root
 |-- dt: date (nullable = true)
 |-- program_title: string (nullable = true)
 |-- device_name: string (nullable = true)
 |-- country_code: string (nullable = true)
 |-- product_type: string (nullable = true)
 |-- unique_users: long (nullable = false)
 |-- content_count: long (nullable = false)
 ```
All the generated columns are coming from the stream dataframe. The only "custom" columns are the unique_users and content_count which are generated by performing *COUNT(UNIQUE user_id)* statement for the users and *COUNT(house_number)* for the content. Finally the results are obtained by doing the necessary *GROUP BY* statements. 
 
And finally for the last task the generated schema is:
```
root
 |-- watched_time: integer (nullable = true)
 |-- genre: string (nullable = true)
 |-- unique_users: long (nullable = true)
```
Which was a more complicated greatest-n-per-group type of query. Here the column genre is coming from the stream dataframe and the rest are generated through nested *SELECT* statements. In the innermost *SELECT* statement we find out the number of unique users and moving upwards we get the to the point where we sum the number of unique users for every genre for every hour of the day. The last piece of the puzzle is to do a pseudo-self *JOIN* of the current table using a slight variation of the same table that contains the max values of the number of unique_users and then use *GROUP BY* to get the desired results. 

The types of columns unique_users and content_count, from the task 2 and 3, could be converted to integer using the function:
```
withColumn("COLUMN_NAME", (col("COLUMN_NAME").cast("integer")))
```
However i thought that in this particular case it is not that important since the data are written into a CSV without storing the data types in the headers and the tables are destroyed once the spark session ends.
