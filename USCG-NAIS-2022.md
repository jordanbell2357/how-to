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

# Trajectories of particular vessels

## MMSI 368174410

```python
job = client.get_job('bquxjob_6e4920c7_18877f73b54') # Job ID inserted based on the query results selected to explore
print(job.query)
```

```sql
SELECT * FROM `ais-data-385301.uscg.nais` WHERE MMSI = '368174410' ORDER BY BaseDateTime ASC;
```

```python
# Running this code will read results from your previous job

job = client.get_job('bquxjob_6e4920c7_18877f73b54') # Job ID inserted based on the query results selected to explore
results = job.to_dataframe()

results.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 160025 entries, 0 to 160024
Data columns (total 17 columns):
 #   Column            Non-Null Count   Dtype              
---  ------            --------------   -----              
 0   MMSI              160025 non-null  object             
 1   BaseDateTime      160025 non-null  datetime64[ns, UTC]
 2   LAT               160025 non-null  float64            
 3   LON               160025 non-null  float64            
 4   SOG               160025 non-null  float64            
 5   COG               160025 non-null  float64            
 6   Heading           160025 non-null  float64            
 7   VesselName        160025 non-null  object             
 8   IMO               160025 non-null  object             
 9   CallSign          160025 non-null  object             
 10  VesselType        160025 non-null  object             
 11  Status            160025 non-null  object             
 12  Length            160025 non-null  float64            
 13  Width             160025 non-null  float64            
 14  Draft             160025 non-null  float64            
 15  Cargo             160025 non-null  object             
 16  TransceiverClass  160025 non-null  object             
dtypes: datetime64[ns, UTC](1), float64(8), object(8)
memory usage: 20.8+ MB
```

```python
import pandas as pd
import matplotlib.pyplot as plt

# Parameters
start_date = '2022-06-01 00:00:00'  # Include the desired hour
end_date = '2022-06-01 23:00:00'  # Include the desired hour
sampling_rate = '30min'  # Options: '1min', '5min', '10min', '15min', '30min', '1H', '2H', '3H', ...

# Define the DataFrame with desired columns
df = results.copy()

# Set the 'BaseDateTime' column as the DataFrame index
df.set_index('BaseDateTime', inplace=True)

# Apply time cutoff
df = df.loc[start_date:end_date]

# Resample the data to the specified sampling rate
sampled_data = df.resample(sampling_rate).first()

# Interpolate the data for smooth trajectories
interpolated_data = sampled_data.interpolate(method='linear')

# Create a plot of the interpolated trajectories
plt.plot(
    interpolated_data['LON'],
    interpolated_data['LAT'],
    '-',
    linewidth=1
)

# Add a dot at the start of the trajectory
plt.plot(
    interpolated_data['LON'].iloc[0],
    interpolated_data['LAT'].iloc[0],
    'bo'
)

# Add styling with arrows indicating the direction of time
dx = interpolated_data['LON'].diff().shift(-1)
dy = interpolated_data['LAT'].diff().shift(-1)
plt.quiver(
    interpolated_data['LON'],
    interpolated_data['LAT'],
    dx,
    dy,
    angles='xy',
    scale_units='xy',
    scale=1,
    width=0.004,  # Increase the width to make the arrows wider
    color='black'
)

# Customize the plot
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.title('Interpolated Trajectories - Sampled Data ({} to {})'.format(start_date, end_date))

plt.show()
```

![Interpolated trajectory for MMSI 368174410](https://github.com/jordanbell2357/how-to/assets/47544607/11f5ee41-e33f-4c5b-a2c0-2fee8ccd4880)

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from scipy.interpolate import interp1d

df = results.copy()

# Set start and end timestamps for the analysis
start_date = pd.Timestamp('2022-06-01 00:00:00')  # Include the desired hour
end_date = pd.Timestamp('2022-09-30 23:00:00')  # Include the desired hour

sampling_rate = '15min'  # Options: '1min', '5min', '10min', '15min', '30min', '1H', '2H', '3H', ...

# Define the DataFrame with desired columns

# Set the 'BaseDateTime' column as the DataFrame index
df.set_index('BaseDateTime', inplace=True)

# Convert to timezone naive UTC
df.index = df.index.tz_convert('UTC').tz_localize(None)

# Apply time cutoff
df = df.loc[start_date:end_date]

# Resample the data to the specified sampling rate
sampled_data = df.resample(sampling_rate).first()

# Interpolate the data for smooth trajectories
interpolated_data = sampled_data.interpolate(method='linear')

# Convert datetime to numerical values
time_vals = (interpolated_data.index - pd.Timestamp("1970-01-01")) // pd.Timedelta('1s')

# Create an interpolation function for the latitude and longitude coordinates
interp_func = interp1d(
    time_vals,
    interpolated_data[['LAT', 'LON']],
    kind='linear',
    axis=0,
    fill_value='extrapolate'
)

# Generate new time points for finer sampling
new_time_vals = np.arange(time_vals.min(), time_vals.max(), 60)  # 60 seconds for 1-minute frequency

# Interpolate the latitude and longitude coordinates at the new time points
new_coords = interp_func(new_time_vals)

# Convert back to datetime format
new_time = pd.to_datetime(new_time_vals, unit='s')

# Create a scatter plot of the interpolated data
plt.scatter(
    new_coords[:, 1],
    new_coords[:, 0],
    c='red',
    marker='o',
    label='Trajectory Points'
)

# Create a line plot of the interpolated trajectory
plt.plot(
    new_coords[:, 1],
    new_coords[:, 0],
    '-',
    linewidth=1,
    color='blue',
    label='Interpolated Trajectory'
)

# Add a dot at the start of the trajectory
plt.plot(
    new_coords[0, 1],
    new_coords[0, 0],
    'bo',
    label='Start Point'
)

# Compute difference for arrow directions
d_coords = np.diff(new_coords, axis=0)

# Add styling with arrows indicating the direction of time
plt.quiver(
    new_coords[:-1, 1],
    new_coords[:-1, 0],
    d_coords[:, 1],
    d_coords[:, 0],
    angles='xy',
    scale_units='xy',
    scale=1,
    width=0.004,  # Increase the width to make the arrows wider
    color='black'
)

# Customize the plot
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.title('Interpolated Trajectories - Sampled Data ({} to {})'.format(start_date, end_date))
plt.legend()

plt.show()
```

![Interpolated trajectory for MMSI 368174410](https://github.com/jordanbell2357/how-to/assets/47544607/1b0d59e2-3dda-457d-97b2-07ea9fb34148)

```python
import h3
import geopandas as gpd
from shapely.geometry import Polygon
import numpy as np

df = results.copy()

# h3
resolution_parameter = 6

# Set start and end timestamps for the analysis
start_date = pd.Timestamp('2022-06-01 00:00:00')  # Include the desired hour
end_date = pd.Timestamp('2022-09-30 23:00:00')  # Include the desired hour

df['BaseDateTime'] = pd.to_datetime(df['BaseDateTime']).dt.tz_convert('UTC').dt.tz_localize(None)

# Apply time cutoff
df_filtered = df[(df['BaseDateTime'] >= start_date) & (df['BaseDateTime'] <= end_date)].copy()

# Convert latitude and longitude to H3 grid indices
df_filtered['H3Index'] = df_filtered.apply(lambda row: h3.geo_to_h3(row['LAT'], row['LON'], resolution=resolution_parameter), axis=1)

# Calculate the time spent in each hexagon based on distances between consecutive timestamps
hexagon_data = []
for hex_id, hex_group in df_filtered.groupby('H3Index'):
    hex_group = hex_group.sort_values('BaseDateTime')
    hex_group['TimeDiff'] = hex_group['BaseDateTime'].diff().fillna(pd.Timedelta(seconds=0))
    hex_group['TimeDiff'] = hex_group['TimeDiff'].apply(lambda x: x.total_seconds())
    total_time = np.sum(hex_group['TimeDiff'])
    hexagon_data.append({'H3Index': hex_id, 'Time': total_time})

# Create a GeoDataFrame with the time spent data
hexagon_data = pd.DataFrame(hexagon_data)
geometry = hexagon_data['H3Index'].apply(lambda h: Polygon(h3.h3_to_geo_boundary(h, geo_json=False)))
density_data = gpd.GeoDataFrame(hexagon_data, geometry=geometry)

# Visualize the density map
density_data.plot(column='Time', cmap='hot_r', legend=True)
```

![Time spent by MMSI 368174410 in hexagonal cells](https://github.com/jordanbell2357/how-to/assets/47544607/e6210bc3-a509-4e39-b44b-310f666a5bc5)

The next method uses Speed Over Ground (SOG).

```python
import h3
import geopandas as gpd
from shapely.geometry import Polygon
import numpy as np

df = results.copy()

# h3
resolution_parameter = 6

# Set start and end timestamps for the analysis
start_date = pd.Timestamp('2022-06-01 00:00:00')  # Include the desired hour
end_date = pd.Timestamp('2022-09-30 23:00:00')  # Include the desired hour

df['BaseDateTime'] = pd.to_datetime(df['BaseDateTime']).dt.tz_convert('UTC').dt.tz_localize(None)

# Apply time cutoff
df_filtered = df[(df['BaseDateTime'] >= start_date) & (df['BaseDateTime'] <= end_date)].copy()

# Convert latitude and longitude to H3 grid indices
df_filtered['H3Index'] = df_filtered.apply(lambda row: h3.geo_to_h3(row['LAT'], row['LON'], resolution=resolution_parameter), axis=1)

# Calculate the duration weighted mean SOG in each hexagon
hexagon_data = []
for hex_id, hex_group in df_filtered.groupby('H3Index'):
    hex_group = hex_group.sort_values('BaseDateTime')
    hex_group['TimeDiff'] = hex_group['BaseDateTime'].diff().fillna(pd.Timedelta(seconds=0))
    hex_group['TimeDiff'] = hex_group['TimeDiff'].apply(lambda x: x.total_seconds())
    total_duration = np.sum(hex_group['TimeDiff'])
    weighted_mean_sog = np.average(hex_group['SOG'], weights=hex_group['TimeDiff'])  # Duration weighted mean SOG
    hexagon_data.append({'H3Index': hex_id, 'WeightedMeanSOG': weighted_mean_sog, 'TotalDuration': total_duration})

# Create a GeoDataFrame with the weighted mean SOG and total duration data
hexagon_data = pd.DataFrame(hexagon_data)
geometry = hexagon_data['H3Index'].apply(lambda h: Polygon(h3.h3_to_geo_boundary(h, geo_json=False)))
density_data = gpd.GeoDataFrame(hexagon_data, geometry=geometry)

# Visualize the density map
density_data.plot(column='WeightedMeanSOG', cmap='hot_r', legend=True)
```

![Duration weighted mean SOG for MMSI 368174410 in hexagonal cells](https://github.com/jordanbell2357/how-to/assets/47544607/584316f2-d391-4bd8-a77a-29dc800f3a4a)

## MMSI 219155000

```python
# Running this code will display the query used to generate your previous job

job = client.get_job('bquxjob_26f1c9e6_188746623e6') # Job ID inserted based on the query results selected to explore
print(job.query)
```

```sql
SELECT * FROM `ais-data-385301.uscg.nais` WHERE MMSI = '219155000' ORDER BY BaseDateTime ASC;
```

```python
# Running this code will read results from your previous job

job = client.get_job('bquxjob_26f1c9e6_188746623e6') # Job ID inserted based on the query results selected to explore
results = job.to_dataframe()

results.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 18973 entries, 0 to 18972
Data columns (total 17 columns):
 #   Column            Non-Null Count  Dtype              
---  ------            --------------  -----              
 0   MMSI              18973 non-null  object             
 1   BaseDateTime      18973 non-null  datetime64[ns, UTC]
 2   LAT               18973 non-null  float64            
 3   LON               18973 non-null  float64            
 4   SOG               18973 non-null  float64            
 5   COG               18973 non-null  float64            
 6   Heading           18973 non-null  float64            
 7   VesselName        18973 non-null  object             
 8   IMO               18973 non-null  object             
 9   CallSign          18973 non-null  object             
 10  VesselType        18973 non-null  object             
 11  Status            18973 non-null  object             
 12  Length            18973 non-null  float64            
 13  Width             18973 non-null  float64            
 14  Draft             18973 non-null  float64            
 15  Cargo             18973 non-null  object             
 16  TransceiverClass  18973 non-null  object             
dtypes: datetime64[ns, UTC](1), float64(8), object(8)
memory usage: 2.5+ MB
```

```python
import pandas as pd
import matplotlib.pyplot as plt

# Parameters
start_date = pd.Timestamp('2022-06-01')
end_date = pd.Timestamp('2022-07-31')
interval_length = '1H'  # Options: '1min', '5min', '10min', '15min', '30min', '1H', '2H', '3H', ...

df = results[['MMSI', 'BaseDateTime', 'LAT', 'LON', 'SOG', 'COG', 'Heading']].copy()
df['BaseDateTime'] = df['BaseDateTime'].dt.tz_convert('UTC').dt.tz_localize(None)
df.set_index('BaseDateTime', inplace=True)

# Calculate the time differences in seconds
df['TimeDifference'] = df.index.to_series().diff().dt.total_seconds()

# Remove the first row (NaN value for 'TimeDifference')
df = df.iloc[1:]

# Apply date range filter
df_filtered = df.loc[start_date:end_date]

if df_filtered.empty:
    print("No data points between '2022-06-01' and '2022-08-31'")
else:
    # Count messages by each interval
    message_counts_by_interval = df_filtered.resample(interval_length).count()['MMSI']

    # Plot the result
    plt.figure(figsize=(10, 6))
    message_counts_by_interval.plot()
    plt.xlabel('Time')
    plt.ylabel('Number of Messages')
    plt.title(f'Distribution of Messages Every {interval_length}')
    plt.grid(True)
    plt.show()
```

![Distribution of Messages Every 1H](https://github.com/jordanbell2357/how-to/assets/47544607/d52f3d1b-4d4e-4c57-a658-7fd444fe2ef4)

```python
import matplotlib.pyplot as plt

results['SOG'].plot(kind='line')
plt.title('Speed Over Ground (SOG) Over Time')
plt.xlabel('Time')
plt.ylabel('SOG (knots)')
plt.show()
```

![Speed Over Ground (SOG) Over Time](https://github.com/jordanbell2357/how-to/assets/47544607/13723a34-181c-4436-97f2-cdaf4b86c7f0)

### Kalman filtering

```python
from pykalman import KalmanFilter
import numpy as np
import matplotlib.pyplot as plt

# Set start and end timestamps for the analysis
start_date = pd.Timestamp('2022-06-01 00:00:00')  # Include the desired hour
end_date = pd.Timestamp('2022-09-30 23:00:00')  # Include the desired hour

df = results[['MMSI', 'BaseDateTime', 'LAT', 'LON', 'SOG', 'COG', 'Heading']].copy()
df['BaseDateTime'] = df['BaseDateTime'].dt.tz_convert('UTC').dt.tz_localize(None)

# Apply time cutoff
df_filtered = df[(df['BaseDateTime'] >= start_date) & (df['BaseDateTime'] <= end_date)]

# Set BaseDateTime as index for filtered dataframe
df_filtered.set_index('BaseDateTime', inplace=True)

# Convert DataFrame to numpy array
coordinates = df_filtered[['LAT', 'LON']].to_numpy()

# Specify initial state mean and covariance
initial_state_mean = [coordinates[0, 0],
                      0,
                      coordinates[0, 1],
                      0]

initial_state_covariance = [[1, 0, 0, 0], 
                            [0, 1, 0, 0],
                            [0, 0, 1, 0],
                            [0, 0, 0, 1]]

# Create a Kalman Filter instance
kf = KalmanFilter(transition_matrices=[[1, 1, 0, 0], 
                                        [0, 1, 0, 0],
                                        [0, 0, 1, 1],
                                        [0, 0, 0, 1]],
                   observation_matrices=[[1, 0, 0, 0], 
                                         [0, 0, 1, 0]],
                   initial_state_mean=initial_state_mean,
                   initial_state_covariance=initial_state_covariance)

# Perform the Kalman filtering
mean_states, covariances = kf.filter(coordinates)

# Visualize the results
plt.figure(figsize=(16, 8))
plt.scatter(coordinates[:, 1], coordinates[:, 0], marker='x', color='b', label='observations')
plt.plot(mean_states[:, 2], mean_states[:, 0], linestyle='-', marker='o', color='r', label='Kalman filter')
plt.legend()
plt.title('Kalman filtering of location data')
plt.xlabel('Longitude')
plt.ylabel('Latitude')
plt.show()
```

![Kalman filtering of location data](https://github.com/jordanbell2357/how-to/assets/47544607/e818ed54-ab7b-4fdc-ba75-1438e90acda5)

### MovingPandas

```python
import movingpandas as mpd
from shapely.geometry import Point
import geopandas as gpd
import pandas as pd

# Set start and end timestamps for the analysis
start_date = pd.Timestamp('2022-06-01 00:00:00')  # Include the desired hour
end_date = pd.Timestamp('2022-09-30 23:00:00')  # Include the desired hour

df = results[['MMSI', 'BaseDateTime', 'LAT', 'LON', 'SOG', 'COG', 'Heading']].copy()
df['BaseDateTime'] = df['BaseDateTime'].dt.tz_convert('UTC').dt.tz_localize(None)

# Apply time cutoff
df_filtered = df[(df['BaseDateTime'] >= start_date) & (df['BaseDateTime'] <= end_date)]

# Set BaseDateTime as index for filtered dataframe
df_filtered.set_index('BaseDateTime', inplace=True)

#df = results[['MMSI', 'BaseDateTime', 'LAT', 'LON', 'SOG', 'COG', 'Heading']].copy()
#df.set_index('BaseDateTime', inplace=True)

# Convert your DataFrame to a GeoDataFrame
gdf = gpd.GeoDataFrame(df_filtered, geometry=gpd.points_from_xy(df_filtered.LON, df_filtered.LAT), crs="EPSG:4326")

# Create a TrajectoryCollection from the GeoDataFrame
traj_collection = mpd.TrajectoryCollection(gdf, 'MMSI')

# You can adjust these parameters as needed:
MIN_LENGTH = 1000  # meters
MIN_DURATION = pd.Timedelta(minutes=60)  # minimum duration of a trip
SPEED = 2  # speed threshold for being considered stationary, in knots

# Segment trajectories into trips
trips = []
for traj in traj_collection:
    splitter = mpd.SpeedSplitter(traj)
    segments = splitter.split(speed=SPEED, duration=MIN_DURATION)

    # Remove short segments
    long_segments = [segment for segment in segments if segment.get_length() > MIN_LENGTH]

    trips.extend(long_segments)

import matplotlib.pyplot as plt

# Sort trips by duration in descending order
trips.sort(key=lambda x: x.get_duration(), reverse=True)

# Select the top 5 longest duration trips
top_5_trips = trips[:5]

# Create a figure and axes
fig, ax = plt.subplots(figsize=(16, 8))

# Assign colors to trajectories
colors = ['red', 'blue', 'green', 'orange', 'purple']

# Plot each trajectory with a different color and add legend
for i, trip in enumerate(top_5_trips):
    color = colors[i]
    trip.plot(ax=ax, color=color, label=f"Trajectory {i+1} ({trip.get_start_time().strftime('%Y-%m-%d %H:%M:%S')})")

# Set plot title and labels
ax.set_title("Five Trajectories with Longest Duration")
ax.set_xlabel("Longitude")
ax.set_ylabel("Latitude")

# Add legend
ax.legend()

# Show the plot
plt.show()
```

![Five Trajectories with Longest Duration](https://github.com/jordanbell2357/how-to/assets/47544607/5af61fff-4e6a-46c7-a0d0-c16976632698)

# Distribution of records

## Histograms and log transforms for number of records by MMSI

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


## KDE plots and log transforms for number of records by MMSI

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

# Duplicate entries for timestamps

For some purposes we want at most one record for a particular time and vessel. This step is not necessary for the visualization we make below, but is necessary once we calculate vessel activity.

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

# BigQuery geographical data

https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions

```sql
ALTER TABLE `ais-data-385301.uscg.no_dups`
ADD COLUMN vessel_geography GEOGRAPHY;

UPDATE `ais-data-385301.uscg.no_dups`
SET vessel_geography = ST_GEOGPOINT(LON, LAT)
WHERE LON IS NOT NULL AND LAT IS NOT NULL;
```

# no_dups_simplified

```sql
CREATE TABLE `ais-data-385301.uscg.no_dups_simplified` AS
SELECT MMSI, BaseDateTime, vessel_geography
FROM `ais-data-385301.uscg.no_dups`;
```

# jslibs.h3 UDF for BigQuery

https://github.com/dtws/bigquery-jslibs

UDF `jslibs.h3.ST_H3`

```sql
DECLARE resolution INT64 DEFAULT 4;

-- Create a new table with daily H3 density data
CREATE TABLE `ais-data-385301.uscg.h3_resolution4_daily_density` AS
WITH 
  -- Filter locations based on the bounding box
  filtered_locations AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      vessel_geography,
      DATE(BaseDateTime) AS DateOnly  -- Extract the date component
    FROM `ais-data-385301.uscg.no_dups_simplified`
    WHERE ST_WITHIN(vessel_geography, ST_GEOGFROMTEXT('POLYGON((-140 10, -50 10, -50 60, -140 60, -140 10))'))
  ),
  -- Create H3 index for each point
  h3_indexed AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      DateOnly,
      jslibs.h3.ST_H3(vessel_geography, resolution) AS h3_index
    FROM filtered_locations
  ),
  -- Count occurrences of each H3 index per day
  daily_density AS (
    SELECT 
      h3_index, 
      DateOnly,
      COUNT(*) AS count
    FROM h3_indexed
    GROUP BY h3_index, DateOnly
  ),
  -- Generate H3 polygons
  h3_polygons AS (
    SELECT 
      h3_index, 
      DateOnly,
      count,
      jslibs.h3.ST_H3_BOUNDARY(h3_index) AS h3_polygon
    FROM daily_density
  )
SELECT * FROM h3_polygons;
```

# h3 visualization with Python

```python
# Running this code will display the query used to generate your previous job

job = client.get_job('bquxjob_7e97f84d_18e0becac51') # Job ID inserted based on the query results selected to explore
print(job.query)
```

```sql
SELECT * FROM `ais-data-385301.uscg.h3_resolution4_daily_density`;
```

```python
# Running this code will read results from your previous job

job = client.get_job('bquxjob_7e97f84d_18e0becac51') # Job ID inserted based on the query results selected to explore
results = job.to_dataframe()
results.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 979955 entries, 0 to 979954
Data columns (total 4 columns):
 #   Column      Non-Null Count   Dtype 
---  ------      --------------   ----- 
 0   h3_index    979955 non-null  object
 1   DateOnly    979955 non-null  dbdate
 2   count       979955 non-null  Int64 
 3   h3_polygon  979955 non-null  object
dtypes: Int64(1), dbdate(1), object(2)
memory usage: 30.8+ MB
```

```python
import numpy as np
import geopandas as gpd
import matplotlib.pyplot as plt
from shapely.geometry import Polygon
import h3
import matplotlib.colors as colors
import pandas as pd

def plot_geo_data_on_date(date, results):
    # Define the lat/lon boundaries
    min_lat, max_lat, min_lon, max_lon = 10, 60, -140, -50

    # Calculate aspect ratio
    width = max_lon - min_lon
    height = max_lat - min_lat
    aspect_ratio = width / height

    # Set the figure size based on the aspect ratio
    fig_width = 10
    fig_height = fig_width / aspect_ratio

    # Filter the DataFrame for the specified date
    filtered_results = results[results['DateOnly'] == pd.to_datetime(date)].copy()

    # Create polygons for each unique hex_id
    hex_polygons = []
    for hex_id in filtered_results['h3_index'].unique():
        hex_boundary = h3.h3_to_geo_boundary(hex_id)
        polygon = Polygon([(lon, lat) for lat, lon in hex_boundary])
        centroid = polygon.centroid
        if min_lat <= centroid.y <= max_lat and min_lon <= centroid.x <= max_lon:
            hex_polygons.append(polygon)

    # Replace inf with max non-inf value and NaN with 1
    filtered_results.loc[filtered_results['count'] == np.inf, 'count'] = filtered_results.loc[filtered_results['count'] != np.inf, 'count'].max()
    filtered_results['count'] = filtered_results['count'].replace({0: 1}).fillna(1)

    # Create a GeoDataFrame
    gdf = gpd.GeoDataFrame(filtered_results, geometry=hex_polygons)

    # Convert 'count' column to float and fill NaN with 1
    gdf['count'] = gdf['count'].astype(float).fillna(1)

    # Calculate min and max values for 'count', setting count_min to 1 if it's 0
    count_min = max(gdf['count'].min(), 1)
    count_max = gdf['count'].max()

    # Plot the choropleth map
    fig, ax = plt.subplots(1, 1, figsize=(fig_width, fig_height))
    gdf.plot(column='count', cmap='YlOrRd', norm=colors.LogNorm(vmin=count_min, vmax=count_max), 
             missing_kwds={'color': 'lightgrey'}, ax=ax)
    plt.xlim(min_lon, max_lon)
    plt.ylim(min_lat, max_lat)
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.title(f'Vessel density map for {date}')
    plt.savefig(f'ais-{date}.png')
    plt.close()

plot_geo_data_on_date('2022-01-01', results)
```

![Vessel density map for 2022-01-01](https://github.com/jordanbell2357/how-to/assets/47544607/b9ece416-b421-480e-9014-5b945bf3fa90)

```python
from datetime import datetime, timedelta

# Function to generate all dates in 2022
def generate_dates_for_2022():
    start_date = datetime(2022, 1, 1)
    end_date = datetime(2022, 12, 31)
    total_days = (end_date - start_date).days + 1
    return [start_date + timedelta(days=x) for x in range(total_days)]

# Function to plot data for each day in 2022
def plot_data_for_each_day(results):
    for date in generate_dates_for_2022():
        formatted_date = date.strftime("%Y-%m-%d")
        print(f"Plotting for {formatted_date}...")
        plot_geo_data_on_date(formatted_date, results)
        print(f"Plotting completed for {formatted_date}")

plt.ioff()
plot_data_for_each_day(results)
```

```python
import zipfile
import os
import glob

# Function to zip all PNG files
def zip_png_files(zip_name):
    with zipfile.ZipFile(zip_name, 'w', zipfile.ZIP_DEFLATED) as zipf:
        for file in glob.glob('ais-*.png'):
            zipf.write(file)
            print(f"Added {file} to the zip archive.")

# Create a zip file named 'ais-2022.zip'
zip_png_files('ais-2022.zip')

print("All PNG files have been zipped successfully.")
```

```python
from google.colab import files
files.download('ais-2022.zip')
```

```bash
ffmpeg -framerate 5 -pattern_type glob -i 'ais-2022-*.png' -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" -c:v libx264 -r 30 -pix_fmt yuv420p ais-2022.mp4
```

```bash
ffmpeg -i ais-2022.mp4 -vf "fps=3,scale=1000:-1:flags=lanczos" -c:v gif ais-2022.gif
```

![Vessel density map animation for each day of 2022](ais-2022.gif)

# U.S. Army Corps of Engineers (USACE) Principal Ports

[Waterborne Commerce Statistics Center (WCSC)](https://www.iwr.usace.army.mil/About/Technical-Centers/WCSC-Waterborne-Commerce-Statistics-Center-2/)

[Principal Ports on U.S. Army Corps of Engineers Geospatial](https://geospatial-usace.opendata.arcgis.com/datasets/8eb8a75c67e84c22af7acf4268692052_0/about)

`Principal_Ports.csv`

```bash
head -n 5 Principal_Ports.csv
```

```csv
﻿X,Y,FID,ID,PORT,TYPE,RANK,PORT_NAME,TOTAL,DOMESTIC,FOREIGN_,IMPORTS,EXPORTS
-73.74816,42.64271,1,1,505,C,78,"Albany Port District, NY",4577888,4049958,527930,421419,106511
-89.18481,37.0403,2,2,2308,I,148,"Alexandria-Cario Port, IL",1168613,1168613,0,0,0
-83.42173,45.06197,3,3,3617,L,110,"Alpena, MI",2360344,2360344,0,0,0
-90.10405,38.8419,4,4,2309,I,57,"America's Central Port, IL",8409663,8409663,0,0,0
```

We import into BigQuery as table `ais-data-385301.usace.principal-ports` with an autodetected schema. We then add a geography column thus

```sql
ALTER TABLE `ais-data-385301.usace.principal-ports`
ADD COLUMN port_geography GEOGRAPHY;

UPDATE `ais-data-385301.usace.principal-ports`
SET port_geography = ST_GEOGPOINT(X, Y)
WHERE X IS NOT NULL AND Y IS NOT NULL;
```

The schema of `ais-data-385301.usace.principal-ports` is then

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
  },
  {
    "name": "port_geography",
    "mode": "",
    "type": "GEOGRAPHY",
    "description": null,
    "fields": []
  }
]
```

# Intervals between timestamps

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
    `ais-data-385301.uscg.no_dups_simplified`
```

# Maximal Stays Near Ports

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

# Port Visits

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

[usace_principal_ports_monthly_visits.csv](https://docs.google.com/spreadsheets/d/e/2PACX-1vT9yHR2DZl1T239DVl3PX-8P_qr5K2rWG9D6rzaKVj8DK7AqQhYbH0QPCJn5Z2rCpVjfQtaWNV_arpi/pub?gid=393238672&single=true&output=csv)

# West coast

```sql
CREATE TABLE `ais-data-385301.uscg.west_coast` AS
SELECT * FROM `ais-data-385301.uscg.no_dups`
WHERE LON BETWEEN -135 AND -110
AND LAT BETWEEN 20 AND 55;
```

```sql
DECLARE resolution INT64 DEFAULT 4;

-- Create a new table with daily H3 density data
CREATE TABLE `ais-data-385301.uscg.h3_resolution4_daily_density_west_coast` AS
WITH 
  -- Filter locations based on the bounding box
  filtered_locations AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      vessel_geography,
      DATE(BaseDateTime) AS DateOnly  -- Extract the date component
    FROM `ais-data-385301.uscg.west_coast`
    WHERE ST_WITHIN(vessel_geography, ST_GEOGFROMTEXT('POLYGON((-135 20, -110 20, -110 55, -135 55, -135 20))'))
  ),
  -- Create H3 index for each point
  h3_indexed AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      DateOnly,
      jslibs.h3.ST_H3(vessel_geography, resolution) AS h3_index
    FROM filtered_locations
  ),
  -- Count occurrences of each H3 index per day
  daily_density AS (
    SELECT 
      h3_index, 
      DateOnly,
      COUNT(*) AS count
    FROM h3_indexed
    GROUP BY h3_index, DateOnly
  ),
  -- Generate H3 polygons
  h3_polygons AS (
    SELECT 
      h3_index, 
      DateOnly,
      count,
      jslibs.h3.ST_H3_BOUNDARY(h3_index) AS h3_polygon
    FROM daily_density
  )
SELECT * FROM h3_polygons;
```

```python
import numpy as np
import geopandas as gpd
import matplotlib.pyplot as plt
from shapely.geometry import Polygon
import h3
import matplotlib.colors as colors
import pandas as pd

def plot_geo_data_on_date(date, results):
    # Define the lat/lon boundaries
    min_lat, max_lat, min_lon, max_lon = 20, 55, -135, -110

    # Calculate aspect ratio
    width = max_lon - min_lon
    height = max_lat - min_lat
    aspect_ratio = width / height

    # Set the figure size based on the aspect ratio
    fig_width = 10
    fig_height = fig_width / aspect_ratio

    # Filter the DataFrame for the specified date
    filtered_results = results[results['DateOnly'] == pd.to_datetime(date)].copy()

    # Create polygons for each unique hex_id
    hex_polygons = []
    for hex_id in filtered_results['h3_index'].unique():
        hex_boundary = h3.h3_to_geo_boundary(hex_id)
        polygon = Polygon([(lon, lat) for lat, lon in hex_boundary])
        centroid = polygon.centroid
        if min_lat <= centroid.y <= max_lat and min_lon <= centroid.x <= max_lon:
            hex_polygons.append(polygon)

    # Replace inf with max non-inf value and NaN with 1
    filtered_results.loc[filtered_results['count'] == np.inf, 'count'] = filtered_results.loc[filtered_results['count'] != np.inf, 'count'].max()
    filtered_results['count'] = filtered_results['count'].replace({0: 1}).fillna(1)

    # Create a GeoDataFrame
    gdf = gpd.GeoDataFrame(filtered_results, geometry=hex_polygons)

    # Convert 'count' column to float and fill NaN with 1
    gdf['count'] = gdf['count'].astype(float).fillna(1)

    # Calculate min and max values for 'count', setting count_min to 1 if it's 0
    count_min = max(gdf['count'].min(), 1)
    count_max = gdf['count'].max()

    # Plot the choropleth map
    fig, ax = plt.subplots(1, 1, figsize=(fig_width, fig_height))
    gdf.plot(column='count', cmap='YlOrRd', norm=colors.LogNorm(vmin=count_min, vmax=count_max), 
             missing_kwds={'color': 'lightgrey'}, ax=ax)
    plt.xlim(min_lon, max_lon)
    plt.ylim(min_lat, max_lat)
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.title(f'Vessel density map for {date}')
    plt.show()
    plt.savefig(f'ais-{date}.png')
    plt.close()

plot_geo_data_on_date('2022-01-01', results)
```

![West coast vessel density plot for 2022-01-01](https://github.com/jordanbell2357/how-to/assets/47544607/c9a8ae16-167c-4ef4-a834-03417f8caa35)

# West coast hourly

```sql
-- Define the bounding box and hexagon resolution
DECLARE resolution INT64 DEFAULT 4;

-- Create a new table with hourly H3 density data
CREATE TABLE `ais-data-385301.uscg.h3_resolution4_hourly_density_west_coast` AS
WITH 
  -- Filter locations based on the bounding box
  filtered_locations AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      vessel_geography,
      FORMAT_DATETIME('%Y-%m-%d %H', BaseDateTime) AS DateTimeHour  -- Extract date and hour
    FROM `ais-data-385301.uscg.west_coast`
  ),
  -- Create H3 index for each point
  h3_indexed AS (
    SELECT 
      MMSI, 
      BaseDateTime, 
      DateTimeHour,
      jslibs.h3.ST_H3(vessel_geography, resolution) AS h3_index
    FROM filtered_locations
  ),
  -- Count occurrences of each H3 index per hour
  hourly_density AS (
    SELECT 
      h3_index, 
      DateTimeHour,
      COUNT(*) AS count
    FROM h3_indexed
    GROUP BY h3_index, DateTimeHour
  ),
  -- Generate H3 polygons
  h3_polygons AS (
    SELECT 
      h3_index, 
      DateTimeHour,
      count,
      jslibs.h3.ST_H3_BOUNDARY(h3_index) AS h3_polygon
    FROM hourly_density
  )
SELECT * FROM h3_polygons;
```

```python
import numpy as np
import geopandas as gpd
import matplotlib.pyplot as plt
from shapely.geometry import Polygon
import h3
import matplotlib.colors as colors
import pandas as pd

def plot_geo_data_on_hour(datetime_hour, results, all_hex_cells):
    # Define the lat/lon boundaries
    min_lat, max_lat, min_lon, max_lon = 20, 55, -135, -110

    # Calculate aspect ratio
    width = max_lon - min_lon
    height = max_lat - min_lat
    aspect_ratio = width / height

    # Set the figure size based on the aspect ratio
    fig_width = 10
    fig_height = fig_width / aspect_ratio

    # Ensure every hex cell is represented for the hour
    hour_data = results[results['DateTimeHour'] == datetime_hour]
    missing_hex_cells = all_hex_cells - set(hour_data['h3_index'])
    missing_data = pd.DataFrame({'h3_index': list(missing_hex_cells), 
                                 'DateTimeHour': datetime_hour, 
                                 'count': 0})
    hour_data = pd.concat([hour_data, missing_data])

    # Create a mapping of h3_index to polygons
    hex_polygons = {hex_id: Polygon([(lon, lat) for lat, lon in h3.h3_to_geo_boundary(hex_id)])
                    for hex_id in all_hex_cells}

    # Filter out polygons not in the bounding box
    hex_polygons = {hex_id: poly for hex_id, poly in hex_polygons.items()
                    if min_lat <= poly.centroid.y <= max_lat and min_lon <= poly.centroid.x <= max_lon}

    # Assign the corresponding polygon to each row
    hour_data['geometry'] = hour_data['h3_index'].map(hex_polygons)

    # Replace inf with max non-inf value and NaN with 1
    hour_data.loc[hour_data['count'] == np.inf, 'count'] = hour_data.loc[hour_data['count'] != np.inf, 'count'].max()
    hour_data['count'] = hour_data['count'].replace({np.nan: 1, 0: 1})

    # Create a GeoDataFrame
    gdf = gpd.GeoDataFrame(hour_data, geometry='geometry')

    # Convert 'count' column to float and fill NaN with 1
    gdf['count'] = gdf['count'].astype(float).fillna(1)

    # Calculate min and max values for 'count', setting count_min to 1 if it's 0
    count_min = max(gdf['count'].min(), 1)
    count_max = gdf['count'].max()

    # Plot the choropleth map
    fig, ax = plt.subplots(1, 1, figsize=(fig_width, fig_height))
    gdf.plot(column='count', cmap='YlOrRd', norm=colors.LogNorm(vmin=count_min, vmax=count_max), 
             missing_kwds={'color': 'lightgrey'}, ax=ax)
    plt.xlim(min_lon, max_lon)
    plt.ylim(min_lat, max_lat)
    plt.xlabel('Longitude')
    plt.ylabel('Latitude')
    plt.title(f'Vessel density map for {datetime_hour}')
    plt.savefig(f'ais-{datetime_hour}.png')
    plt.close()

all_hex_cells = set(results['h3_index'])
```

```python
plot_geo_data_on_hour('2022-01-01 00', results, all_hex_cells)
```

![2022-01-01 00](https://github.com/jordanbell2357/how-to/assets/47544607/bb0f3bf3-a4e5-4bbe-a1bd-886afc67f6f1)

```python
from datetime import datetime, timedelta

# Function to generate all hours in January 2022
def generate_hours_for_january_2022():
    start_date = datetime(2022, 1, 1)
    end_date = datetime(2022, 1, 31)
    hours = []
    while start_date <= end_date:
        for hour in range(24):
            hours.append(start_date + timedelta(hours=hour))
        start_date += timedelta(days=1)
    return hours

# Function to plot data for each hour in January 2022
def plot_data_for_each_hour(results):
    for datetime_hour in generate_hours_for_january_2022():
        formatted_datetime_hour = datetime_hour.strftime("%Y-%m-%d %H")
        print(f"Plotting for {formatted_datetime_hour}...")
        plot_geo_data_on_hour(formatted_datetime_hour, results, all_hex_cells)
        print(f"Plotting completed for {formatted_datetime_hour}")
```

```python
plt.ioff()  # Turn off interactive plotting
plot_data_for_each_hour(results)
```


