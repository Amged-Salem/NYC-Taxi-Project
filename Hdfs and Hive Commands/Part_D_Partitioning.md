# Part D — Partitioning

## Overview
This section demonstrates **Hive table partitioning**, a powerful technique for improving query performance by organizing data into separate directories based on partition keys. We'll create a partitioned external table for trips data, partitioned by year and month.

**Key Concepts:**
- **Partitioning**: Dividing large tables into smaller, manageable pieces based on partition keys
- **Dynamic Partitioning**: Automatically creating partitions during data insertion
- **Partition Pruning**: Query optimization that scans only relevant partitions

---

## Task 1: Create Partitioned External Table

**Objective:** Create an external table partitioned by year and month for better query performance.

### Create Directory for Partitioned Table

```bash
# Create base directory for partitioned table
hdfs dfs -mkdir -p /user/root/partitioned_trips
```

### Create Partitioned External Table

```sql
CREATE EXTERNAL TABLE trips_partitioned (
    vendor_id STRING,
    pickup_datetime STRING,
    dropoff_datetime STRING,
    passenger_count STRING,
    trip_distance STRING,
    rate_code STRING,
    store_and_fwd_flag STRING,
    payment_type STRING,
    fare_amount STRING,
    extra STRING,
    mta_tax STRING,
    tip_amount STRING,
    tolls_amount STRING,
    imp_surcharge STRING,
    pickup_location_id STRING,
    dropoff_location_id STRING
)
PARTITIONED BY (year INT, month INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/root/partitioned_trips/';
```

**Result:**
```
OK
Time taken: 0.992 seconds
```

**Key Features:**
- **PARTITIONED BY (year INT, month INT)**: Defines partition columns
- **External table**: Data location controlled by user
- **Partition columns not in main schema**: year and month are separate partition keys

### Initial State Check

```sql
SELECT * FROM trips_partitioned LIMIT 3;
```

**Result:**
```
OK
Time taken: 1.714 seconds
```
*(No results - table is empty initially)*

---

## Task 2: Enable Dynamic Partitioning

**Objective:** Configure Hive to automatically create partitions during data insertion.

### Configure Dynamic Partitioning Settings

```sql
-- Enable dynamic partitioning
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.exec.max.dynamic.partitions=1000;
SET hive.exec.max.dynamic.partitions.pernode=100;
```

**Configuration Explained:**
- `hive.exec.dynamic.partition=true`: Enables dynamic partitioning
- `hive.exec.dynamic.partition.mode=nonstrict`: Allows all partitions to be dynamic
- `hive.exec.max.dynamic.partitions=1000`: Maximum total dynamic partitions
- `hive.exec.max.dynamic.partitions.pernode=100`: Maximum partitions per mapper/reducer

---

## Task 3: Insert Data with Dynamic Partitioning

**Objective:** Load data from the existing trips_external table into the partitioned table using dynamic partitioning.

### Dynamic Partition Insert

```sql
INSERT OVERWRITE TABLE trips_partitioned 
PARTITION (year, month)
SELECT 
    vendor_id,
    pickup_datetime,
    dropoff_datetime,
    passenger_count,
    trip_distance,
    rate_code,
    store_and_fwd_flag,
    payment_type,
    fare_amount,
    extra,
    mta_tax,
    tip_amount,
    tolls_amount,
    imp_surcharge,
    pickup_location_id,
    dropoff_location_id,
    YEAR(FROM_UNIXTIME(UNIX_TIMESTAMP(pickup_datetime, 'M/d/yyyy H:mm'))) AS year,
    MONTH(FROM_UNIXTIME(UNIX_TIMESTAMP(pickup_datetime, 'M/d/yyyy H:mm'))) AS month
FROM trips_external;
```


**Final Result:**
```
OK
Time taken: 48.402 seconds
```

**Performance Analysis:**
- **Total execution time**: 48.4 seconds
- **HDFS Read**: 77.8 MB (source data)
- **HDFS Write**: 76.9 MB (partitioned data)
- **CPU time**: 24.96 seconds
- **Dynamic partitions created**: Automatically based on year/month values

**Date Parsing Logic:**
- `UNIX_TIMESTAMP(pickup_datetime, 'M/d/yyyy H:mm')`: Parses date string to Unix timestamp
- `FROM_UNIXTIME()`: Converts Unix timestamp to date
- `YEAR()` and `MONTH()`: Extract year and month for partitioning

---

## Task 4: Refresh Partitions with MSCK REPAIR

**Objective:** Ensure Hive metastore is synchronized with the actual partition directories created in HDFS.

### Run MSCK REPAIR TABLE

```sql
MSCK REPAIR TABLE trips_partitioned;
```

**Result:**
```
OK
Time taken: 0.118 seconds
```

**What MSCK REPAIR Does:**
- Scans HDFS directories for partition folders
- Updates Hive metastore with discovered partitions
- Ensures partition metadata consistency
- Fast operation (0.118 seconds)

---

## Task 5: Verify Partitions Exist

**Objective:** Confirm that partitions were created correctly and examine the data distribution.

### Show All Partitions

```sql
SHOW PARTITIONS trips_partitioned;
```

**Result:**
```
year=2008/month=12
year=2009/month=1
year=2017/month=12
year=2018/month=1
year=2018/month=2
year=2018/month=3
year=2018/month=4
year=2018/month=5
year=2018/month=6
year=2018/month=7
year=2018/month=8
year=2018/month=9
year=2018/month=10
year=2018/month=11
year=2018/month=12
year=2020/month=3
Time taken: 0.151 seconds, Fetched: 16 row(s)
```

**Partition Analysis:**
- **Total partitions**: 16
- **Date range**: 2008-2020 (mostly 2018)
- **Primary year**: 2018 (12 months of data)
- **Outliers**: Few records from 2008, 2009, 2017, 2020

### Count Records per Partition

```sql
SELECT year, month, COUNT(*) as record_count 
FROM trips_partitioned 
GROUP BY year, month 
ORDER BY year, month;
```



**Data Distribution Results:**
```
2008    12      7
2009    1       6
2017    12      6
2018    1       81811
2018    2       78910
2018    3       176884
2018    4       87100
2018    5       86461
2018    6       81148
2018    7       74042
2018    8       73121
2018    9       75337
2018    10      81738
2018    11      75481
2018    12      76522
2020    3       1
```

**Data Distribution Analysis:**
- **Total records**: 1,048,575 (matches original dataset)
- **Peak month**: March 2018 (176,884 records)
- **Consistent 2018 data**: ~70k-90k records per month
- **Data quality issues**: Few outlier records in other years
- **Performance benefit**: Queries can now target specific year/month partitions

---

### Partitioning Advantages Demonstrated

| Aspect | Before Partitioning | After Partitioning |
|--------|-------------------|-------------------|
| **Data Organization** | Single large table | 16 separate partitions |
| **Query Targeting** | Full table scan | Partition pruning |
| **Data Management** | Monolithic | Granular by time period |
| **Performance** | Slower for date ranges | Faster for specific periods |
| **Maintenance** | Single unit | Per-partition operations |


