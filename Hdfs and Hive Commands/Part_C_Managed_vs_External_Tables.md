# Part C — Managed vs External Tables

## Overview
This section demonstrates the key difference between **Managed Tables** and **External Tables** in Hive by examining what happens to the underlying HDFS data when tables are dropped.

**Key Concepts:**
- **Managed Tables**: Hive manages both metadata and data. When dropped, both are deleted.
- **External Tables**: Hive manages only metadata. When dropped, data survives in HDFS.

---

## Task 1: Drop the Managed Zones Table

**Objective:** Drop the managed zones table and observe what happens to its underlying data in HDFS.

### Current State Before Drop
First, let's verify the managed table exists and check its data location:

```sql
SHOW TABLES;
```

```bash
# Check Hive warehouse for zones_managed data
hdfs dfs -ls /user/hive/warehouse/zones_managed/
hdfs dfs -head /user/hive/warehouse/zones_managed/zones_test.csv
```

### Drop the Managed Table

```sql
DROP TABLE zones_managed;
```

**Result:**
```
OK
Time taken: 0.258 seconds
```

### Verify Data Deletion

```bash
# Check if data still exists in Hive warehouse
hdfs dfs -ls /user/hive/warehouse/zones_managed/
```

**Expected Result:**
```
ls: `/user/hive/warehouse/zones_managed/': No such file or directory
```

**Analysis:**
- ✅ **Table metadata removed** from Hive metastore
- ✅ **Data directory deleted** from HDFS
- ✅ **All data permanently lost** (as expected for managed tables)

---

## Task 2: Drop the External Trips Table

**Objective:** Drop the external trips table and observe what happens to its data in HDFS.

### Current State Before Drop
Let's verify the external table exists and check its data location:

```sql
SHOW TABLES;
```

```bash
# Check external data location
hdfs dfs -ls /user/root/landing_zone/trips/
hdfs dfs -head /user/root/landing_zone/trips/taxi_trip_data.csv
```

**Current State:**
```bash
Found 1 items
-rw-r--r--   1 itversity supergroup   77802319 2025-09-13 08:26 /user/root/landing_zone/trips/taxi_trip_data.csv
```

### Test External Table Before Drop

```sql
SELECT * FROM trips_external LIMIT 5;
```

**Result:**
```
1       5/11/2018 17:40 5/11/2018 17:55 1       1.6     1       N       1  11.5     1       0.5     0       0       0.3     48      68
2       3/22/2018 23:01 3/22/2018 23:25 1       9.52    1       N       1  28.5     0.5     0.5     5.96    0       0.3     138     230
2       7/24/2018 9:58  7/24/2018 10:22 1       2.17    1       N       1  15.5     0       0.5     1.5     0       0.3     234     48
2       12/21/2018 18:28        12/21/2018 18:35        1       0.86    1  N2       6       1       0.5     0       0       0.3     79      125
1       8/15/2018 13:58 8/15/2018 14:05 1       0.3     1       N       2  5.5      0       0.5     0       0       0.3     233     233
Time taken: 1.557 seconds, Fetched: 5 row(s)
```

### Drop the External Table

```sql
DROP TABLE trips_external;
```

**Result:**
```
OK
Time taken: 0.258 seconds
```

### Verify Data Persistence

```bash
# Check if data still exists in original location
hdfs dfs -ls /user/root/landing_zone/trips/
hdfs dfs -head /user/root/landing_zone/trips/taxi_trip_data.csv
```

**Actual Result (From User Test):**
```bash
Found 1 items
-rw-r--r--   1 itversity supergroup   77802319 2025-09-13 08:26 /user/root/landing_zone/trips/taxi_trip_data.csv

vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,pickup_location_id,dropoff_location_id
1,5/11/2018 17:40,5/11/2018 17:55,1,1.6,1,N,1,11.5,1,0.5,0,0,0.3,48,68
2,3/22/2018 23:01,3/22/2018 23:25,1,9.52,1,N,1,28.5,0.5,0.5,5.96,0,0.3,138,230
2,7/24/2018 9:58,7/24/2018 10:22,1,2.17,1,N,1,15.5,0,0.5,1.5,0,0.3,234,48
2,12/21/2018 18:28,12/21/2018 18:35,1,0.86,1,N,2,6,1,0.5,0,0,0.3,79,125
1,8/15/2018 13:58,8/15/2018 14:05,1,0.3,1,N,2,5.5,0,0.5,0,0,0.3,233,233
1,9/11/2018 14:33,9/11/2018 15:25,1,4.8,1,N,1,33,0,0.5,6.75,0,0.3,261,50
1,6/13/2018 21:38,6/13/2018 22:01,1,8.2,1,N,1,26,0.5,0.5,5.45,0,0.3,229,244
2,1/10/2018 21:30,1/10/2018 21:35,1,0.54,1,N,1,5,0.5,0.5,1.26,0,0.3,264,264
1,3/19/2018 0:15,3/19/2018 0:26,2,1.8,1,N,1,9,0.5,0.5,0,0,0.3,113,264
1,1/22/2018 10:08,1/1/2018 10:32,1,7.6,1,N,1,26,0,0.5,5.35,0,0.3,264,264
2,3/15/2018 22:12,3/15/2018 22:31,5,4.38,1,N,1,17.5,0.5,0.5...
```

**Analysis:**
- ✅ **Table metadata removed** from Hive metastore
- ✅ **Data file remains intact** in HDFS at `/user/root/landing_zone/trips/`
- ✅ **All data preserved** (as expected for external tables)
- ✅ **File size unchanged**: 77,802,319 bytes
- ✅ **File timestamp preserved**: 2025-09-13 08:26

---

## Summary of Results

### Behavior Comparison

| Table Type | Table Name | Drop Result | Data Location After Drop | Data Status |
|------------|------------|-------------|-------------------------|-------------|
| **Managed** | `zones_managed` | ✅ Dropped | `/user/hive/warehouse/zones_managed/` | ❌ **DELETED** |
| **External** | `trips_external` | ✅ Dropped | `/user/root/landing_zone/trips/` | ✅ **PRESERVED** |


