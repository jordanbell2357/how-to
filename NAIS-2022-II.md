# Removing duplicate entries for MMSI, BaseDateTime pairs

Removing duplicate entries for MMSI, BaseDateTime pairs

```sql
CREATE TABLE `ais-data-385301.uscg.no_dups` AS
SELECT 
    *
FROM 
    (
        SELECT 
            *, 
            ROW_NUMBER() OVER(PARTITION BY MMSI, BaseDateTime ORDER BY MMSI) AS row_num 
        FROM 
            `ais-data-385301.uscg.nais`
    )
WHERE 
    row_num = 1;
```

Count

```sql
SELECT FORMAT("%'d", COUNT(*)) FROM `ais-data-385301.uscg.no_dups`;
```

> 2,965,744,033

# Parsing geography

```sql
ALTER TABLE `ais-data-385301.uscg.no_dups`
ADD COLUMN vessel_geography GEOGRAPHY;

UPDATE `ais-data-385301.uscg.no_dups`
SET vessel_geography = ST_GEOGPOINT(LON, LAT)
WHERE LON IS NOT NULL AND LAT IS NOT NULL;
```

# Ports

- [USACE Principal Ports List](https://www.iwr.usace.army.mil/About/Technical-Centers/WCSC-Waterborne-Commerce-Statistics-Center-2/WCSC-Navigation-Facilities/)

- [Principal Ports | U.S. Army Corps of Engineers Geospatial](https://geospatial-usace.opendata.arcgis.com/datasets/8eb8a75c67e84c22af7acf4268692052_0/about)

Download `Principal_Ports.csv`. There are 150 records.

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


