# Ports

- [USACE Principal Ports List](https://www.iwr.usace.army.mil/About/Technical-Centers/WCSC-Waterborne-Commerce-Statistics-Center-2/WCSC-Navigation-Facilities/)

- [Principal Ports | U.S. Army Corps of Engineers Geospatial](https://geospatial-usace.opendata.arcgis.com/datasets/8eb8a75c67e84c22af7acf4268692052_0/about)

Download `Principal_Ports.csv`. There are 150 records. It head is

```csv
﻿X,Y,FID,ID,PORT,TYPE,RANK,PORT_NAME,TOTAL,DOMESTIC,FOREIGN_,IMPORTS,EXPORTS
-73.74816,42.64271,1,1,505,C,78,"Albany Port District, NY",4577888,4049958,527930,421419,106511
-89.18481,37.0403,2,2,2308,I,148,"Alexandria-Cario Port, IL",1168613,1168613,0,0,0
-83.42173,45.06197,3,3,3617,L,110,"Alpena, MI",2360344,2360344,0,0,0
-90.10405,38.8419,4,4,2309,I,57,"America's Central Port, IL",8409663,8409663,0,0,0
-122.59961,48.49594,5,5,4730,C,50,"Anacortes, WA",9612351,7064800,2547551,1759229,788322
-80.79398,41.90369,6,6,3219,L,136,"Ashtabula Port Authority, OH",1603644,1118432,485212,418370,66842
-76.56164,39.25083,7,7,700,C,17,"Baltimore, MD",37439579,4469775,32969804,13605961,19363843
-94.09499,30.08487,8,8,2393,C,7,"Beaumont, TX",74555488,18238925,56316563,16355729,39960834
-71.05229,42.35094,9,9,149,C,40,"Boston, MA",13308532,3132842,10175690,8885672,1290018
```

Create table from this with autodetect schema `ais-data-385301.usace.principal-ports`.

This is the schema

```json
[
  {
    "name": "X",
    "mode": "NULLABLE",
    "type": "FLOAT",
    "description": null,
    "fields": []
  },
  {
    "name": "Y",
    "mode": "NULLABLE",
    "type": "FLOAT",
    "description": null,
    "fields": []
  },
  {
    "name": "FID",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "ID",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "PORT",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "TYPE",
    "mode": "NULLABLE",
    "type": "STRING",
    "description": null,
    "fields": []
  },
  {
    "name": "RANK",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "PORT_NAME",
    "mode": "NULLABLE",
    "type": "STRING",
    "description": null,
    "fields": []
  },
  {
    "name": "TOTAL",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "DOMESTIC",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "FOREIGN_",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "IMPORTS",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  },
  {
    "name": "EXPORTS",
    "mode": "NULLABLE",
    "type": "INTEGER",
    "description": null,
    "fields": []
  }
]
```

Then

```sql
ALTER TABLE `ais-data-385301.usace.principal-ports`
ADD COLUMN port_geography GEOGRAPHY;

UPDATE `ais-data-385301.usace.principal-ports`
SET port_geography = ST_GEOGPOINT(X, Y)
WHERE X IS NOT NULL AND Y IS NOT NULL;
```

# Further manipulation of AIS data

```sql
CREATE TABLE `ais-data-385301.uscg.no_dups_simplified` AS
SELECT 
    MMSI, 
    BaseDateTime, 
    vessel_geography
FROM 
    `ais-data-385301.uscg.no_dups`;
```

Timestamp differences

```sql
CREATE TABLE `ais-data-385301.uscg.timestamp_diff` AS
SELECT
    MMSI,
    BaseDateTime,
    vessel_geography,
    TIMESTAMP_DIFF(
        BaseDateTime, 
        LAG(BaseDateTime) OVER (PARTITION BY MMSI ORDER BY BaseDateTime), 
        MINUTE
    ) AS time_diff_minutes
FROM 
    `ais-data-385301.uscg.no_dups_simplified`;
```

# Join vessel table and port table

Create table of maximal visits to 20km radius disc around ports.

```sql
CREATE TABLE `ais-data-385301.uscg.principal_ports_max_stay` AS
WITH time_at_port AS (
    SELECT
        v.MMSI,
        v.BaseDateTime,
        p.ID,
        v.time_diff_minutes
    FROM 
        `ais-data-385301.uscg.timestamp_diff` AS v
    JOIN
        `ais-data-385301.usace.principal-ports` AS p
    ON
        ST_DWithin(v.vessel_geography, p.port_geography, 20000) -- distance within 20km
    WHERE
        v.time_diff_minutes >= 180  -- time difference is at least 3 hours
),
ordered_stays AS (
    SELECT *,
        LAG(ID) OVER (PARTITION BY MMSI ORDER BY BaseDateTime) AS last_port,
        LEAD(ID) OVER (PARTITION BY MMSI ORDER BY BaseDateTime) AS next_port
    FROM time_at_port
),
continuous_stays AS (
    SELECT *,
        SUM(CASE WHEN ID = last_port THEN 0 ELSE 1 END) OVER (PARTITION BY MMSI ORDER BY BaseDateTime) AS stay_group
    FROM ordered_stays
),
grouped_stays AS (
    SELECT 
        MMSI,
        ID,
        MIN(BaseDateTime) AS stay_start,
        MAX(BaseDateTime) AS stay_end,
        SUM(time_diff_minutes) AS total_duration
    FROM continuous_stays
    GROUP BY MMSI, ID, stay_group
)
SELECT *
FROM grouped_stays
WHERE total_duration >= 180  -- time difference is at least 3 hours
ORDER BY MMSI, stay_start;
```

Count monthly visits

```sql
CREATE TABLE `ais-data-385301.uscg.usace_principal_ports_monthly_visits` AS
WITH port_monthly_visits AS (
    SELECT 
        ID, 
        FORMAT_TIMESTAMP('%Y-%m', stay_start) AS month, 
        COUNT(MMSI) AS visit_Count
    FROM 
        `ais-data-385301.uscg.principal_ports_max_stay`
    GROUP BY 
        ID, Month
)
SELECT 
    P.ID,
    V.Month,
    V.visit_Count,
    P.PORT_NAME,
    P.port_geography
FROM 
    port_monthly_visits AS V
JOIN
    `ais-data-385301.usace.principal-ports` AS P
ON
    V.ID = P.ID
ORDER BY 
    V.ID, V.month;
```

[usace_principal_ports_monthly_visits.csv](https://docs.google.com/spreadsheets/d/e/2PACX-1vTubVzVNt76z3St9N7RPM_e6v5xDWaMqR17ql0XnDMz-SN2QCzxAtwJ7LTg7livY-nEwsDnHtbE4ZKd/pub?gid=798031662&single=true&output=csv)

[Google Sheets view](https://docs.google.com/spreadsheets/d/e/2PACX-1vTubVzVNt76z3St9N7RPM_e6v5xDWaMqR17ql0XnDMz-SN2QCzxAtwJ7LTg7livY-nEwsDnHtbE4ZKd/pubhtml?gid=951681385&single=true)

# Comparing to sources of truth

We now have to compare our estimates for monthly port visits with sources of truth. The two parameters for measuring continuous visits to a port are radius of port and minimum time spent within that radius.

After comparing to sources of truth, there is next a cycle of adjusting parameters and comparing results to sources of truth.
