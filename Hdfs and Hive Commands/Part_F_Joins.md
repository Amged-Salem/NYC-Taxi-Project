# Part F — Joins

## Overview
This section demonstrates **Hive table joins**, a fundamental operation for combining data from multiple tables based on related columns. We'll join trip data with zone data to enrich our analysis with meaningful location names and geographic information.



---

## Task 1: Create External Zones Table

**Objective:** Create an external table pointing to the zones data in `/user/root/landing_zone/zones/` to enable joins with trip data.

### Create External Zones Table

```sql
-- Create external zones table pointing to landing_zone/zones/
CREATE EXTERNAL TABLE zones_external (
    zone_id STRING,
    zone_name STRING,
    borough STRING,
    zone_geom STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/root/landing_zone/zones/'
TBLPROPERTIES ("skip.header.line.count"="1");
```

**Result:**
```
OK
Time taken: 1.179 seconds
```

**Key Features:**
- **External table**: Data remains in original landing_zone location
- **Header skipping**: `TBLPROPERTIES ("skip.header.line.count"="1")`
- **Comma-delimited**: Matches the CSV format of zones data
- **JOIN-ready**: zone_id can be used as join key with pickup_location_id

### Verify Zones Table Data

```sql
SELECT * FROM zones_external LIMIT 5;
```

**Result:**
```
1       Newark Airport  EWR     "POLYGON((-74.1856319999999 40.6916479999999
3       Allerton/Pelham Gardens Bronx   "POLYGON((-73.848596761 40.8716707849999
18      Bedford Park    Bronx   "POLYGON((-73.8844286139999 40.8668003789999
20      Belmont Bronx   "POLYGON((-73.8839239579998 40.8644177609999
31      Bronx Park      Bronx   "POLYGON((-73.8710017319999 40.8572767429999
Time taken: 2.877 seconds, Fetched: 5 row(s)
```



### Test Specific Zone Lookups

```sql
SELECT zone_id, zone_name, borough 
FROM zones_external 
WHERE zone_id IN ('1', '48', '68', '138', '230')
LIMIT 10;
```

**Result:**
```
1       Newark Airport  EWR
138     LaGuardia Airport       Queens
48      Clinton East    Manhattan
68      East Chelsea    Manhattan
230     Times Sq/Theatre District       Manhattan
Time taken: 0.428 seconds, Fetched: 5 row(s)
```

**Zone Analysis:**
- **Airport coverage**: Both Newark (1) and LaGuardia (138) airports
- **Manhattan focus**: Multiple Manhattan zones (48, 68, 230)
- **Popular destinations**: Times Square/Theatre District included
- **Fast lookup**: 0.428 seconds for targeted zone queries

---

## Task 2: Perform JOIN Between Trips and Zones

**Objective:** Join trip data with zone data to show pickup zone names alongside fare and tip information.

### Basic JOIN Query

```sql
-- Join trips with zones to show pickup zone names alongside fares and tips
SELECT 
    t.pickup_location_id,
    z.zone_name as pickup_zone_name,
    z.borough as pickup_borough,
    t.fare_amount,
    t.tip_amount,
    t.pickup_datetime,
    t.trip_distance
FROM trips_external t
JOIN zones_external z ON t.pickup_location_id = z.zone_id
LIMIT 10;
```

**Execution Details:**
```
Query ID = itversity_20250913115054_101dba84-3ddc-45fe-b753-cf13d5917a96
Total jobs = 1

2025-09-13 11:51:01     Starting to launch local task to process map join; maximum memory = 239075328
2025-09-13 11:51:03     Uploaded 1 File to: file:/tmp/itversity/8f91ea53-2a75-41d2-8ca7-f9e59d0c09f1/hive_2025-09-13_11-50-54_196_54913247629863419-1/-local-10004/HashTable-Stage-3/MapJoin-mapfile01--.hashtable (12655 bytes)
Execution completed successfully
MapredLocal task succeeded

Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_1757751399646_0007

Hadoop job information for Stage-3: number of mappers: 1; number of reducers: 0
2025-09-13 11:51:15,744 Stage-3 map = 0%,  reduce = 0%
2025-09-13 11:51:22,105 Stage-3 map = 100%,  reduce = 0%, Cumulative CPU 4.38 sec

MapReduce Total cumulative CPU time: 4 seconds 380 msec
Ended Job = job_1757751399646_0007
MapredLocal task succeeded
```

**Final Result:**
```
MapReduce Jobs Launched:
Stage-Stage-3: Map: 1   Cumulative CPU: 4.38 sec   HDFS Read: 622334 HDFS Write: 794 SUCCESS
Total MapReduce CPU Time Spent: 4 seconds 380 msec
OK
```

**JOIN Results:**
```
48      Clinton East    Manhattan       11.5    0       5/11/2018 17:40 1.6
138     LaGuardia Airport       Queens  28.5    5.96    3/22/2018 23:01 9.52
234     Union Sq        Manhattan       15.5    1.5     7/24/2018 9:58  2.17
79      East Village    Manhattan       6       0       12/21/2018 18:28   0.86
233     UN/Turtle Bay South     Manhattan       5.5     0       8/15/2018 13:58     0.3
261     World Trade Center      Manhattan       33      6.75    9/11/2018 14:33     4.8
229     Sutton Place/Turtle Bay North   Manhattan       26      5.45    6/13/2018 21:38     8.2
113     Greenwich Village North Manhattan       9       0       3/19/2018 0:15      1.8
231     TriBeCa/Civic Center    Manhattan       17.5    4.7     3/15/2018 22:12     4.38
234     Union Sq        Manhattan       14.5    3.05    1/21/2018 16:12 3
Time taken: 30.067 seconds, Fetched: 10 row(s)
```

**Performance Analysis:**
- **Total execution time**: 30.067 seconds
- **CPU time**: 4.38 seconds
- **HDFS Read**: 622,334 bytes (~608 KB)
- **Map-side join optimization**: Used hash table for zones (12,655 bytes)
- **No reducers**: Map-only job due to join optimization

**JOIN Optimization Details:**
1. **Map-side join detected**: Hive identified zones as small table
2. **Hash table creation**: Zones data loaded into memory (12.7 KB)
3. **Local task execution**: Hash table built locally before MapReduce
4. **Efficient processing**: No shuffle phase required

---



