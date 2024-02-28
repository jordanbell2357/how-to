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
