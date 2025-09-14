# Part E — Bucketing

## Overview
This section demonstrates **Hive table bucketing**, a data organization technique that divides table data into a fixed number of buckets based on the hash value of one or more bucketing columns. Bucketing is particularly useful for improving JOIN performance and enabling efficient sampling.

## Task 1: Create Bucketed Hive Table

**Objective:** Create a bucketed table clustered by `pickup_location_id` to optimize queries and joins on location data.

### Create Bucketed Table

```sql
-- Create bucketed table clustered by pickup_location_id
CREATE TABLE trips_bucketed (
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
CLUSTERED BY (pickup_location_id) INTO 10 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

**Result:**
```
OK
Time taken: 0.099 seconds
```

**Key Features:**
- **CLUSTERED BY (pickup_location_id)**: Defines the bucketing column
- **INTO 10 BUCKETS**: Creates exactly 10 bucket files
- **Hash-based distribution**: Data distributed using hash function on pickup_location_id
- **Managed table**: Stored in Hive warehouse directory

## Task 2: Enable Bucketing and Configure Settings

**Objective:** Configure Hive settings to properly handle bucketed table operations.

### Configure Bucketing Settings

```sql
-- Enable bucketing
SET hive.enforce.bucketing=true;
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
```

**Configuration Explained:**
- `hive.enforce.bucketing=true`: Ensures proper bucketing during INSERT operations
- `hive.exec.dynamic.partition=true`: Enables dynamic partitioning (if needed)
- `hive.exec.dynamic.partition.mode=nonstrict`: Allows flexible partitioning mode

---

## Task 3: Insert Data into Bucketed Table

**Objective:** Load data from the external trips table into the bucketed table with proper bucket distribution.

### Insert Data with Bucketing

```sql
-- Insert data from trips_external into bucketed table
INSERT OVERWRITE TABLE trips_bucketed
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
    dropoff_location_id
FROM trips_external;
```

**Execution Details:**
```
Query ID = itversity_20250913085353_fe410a72-3617-4419-ae83-069acbfae899
Total jobs = 2
Launching Job 1 out of 2
Number of reduce tasks determined at compile time: 10
```

**Stage 1 - Bucketing Process:**
```
Starting Job = job_1757751399646_0004
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 10

2025-09-13 08:54:00,572 Stage-1 map = 0%,  reduce = 0%
2025-09-13 08:54:09,833 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 8.48 sec
2025-09-13 08:54:22,865 Stage-1 map = 100%,  reduce = 10%, Cumulative CPU 18.2 sec
2025-09-13 08:54:23,961 Stage-1 map = 100%,  reduce = 20%, Cumulative CPU 25.95 sec
2025-09-13 08:54:28,454 Stage-1 map = 100%,  reduce = 30%, Cumulative CPU 36.38 sec
2025-09-13 08:54:29,505 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 56.37 sec
2025-09-13 08:54:31,612 Stage-1 map = 100%,  reduce = 60%, Cumulative CPU 66.55 sec
2025-09-13 08:54:34,811 Stage-1 map = 100%,  reduce = 70%, Cumulative CPU 75.37 sec
2025-09-13 08:54:35,855 Stage-1 map = 100%,  reduce = 80%, Cumulative CPU 84.1 sec
2025-09-13 08:54:37,940 Stage-1 map = 100%,  reduce = 90%, Cumulative CPU 91.22 sec
2025-09-13 08:54:38,962 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 98.92 sec

MapReduce Total cumulative CPU time: 1 minutes 38 seconds 920 msec
Ended Job = job_1757751399646_0004
Loading data to table default.trips_bucketed
```

**Stage 2 - Final Processing:**
```
Launching Job 2 out of 2
Number of reduce tasks determined at compile time: 1
Hadoop job information for Stage-3: number of mappers: 1; number of reducers: 1

2025-09-13 08:54:50,816 Stage-3 map = 0%,  reduce = 0%
2025-09-13 08:54:55,949 Stage-3 map = 100%,  reduce = 0%, Cumulative CPU 1.9 sec
2025-09-13 08:55:01,074 Stage-3 map = 100%,  reduce = 100%, Cumulative CPU 4.16 sec

MapReduce Total cumulative CPU time: 4 seconds 160 msec
Ended Job = job_1757751399646_0005
```

**Final Result:**
```
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 10   Cumulative CPU: 98.92 sec   HDFS Read: 77967043 HDFS Write: 76823998 SUCCESS
Stage-Stage-3: Map: 1  Reduce: 1   Cumulative CPU: 4.16 sec   HDFS Read: 96154 HDFS Write: 9774 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 43 seconds 80 msec
OK
Time taken: 68.619 seconds
```

**Performance Analysis:**
- **Total execution time**: 68.6 seconds
- **Total CPU time**: 1 minute 43 seconds
- **HDFS Read**: 77.9 MB (source data)
- **HDFS Write**: 76.8 MB (bucketed data)
- **Reducers used**: 10 (one per bucket)
- **Two-stage process**: Bucketing + final table loading

**Bucketing Process Breakdown:**
1. **Stage 1**: Hash-based data distribution into 10 buckets (10 reducers)
2. **Stage 2**: Final table loading and metadata updates (1 reducer)
3. **Parallel processing**: All 10 buckets created simultaneously

---

## Task 4: Verify Row Count Matches

**Objective:** Confirm data integrity by verifying that all records were successfully transferred to the bucketed table.

### Count Records in Bucketed Table

```sql
SELECT COUNT(*) FROM trips_bucketed;
```

**Result:**
```
1048575
Time taken: 0.136 seconds, Fetched: 1 row(s)
```

**Performance Note:** 
- **Extremely fast count**: 0.136 seconds (vs. 20+ seconds for external table)
- **No MapReduce job**: Count retrieved from metadata/statistics
- **Bucketing benefit**: Optimized metadata tracking

### Count Records in Original External Table

```sql
SELECT COUNT(*) FROM trips_external;
```

**Execution Details:**
```
Query ID = itversity_20250913085517_ba45c849-3724-4b8d-b43f-c19b2d80ca26
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1

Starting Job = job_1757751399646_0006
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1

2025-09-13 08:55:25,135 Stage-1 map = 0%,  reduce = 0%
2025-09-13 08:55:30,264 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 3.06 sec
2025-09-13 08:55:35,382 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 5.46 sec

MapReduce Total cumulative CPU time: 5 seconds 460 msec
Ended Job = job_1757751399646_0006
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 5.46 sec   HDFS Read: 77818190 HDFS Write: 107 SUCCESS
Total MapReduce CPU Time Spent: 5 seconds 460 msec
```

**Result:**
```
1048575
Time taken: 20.276 seconds, Fetched: 1 row(s)
```

### Data Integrity Verification

| Table | Record Count | Query Time | Method |
|-------|-------------|------------|---------|
| **trips_external** | 1,048,575 | 20.276 seconds | MapReduce scan |
| **trips_bucketed** | 1,048,575 | 0.136 seconds | Metadata lookup |

**✅ Data Integrity Confirmed:**
- **Exact match**: Both tables contain identical record counts
- **No data loss**: All 1,048,575 records successfully transferred
- **Performance improvement**: 149x faster count query on bucketed table

---

## Additional Verification Commands

### Check HDFS Directory Structure

```bash
# Check bucketed table directory
hdfs dfs -ls /user/hive/warehouse/trips_bucketed/

- output: 10 bucket files
- 000000_0, 000001_0, 000002_0, ..., 000009_0
```

### Analyze Bucket Distribution

```sql
-- Check distribution of pickup locations across buckets
SELECT pickup_location_id, COUNT(*) as records_per_location
FROM trips_bucketed 
GROUP BY pickup_location_id 
ORDER BY records_per_location DESC 
LIMIT 20;

-- Sample data from bucketed table
SELECT * FROM trips_bucketed LIMIT 5;
```

### Table Properties

```sql
-- Check table structure and bucketing information
DESCRIBE FORMATTED trips_bucketed;
```

---

## Performance Benefits Analysis

### Bucketing Advantages Demonstrated

| Aspect | External Table | Bucketed Table |
|--------|----------------|----------------|
| **COUNT() Query** | 20.3 seconds (MapReduce) | 0.14 seconds (metadata) |
| **File Organization** | Single large file | 10 balanced bucket files |
| **JOIN Performance** | Full table scans | Bucket-to-bucket joins |
| **Sampling** | Random sampling | Efficient bucket sampling |
| **Query Pruning** | No optimization | Bucket pruning possible |


---

