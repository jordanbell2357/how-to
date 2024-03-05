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

# jslibs.h3 UDF for BigQuery

https://github.com/dtws/bigquery-jslibs

UDF `jslibs.h3.ST_H3`

```sql
-- Define the bounding box and hexagon resolution
DECLARE min_lat FLOAT64 DEFAULT 10;
DECLARE max_lat FLOAT64 DEFAULT 60;
DECLARE min_lon FLOAT64 DEFAULT -140;
DECLARE max_lon FLOAT64 DEFAULT -50;
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
