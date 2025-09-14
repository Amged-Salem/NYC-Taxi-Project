# Part A — HDFS Practice



## Task 1: Create HDFS Directories

**Objective:** Create landing_zone/zones, landing_zone/trips, and Archive directories in HDFS.

### Commands:
```bash
hdfs dfs -mkdir -p /user/root/landing_zone
hdfs dfs -mkdir -p /user/root/landing_zone/zones
hdfs dfs -mkdir -p /user/root/landing_zone/trips
hdfs dfs -mkdir -p /user/root/Archive
```

**Verification:**
```bash
hdfs dfs -ls /user/root/landing_zone/
```

---

## Task 2: Upload Files to Landing Zone

**Objective:** Upload taxi_zone_geo.csv and taxi_trip_data.csv into their respective landing zone directories.

### Commands:
```bash
# Upload taxi_zone_geo.csv to zones directory
hdfs dfs -put /data/etisalast/taxi_zone_geo.csv /user/root/landing_zone/zones/

# Upload taxi_trip_data.csv to trips directory
hdfs dfs -put /data/etisalast/taxi_trip_data.csv /user/root/landing_zone/trips/
```

---

## Task 3: Verify File Upload

**Objective:** Verify that the files were uploaded correctly.

### Commands:
```bash
# List contents of landing_zone
hdfs dfs -ls /user/root/landing_zone/

# List contents of zones directory
hdfs dfs -ls /user/root/landing_zone/zones/

# List contents of trips directory
hdfs dfs -ls /user/root/landing_zone/trips/

# Check file sizes
hdfs dfs -du -h /user/root/landing_zone/
```

---

## Task 4: Create Backup Copy

**Objective:** Copy the trips file into a backup directory inside /landing_zone/trips/archive_copy.

### Commands:
```bash
# Create backup directory
hdfs dfs -mkdir -p /user/root/landing_zone/trips/archive_copy

# Copy trips file to backup directory
hdfs dfs -cp /user/root/landing_zone/trips/taxi_trip_data.csv /user/root/landing_zone/trips/archive_copy/

# Verify backup creation
hdfs dfs -ls /user/root/landing_zone/trips/archive_copy/
```

---

## Task 5: Move File to Archive

**Objective:** Move the trips file to /Archive.

### Commands:
```bash
# Move trips file to Archive directory
hdfs dfs -mv /user/root/landing_zone/trips/taxi_trip_data.csv /user/root/Archive/
```

**Verification:**
```bash
hdfs dfs -ls /user/root/Archive/
hdfs dfs -ls /user/root/landing_zone/trips/
```

---

## Task 6: Copy File Back

**Objective:** Copy the file back from /Archive to /landing_zone/trips.

### Commands:
```bash
# Copy file back from Archive to trips directory
hdfs dfs -cp /user/root/Archive/taxi_trip_data.csv /user/root/landing_zone/trips/
```

**Verification:**
```bash
hdfs dfs -ls /user/root/landing_zone/trips/
```

---

## Task 7: Cleanup

**Objective:** Delete the /Archive directory.

### Commands:
```bash
# Delete Archive directory and all its contents
hdfs dfs -rm -r /user/root/Archive
```

**Verification:**
```bash
hdfs dfs -ls /user/root/
```

---

## Summary

At the end of Part A, the HDFS structure should contain:
- `/user/root/landing_zone/zones/` - containing taxi_zone_geo.csv
- `/user/root/landing_zone/trips/` - containing taxi_trip_data.csv
- `/user/root/landing_zone/trips/archive_copy/` - containing backup copy of taxi_trip_data.csv
- `/user/root/Archive/` - deleted (no longer exists)

## Key HDFS Commands Used:
- `hdfs dfs -mkdir -p` - Create directories
- `hdfs dfs -put` - Upload files from local to HDFS
- `hdfs dfs -ls` - List directory contents
- `hdfs dfs -du -h` - Display disk usage
- `hdfs dfs -cp` - Copy files within HDFS
- `hdfs dfs -mv` - Move/rename files within HDFS
- `hdfs dfs -rm -r` - Delete directories and files recursively

---

