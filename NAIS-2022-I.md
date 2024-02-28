# Data sources

- Office for Coastal Management, 2023: Nationwide Automatic Identification System 2022, <https://www.fisheries.noaa.gov/inport/item/67336>. GUID: gov.noaa.nmfs.inport:67336. Updated: February 6, 2023.
- [MarineCadastre.gov](https://marinecadastre.gov/)

# Schema

[Specifying a schema](https://cloud.google.com/bigquery/docs/schemas)

`MarineCadastre_schema.json`

```json
[
  {
    "description": "Maritime Mobile Service Identity value",
    "mode": "REQUIRED",
    "name": "MMSI",
    "type": "STRING"
  },
  {
    "description": "Full UTC date and time",
    "mode": "REQUIRED",
    "name": "BaseDateTime",
    "type": "TIMESTAMP"
  },
  {
    "description": "decimal degrees. Latitude",
    "mode": "REQUIRED",
    "name": "LAT",
    "type": "FLOAT"
  },
  {
    "description": "decimal degrees. Longitude",
    "mode": "REQUIRED",
    "name": "LON",
    "type": "FLOAT"
  },
  {
    "description": "knots. Speed Over Ground",
    "mode": "REQUIRED",
    "name": "SOG",
    "type": "FLOAT"
  },
  {
    "description": "degrees. Course Over Ground",
    "mode": "REQUIRED",
    "name": "COG",
    "type": "FLOAT"
  },
  {
    "description": "degrees. True heading angle",
    "mode": "REQUIRED",
    "name": "Heading",
    "type": "FLOAT"
  },
  {
    "description": "Name as shown on the station radio license",
    "mode": "NULLABLE",
    "name": "VesselName",
    "type": "STRING"
  },
  {
    "description": "International Maritime Organization Vessel number",
    "mode": "NULLABLE",
    "name": "IMO",
    "type": "STRING"
  },
  {
    "description": "Call sign as assigned by FCC",
    "mode": "NULLABLE",
    "name": "CallSign",
    "type": "STRING"
  },
  {
    "description": "Vessel type as defined in NAIS specifications",
    "mode": "NULLABLE",
    "name": "VesselType",
    "type": "STRING"
  },
  {
    "description": "Navigation status as defined by the COLREGS",
    "mode": "NULLABLE",
    "name": "Status",
    "type": "STRING"
  },
  {
    "description": "Length of vessel (see NAIS specifications)",
    "mode": "NULLABLE",
    "name": "Length",
    "type": "FLOAT"
  },
  {
    "description": "Width of vessel (see NAIS specifications)",
    "mode": "NULLABLE",
    "name": "Width",
    "type": "FLOAT"
  },
  {
    "description": "Draft depth of vessel (see NAIS specifications)",
    "mode": "NULLABLE",
    "name": "Draft",
    "type": "FLOAT"
  },
  {
    "description": "Cargo type (see NAIS specification and codes)",
    "mode": "NULLABLE",
    "name": "Cargo",
    "type": "STRING"
  },
  {
    "description": "Class of AIS transceiver",
    "mode": "REQUIRED",
    "name": "TransceiverClass",
    "type": "STRING"
  }
]
```
 
[bq command-line tool reference](https://cloud.google.com/bigquery/docs/reference/bq-cli-reference)

# bq mk

```bash
bq mk --table --schema=MarineCadastre_schema.json uscg.nais
```

# bq show

Check that schema of BigQuery table is same as schema in `MarineCadastre_schema.json`.

```bash
bq show --schema --format=prettyjson ais-data-385301:uscg.nais | diff MarineCadastre_schema.json -
```

# bq load

Manually for each month, we do (here for June)

```bash
y='2022'
m='06'
for d in {01..30}; do \
curl -O https://coast.noaa.gov/htdata/CMSP/AISDataHandler/${y}/AIS_${y}_${m}_${d}.zip; \
unzip AIS_${y}_${m}_${d}.zip; \
rm AIS_${y}_${m}_${d}.zip; \
gsutil cp AIS_${y}_${m}_${d}.csv gs://jordanbell2357marinecadastre/; \
rm AIS_${y}_${m}_${d}; \
bq load \
--source_format=CSV \
--skip_leading_rows=1 \
--max_bad_records=200 \
--schema=MarineCadastre_schema.json \
uscg.nais \
gs://jordanbell2357marinecadastre/AIS_${y}_${m}_${d}; \
gsutil rm gs://jordanbell2357marinecadastre/AIS_${y}_${m}_${d}.csv; \
done
```

# Investigating data in BigQuery

Number of records

```sql
SELECT FORMAT("%'d", COUNT(*)) FROM `ais-data-385301.uscg.nais`;
```

> 2,966,617,246

We execute this query in BigQuery console and work with the results as a Pandas dataframe:

```sql
SELECT MMSI, COUNT(MMSI) AS MMSI_count FROM `ais-data-385301.uscg.nais` GROUP BY MMSI ORDER BY COUNT(MMSI);
```

In the [Python notebook](https://github.com/jordanbell2357/uscg-nais-data/blob/main/NAIS-2022.ipynb) we execute

```python
# Running this code will display the query used to generate your previous job

job = client.get_job('bquxjob_56d440df_18decc339b8') # Job ID inserted based on the query results selected to explore
print(job.query)
```

This outputs the SQL query

```sql
SELECT MMSI, COUNT(MMSI) AS MMSI_count FROM `ais-data-385301.uscg.nais` GROUP BY MMSI ORDER BY COUNT(MMSI);
```

Then

```python
# Running this code will read results from your previous job

job = client.get_job('bquxjob_56d440df_18decc339b8') # Job ID inserted based on the query results selected to explore
results = job.to_dataframe()
```

Running

```python
results.describe()
```

displays

|index|MMSI\_count|
|---|---|
|count|85683\.0|
|mean|34623\.17199444464|
|std|75018\.4319475712|
|min|1\.0|
|25%|304\.0|
|50%|4609\.0|
|75%|27060\.0|
|max|472990\.0|

## Histograms and log transforms

```python
import numpy as np
import matplotlib.pyplot as plt

# Convert 'MMSI_count' to a 1-dimensional numpy array
mmsi_count = np.array(results['MMSI_count'])

# Create a larger figure
plt.figure(figsize=(15, 9))

# Create a histogram using matplotlib
plt.hist(mmsi_count, bins='auto', edgecolor='black', alpha=0.7)
plt.xlabel("MMSI Count")
plt.ylabel("Frequency")
plt.title("Distribution of MMSI Count")

plt.show()
```

![Histogram of distribution of MMSI Count](https://github.com/jordanbell2357/how-to/assets/47544607/bc6d9162-3927-45fe-bd8d-648523671697)


```python
import numpy as np
import matplotlib.pyplot as plt

# Convert 'MMSI_count' to a 1-dimensional numpy array
mmsi_count = np.array(results['MMSI_count'], dtype=float)  # Convert to float

# Apply log transform
log_mmsi_count = np.log(mmsi_count)

# Create a larger figure
plt.figure(figsize=(15, 9))

# Create a histogram using matplotlib
plt.hist(log_mmsi_count, bins='auto', edgecolor='black', alpha=0.7)
plt.xlabel("Log MMSI Count")
plt.ylabel("Frequency")
plt.title("Distribution of Log MMSI Count")

plt.show()
```

![Histogram of distribution of Log MMSI Count](https://github.com/jordanbell2357/how-to/assets/47544607/7a45d62e-ce91-43f4-9f64-e65065b34e06)


## KDE plots and log transforms

[seaborn.kdeplot](https://seaborn.pydata.org/generated/seaborn.kdeplot.html)

```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Convert 'MMSI_count' to a 1-dimensional numpy array
mmsi_count = np.array(results['MMSI_count'])

# Create a larger figure
plt.figure(figsize=(15, 9))

# Create a KDE plot using seaborn
sns.kdeplot(mmsi_count, fill=True)
plt.xlabel("MMSI Count")
plt.ylabel("Density")
plt.title("Distribution of MMSI Count")
plt.show()
```

![KDE plot of distribution of MMSI Count](https://github.com/jordanbell2357/how-to/assets/47544607/4ba279fa-2fba-4341-83d0-43a30eb84aa6)


```python
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Convert 'MMSI_count' to a 1-dimensional numpy array
mmsi_count = np.array(results['MMSI_count'], dtype=float)

# Apply log transform
log_mmsi_count = np.log(mmsi_count)

# Create a larger figure
plt.figure(figsize=(15, 9))

# Create a KDE plot using seaborn
sns.kdeplot(log_mmsi_count, fill=True)
plt.xlabel("Log MMSI Count")
plt.ylabel("Density")
plt.title("Distribution of Log MMSI Count")
plt.show()
```

![KDE plot of distribution of Log MMSI Count](https://github.com/jordanbell2357/how-to/assets/47544607/e36448dc-dcef-4e60-a9b0-7e2a6472f78f)
