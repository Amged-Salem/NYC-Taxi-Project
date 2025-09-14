# Part B — Hive Tables
## Task 1: Create a Managed Table for the Zones Dataset

**Objective:** Create a managed table for the zones dataset with proper schema definition.

### Data Structure Analysis
First, we examined the zones data structure:
```
zone_id,zone_name,borough,zone_geoms
1,Newark Airport,EWR,"POLYGON((-74.1856319999999 40.6916479999999, ...))"
3,Allerton/Pelham Gardens,Bronx,"POLYGON((-73.848596761 40.8716707849999, ...))"
```

**Important Discovery:** The zones file is **comma-separated**, not tab-separated!

### Create Managed Table Command (Correct Solution):
```sql
CREATE TABLE zones_managed (
    zone_id STRING,
    zone_name STRING,
    borough STRING,
    zone_geom STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

**Result:**
```
OK
Time taken: 0.509 seconds
```

**Key Features of Managed Table:**
- Hive manages both metadata and data
- Data is stored in Hive warehouse directory
- Dropping table deletes both metadata and data
- **Comma-delimited format** for data parsing (not tab-delimited)
- **STRING data types** used to avoid parsing issues

---

## Task 2: Load Data from HDFS into Managed Table

**Objective:** Load the taxi zone data from HDFS into the managed table.

### Load Data Command (Correct Solution):
```bash
# First, upload the file to a temporary location
hdfs dfs -put /data/etisalat/taxi_zone_geo.csv /tmp/zones_test.csv
```

```sql
LOAD DATA INPATH '/tmp/zones_test.csv' INTO TABLE zones_managed;
```

**Result:**
```
Loading data to table default.zones_managed
OK
Time taken: 0.485 seconds
```

**What Happened:**
- Data file was **moved** from temporary location to Hive warehouse
- Using `/tmp/` location avoids path conflicts
- Data is now managed entirely by Hive
- **265 records** loaded successfully (including header)

---

## Task 3: Create External Table for Trips Dataset

**Objective:** Create an external table pointing to the trips data in HDFS location.

### Data Structure Analysis
First, we examined the trips data structure:
```
vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,pickup_location_id,dropoff_location_id
1,5/11/2018 17:40,5/11/2018 17:55,1,1.6,1,N,1,11.5,1,0.5,0,0,0.3,48,68
2,3/22/2018 23:01,3/22/2018 23:25,1,9.52,1,N,1,28.5,0.5,0.5,5.96,0,0.3,138,230
```

```bash
# First, create the directory and upload the file to the external location
hdfs dfs -mkdir -p /user/root/landing_zone/trips
hdfs dfs -put /data/etisalat/taxi_trip_data.csv /user/root/landing_zone/trips/
```

```sql
CREATE EXTERNAL TABLE trips_external (
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
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/root/landing_zone/trips/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

**Result:**
```
OK
Time taken: 1.027 seconds
```

**Key Features of True External Table:**
- Used **EXTERNAL TABLE** keyword
- Used **LOCATION** clause pointing to existing data
- **No LOAD DATA INPATH** - data stays in original location
- **TBLPROPERTIES** to skip header row
- **Comma-separated delimiter** (`,`) correctly specified
- Data survives table drops

---

## Task 4: Verify Data Accessibility with SELECT Queries

**Objective:** Run SELECT queries to verify that both tables are accessible and contain data.

### 4.1 Count Records in Managed Table

```sql
SELECT COUNT(*) FROM zones_managed;
```

**Execution Details:**
```
Query ID = itversity_20250911201245_e8b61f74-a031-434a-b53e-d33eccfb9870
Total jobs = 1
MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 4.42 sec   HDFS Read: 3495871 HDFS Write: 103 SUCCESS
Total MapReduce CPU Time Spent: 4 seconds 420 msec
```

**Result (Working Solution):**
```
265
Time taken: ~20 seconds, Fetched: 1 row(s)
```

**Sample Data:**
```sql
SELECT * FROM zones_managed LIMIT 3;
```
```
zone_id    zone_name              borough    zone_geom
zone_id    zone_name              borough    zone_geom
1          Newark Airport         EWR        POLYGON((-74.1856319999999 40.6916479999999, ...))
3          Allerton/Pelham Gardens Bronx     POLYGON((-73.848596761 40.8716707849999, ...))
```

**Analysis:**
- **265 zones** in the dataset (including header row)
- HDFS Read: ~3.5 MB (zones data)
- Processing time: ~20 seconds
- Used 1 Mapper and 1 Reducer
- **Problem Solved:** Data now displays correctly with comma delimiter

### 4.2 Test True External Table

```sql
SELECT * FROM trips_external LIMIT 5;
```

**Result (True External Table):**
```
1       5/11/2018 17:40 5/11/2018 17:55 1       1.6     1       N       1  11.5     1       0.5     0       0       0.3     48      68
2       3/22/2018 23:01 3/22/2018 23:25 1       9.52    1       N       1  28.5     0.5     0.5     5.96    0       0.3     138     230
2       7/24/2018 9:58  7/24/2018 10:22 1       2.17    1       N       1  15.5     0       0.5     1.5     0       0.3     234     48
2       12/21/2018 18:28        12/21/2018 18:35        1       0.86    1  N2       6       1       0.5     0       0       0.3     79      125
1       8/15/2018 13:58 8/15/2018 14:05 1       0.3     1       N       2  5.5      0       0.5     0       0       0.3     233     233
Time taken: 1.557 seconds, Fetched: 5 row(s)
```

**Analysis:**
- **Header row automatically skipped** due to `TBLPROPERTIES ("skip.header.line.count"="1")`
- Data displays correctly with comma delimiter
- Fast query execution (~1.5 seconds)
- Data remains in original HDFS location: `/user/root/landing_zone/trips/`

---

```

1. **Always verify file format first**: Use `hdfs dfs -head` to check delimiter and structure
2. **Use STRING data types**: Avoids parsing issues with complex data
3. **LOAD DATA INPATH is more reliable**: Better than LOCATION for external tables
4. **Both files were comma-separated**: Not tab-separated as initially assumed
5. **Temporary file approach works**: Using `/tmp/` locations avoids path issues

### Final Architecture:

| Table | Type | Delimiter | Data Types | Creation Method | Data Location |
|-------|------|-----------|------------|----------------|---------------|
| **zones_managed** | Managed | Comma (`,`) | All STRING | LOAD DATA INPATH | Hive Warehouse |
| **trips_external** | External | Comma (`,`) | All STRING | LOCATION clause | /user/root/landing_zone/trips/ |

**Key Differences:**
- **Managed Table**: Data moved to Hive warehouse, deleted when table dropped
- **External Table**: Data stays in original location, survives table drops

---




