# Part G — Transactional (ACID) Tables


**Key Concepts:**
- **ACID Properties**: Ensuring data consistency and reliability
- **Transactional Tables**: Tables that support UPDATE and DELETE operations
- **ORC Format**: Required storage format for ACID tables
- **Bucketing**: Mandatory for transactional tables
- **Compaction**: Background process to optimize ACID table performance

---

## Task 1: Enable Hive ACID Properties

**Objective:** Configure Hive session to support transactional operations.

### Configure ACID Settings

```sql
-- Enable ACID properties for transactional tables
SET hive.support.concurrency=true;
SET hive.enforce.bucketing=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
SET hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
SET hive.compactor.initiator.on=true;
SET hive.compactor.worker.threads=1;
```

**Configuration Explained:**
- `hive.support.concurrency=true`: Enables concurrent access to tables
- `hive.enforce.bucketing=true`: Ensures proper bucketing for ACID tables
- `hive.exec.dynamic.partition.mode=nonstrict`: Allows dynamic partitioning
- `hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager`: Sets transaction manager
- `hive.compactor.initiator.on=true`: Enables automatic compaction
- `hive.compactor.worker.threads=1`: Sets compaction worker threads

---

## Task 2: Create Transactional Bucketed Table

**Objective:** Create a transactional table with proper bucketing and ORC storage format.

### Create Transactional Table

```sql
-- Create transactional bucketed table for trips
CREATE TABLE trips_transactional (
    vendor_id STRING,
    pickup_datetime STRING,
    dropoff_datetime STRING,
    passenger_count INT,
    trip_distance DOUBLE,
    rate_code STRING,
    store_and_fwd_flag STRING,
    payment_type STRING,
    fare_amount DOUBLE,
    extra DOUBLE,
    mta_tax DOUBLE,
    tip_amount DOUBLE,
    tolls_amount DOUBLE,
    imp_surcharge DOUBLE,
    pickup_location_id STRING,
    dropoff_location_id STRING
)
CLUSTERED BY (pickup_location_id) INTO 4 BUCKETS
STORED AS ORC
TBLPROPERTIES ('transactional'='true');
```

**Result:**
```
OK
Time taken: 0.998 seconds
```

**Key Features:**
- **CLUSTERED BY (pickup_location_id) INTO 4 BUCKETS**: Required bucketing for ACID
- **STORED AS ORC**: Mandatory storage format for transactional tables
- **TBLPROPERTIES ('transactional'='true')**: Enables ACID operations
- **Proper data types**: INT and DOUBLE for numeric fields

**Why These Requirements?**
- **ORC format**: Supports ACID operations with row-level locking
- **Bucketing**: Enables efficient UPDATE/DELETE operations
- **4 buckets**: Balances parallelism with file management overhead

---

## Task 3: Insert Subset of Trips Data

**Objective:** Load a sample dataset (10,000 records) into the transactional table with proper data type conversion.

### Insert Data with Type Conversion

```sql
-- Insert a subset of trips (first 10,000 records) into transactional table
INSERT INTO trips_transactional
SELECT 
    vendor_id,
    pickup_datetime,
    dropoff_datetime,
    CAST(passenger_count AS INT) as passenger_count,
    CAST(trip_distance AS DOUBLE) as trip_distance,
    rate_code,
    store_and_fwd_flag,
    payment_type,
    CAST(fare_amount AS DOUBLE) as fare_amount,
    CAST(extra AS DOUBLE) as extra,
    CAST(mta_tax AS DOUBLE) as mta_tax,
    CAST(tip_amount AS DOUBLE) as tip_amount,
    CAST(tolls_amount AS DOUBLE) as tolls_amount,
    CAST(imp_surcharge AS DOUBLE) as imp_surcharge,
    pickup_location_id,
    dropoff_location_id
FROM trips_external
LIMIT 10000;
```

**Execution Details:**
```
Query ID = itversity_20250913121347_d6dfd435-5c04-4094-a892-2b623e9a684e
Total jobs = 3
```

**Stage 1 - Data Selection:**
```
Launching Job 1 out of 3
Number of reduce tasks determined at compile time: 1
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1

2025-09-13 12:13:56,116 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:14:02,347 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 4.77 sec
2025-09-13 12:14:08,531 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 7.51 sec

MapReduce Total cumulative CPU time: 7 seconds 510 msec
Ended Job = job_1757751399646_0008
```

**Stage 2 - Bucketing Process:**
```
Launching Job 2 out of 3
Number of reduce tasks determined at compile time: 4
Hadoop job information for Stage-2: number of mappers: 1; number of reducers: 4

2025-09-13 12:14:21,731 Stage-2 map = 0%,  reduce = 0%
2025-09-13 12:14:27,913 Stage-2 map = 100%,  reduce = 0%, Cumulative CPU 2.79 sec
2025-09-13 12:14:37,419 Stage-2 map = 100%,  reduce = 25%, Cumulative CPU 9.8 sec
2025-09-13 12:14:39,499 Stage-2 map = 100%,  reduce = 75%, Cumulative CPU 22.4 sec
2025-09-13 12:14:40,534 Stage-2 map = 100%,  reduce = 100%, Cumulative CPU 28.41 sec

MapReduce Total cumulative CPU time: 28 seconds 410 msec
Ended Job = job_1757751399646_0009
Loading data to table default.trips_transactional
```

**Stage 3 - Final Processing:**
```
Launching Job 3 out of 3
Number of reduce tasks determined at compile time: 1
Hadoop job information for Stage-4: number of mappers: 1; number of reducers: 1

2025-09-13 12:14:54,559 Stage-4 map = 0%,  reduce = 0%
2025-09-13 12:14:59,694 Stage-4 map = 100%,  reduce = 0%, Cumulative CPU 1.9 sec
2025-09-13 12:15:05,850 Stage-4 map = 100%,  reduce = 100%, Cumulative CPU 4.19 sec

MapReduce Total cumulative CPU time: 4 seconds 190 msec
Ended Job = job_1757751399646_0010
```

**Final Result:**
```
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 7.51 sec   HDFS Read: 1009540 HDFS Write: 1219660 SUCCESS
Stage-Stage-2: Map: 1  Reduce: 4   Cumulative CPU: 28.41 sec   HDFS Read: 1284225 HDFS Write: 237836 SUCCESS
Stage-Stage-4: Map: 1  Reduce: 1   Cumulative CPU: 4.19 sec   HDFS Read: 51683 HDFS Write: 8770 SUCCESS
Total MapReduce CPU Time Spent: 40 seconds 110 msec
OK
Time taken: 80.019 seconds
```

**Performance Analysis:**
- **Total execution time**: 80.019 seconds
- **Total CPU time**: 40.11 seconds
- **Three-stage process**: Selection → Bucketing → Final loading
- **4 reducers in Stage 2**: One per bucket for parallel processing
- **HDFS I/O**: ~1MB read, ~1.5MB written (data transformation and ORC compression)

### Verify Data Loading

```sql
SELECT COUNT(*) FROM trips_transactional;
```

**Execution Details:**
```
Query ID = itversity_20250913121516_33b85f1d-88bc-4a01-bbdb-c5d5de304b64
Hadoop job information for Stage-1: number of mappers: 4; number of reducers: 1

2025-09-13 12:15:23,959 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:15:33,635 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 23.85 sec
2025-09-13 12:15:39,849 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 27.12 sec

MapReduce Jobs Launched:
Stage-Stage-1: Map: 4  Reduce: 1   Cumulative CPU: 27.12 sec   HDFS Read: 109203 HDFS Write: 105 SUCCESS
Total MapReduce CPU Time Spent: 27 seconds 120 msec
```

**Result:**
```
10000
Time taken: 24.368 seconds, Fetched: 1 row(s)
```

**Verification Analysis:**
- **Exact count match**: 10,000 records successfully loaded
- **4 mappers**: One per bucket file
- **Efficient processing**: 24.4 seconds for count operation

---

## Task 4: Query Trips with passenger_count = 1

**Objective:** Identify records that will be updated to demonstrate UPDATE operation.

### Find Single-Passenger Trips

```sql
SELECT COUNT(*) as trips_with_1_passenger
FROM trips_transactional 
WHERE passenger_count = 1;
```

**Execution Details:**
```
Query ID = itversity_20250913121620_9a9dee8d-4bce-447a-a346-fa2977ab7f79
Hadoop job information for Stage-1: number of mappers: 4; number of reducers: 1

2025-09-13 12:16:28,293 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:16:52,904 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 106.68 sec
2025-09-13 12:17:02,145 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 110.43 sec

MapReduce Jobs Launched:
Stage-Stage-1: Map: 4  Reduce: 1   Cumulative CPU: 110.43 sec   HDFS Read: 116443 HDFS Write: 104 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 50 seconds 430 msec
```

**Result:**
```
7064
Time taken: 43.377 seconds, Fetched: 1 row(s)
```


---

## Task 5: Update passenger_count from 1 to 2

**Objective:** Demonstrate UPDATE operation on transactional table.

### Perform UPDATE Operation

```sql
UPDATE trips_transactional 
SET passenger_count = 2 
WHERE passenger_count = 1;
```

**Execution Details:**
```
Query ID = itversity_20250913121722_6cfb5382-856c-47d0-9e48-35f3fe3c15a1
Number of reduce tasks determined at compile time: 4
Hadoop job information for Stage-1: number of mappers: 4; number of reducers: 4

2025-09-13 12:17:30,651 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:17:51,932 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 51.33 sec
2025-09-13 12:18:01,342 Stage-1 map = 100%,  reduce = 25%, Cumulative CPU 97.79 sec
2025-09-13 12:18:02,396 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 104.85 sec
2025-09-13 12:18:03,449 Stage-1 map = 100%,  reduce = 75%, Cumulative CPU 112.28 sec
2025-09-13 12:18:04,483 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 119.31 sec

MapReduce Jobs Launched:
Stage-Stage-1: Map: 4  Reduce: 4   Cumulative CPU: 119.31 sec   HDFS Read: 353891 HDFS Write: 158115 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 59 seconds 310 msec
```

**Result:**
```
OK
Time taken: 43.912 seconds
```

**UPDATE Operation Analysis:**
- **Total execution time**: 43.912 seconds
- **CPU time**: 1 minute 59 seconds
- **4 mappers, 4 reducers**: One per bucket for parallel processing
- **HDFS I/O**: 353KB read, 158KB written
- **Updated 7,064 records**: All single-passenger trips

**ACID UPDATE Process:**
1. **Read phase**: Identify records matching WHERE condition
2. **Update phase**: Create new versions of matching records
3. **Write phase**: Write updated records to delta files
4. **Metadata update**: Update transaction log

---

## Task 6: Delete Trips with trip_distance = 0

**Objective:** Demonstrate DELETE operation on transactional table.

### Find Zero-Distance Trips

```sql
SELECT COUNT(*) as trips_with_zero_distance
FROM trips_transactional 
WHERE trip_distance = 0.0;
```

**Execution Details:**
```
Query ID = itversity_20250913121827_499bee8f-884c-4342-ad0b-4dac1df57cb1
Hadoop job information for Stage-1: number of mappers: 8; number of reducers: 1

2025-09-13 12:18:35,031 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:20:00,868 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:20:03,067 Stage-1 map = 50%,  reduce = 0%, Cumulative CPU 204.29 sec
2025-09-13 12:20:04,104 Stage-1 map = 75%,  reduce = 0%, Cumulative CPU 218.5 sec
2025-09-13 12:20:13,523 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 230.62 sec
2025-09-13 12:20:14,560 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 233.41 sec

MapReduce Jobs Launched:
Stage-Stage-1: Map: 8  Reduce: 1   Cumulative CPU: 233.41 sec   HDFS Read: 280039 HDFS Write: 102 SUCCESS
Total MapReduce CPU Time Spent: 3 minutes 53 seconds 410 msec
```

**Result:**
```
65
Time taken: 109.48 seconds, Fetched: 1 row(s)
```

**Analysis:**
- **65 trips** with zero distance (0.65% of data)
- **8 mappers**: Increased from 4 due to delta files from UPDATE operation
- **High execution time**: 109.48 seconds (processing base + delta files)

### Perform DELETE Operation

```sql
DELETE FROM trips_transactional 
WHERE trip_distance = 0.0;
```

**Execution Details:**
```
Query ID = itversity_20250913122033_1ebe69aa-7765-441f-8d48-2a860bf90021
Number of reduce tasks determined at compile time: 4
Hadoop job information for Stage-1: number of mappers: 8; number of reducers: 4

2025-09-13 12:20:43,935 Stage-1 map = 0%,  reduce = 0%
2025-09-13 12:20:58,030 Stage-1 map = 50%,  reduce = 0%, Cumulative CPU 31.69 sec
2025-09-13 12:20:59,056 Stage-1 map = 75%,  reduce = 0%, Cumulative CPU 49.43 sec
2025-09-13 12:21:11,198 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 72.63 sec
2025-09-13 12:21:15,429 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 97.2 sec

MapReduce Jobs Launched:
Stage-Stage-1: Map: 8  Reduce: 4   Cumulative CPU: 97.2 sec   HDFS Read: 300191 HDFS Write: 5828 SUCCESS
Total MapReduce CPU Time Spent: 1 minutes 37 seconds 200 msec
```

**Result:**
```
OK
Time taken: 45.451 seconds
```

**DELETE Operation Analysis:**
- **Total execution time**: 45.451 seconds
- **CPU time**: 1 minute 37 seconds
- **8 mappers, 4 reducers**: Processing both base and delta files
- **HDFS I/O**: 300KB read, 6KB written
- **Deleted 65 records**: All zero-distance trips

---

## Performance Analysis and ACID Characteristics

### Operation Performance Comparison

| Operation | Records Affected | Execution Time | CPU Time | Mappers | Notes |
|-----------|------------------|----------------|----------|---------|--------|
| **INSERT** | 10,000 | 80.02s | 40.11s | 1→4 | Three-stage process |
| **COUNT (initial)** | 10,000 | 24.37s | 27.12s | 4 | Baseline performance |
| **SELECT (filter)** | 7,064 found | 43.38s | 110.43s | 4 | High CPU for filtering |
| **UPDATE** | 7,064 updated | 43.91s | 119.31s | 4→4 | Row-level updates |
| **SELECT (post-update)** | 65 found | 109.48s | 233.41s | 8 | Delta files impact |
| **DELETE** | 65 deleted | 45.45s | 97.2s | 8→4 | Final cleanup |

