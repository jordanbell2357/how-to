# Monthly Climate Summaries

## Download and process CSV files

https://climate.weather.gc.ca/prods_servs/cdn_climate_summary_e.html

```bash
ubuntu@LAPTOP-JBell:~/climate$ cat en_climate_summaries_legend.txt
﻿Legend
Long    Longitude (West - , degrees)
Lat     Latitude (North + , degrees)
Stn_Name        Station Name
Clim_ID Climate Identifier
Prov_or_Ter     Province or Territory
Tm      Mean Temperature (°C)
DwTm    Days without Valid Mean Temperature
D       Mean Temperature difference from Normal (1981-2010) (°C)
Tx      Highest Monthly Maximum Temperature (°C)
DwTx    Days without Valid Maximum Temperature
Tn      Lowest Monthly Minimum Temperature (°C)
DwTn    Days without Valid Minimum Temperature
S       Snowfall (cm)
DwS     Days without Valid Snowfall
S%N     Percent of Normal (1981-2010) Snowfall
P       Total Precipitation (mm)
DwP     Days without Valid Precipitation
P%N     Percent of Normal (1981-2010) Precipitation
S_G     Snow on the ground at the end of the month (cm)
Pd      Number of days with Precipitation 1.0 mm or more
BS      Bright Sunshine (hours)
DwBS    Days without Valid Bright Sunshine
BS%     Percent of Normal (1981-2010) Bright Sunshine
HDD     Degree Days below 18 °C
CDD     Degree Days above 18 °C
NA      Not Available
```

```bash
ubuntu@LAPTOP-JBell:~/climate$ dos2unix en_climate_summaries_ON*.csv
```

## Inspect and plot with Gnuplot

```bash
ubuntu@LAPTOP-JBell:~/climate$ head en_climate_summaries_ON_01-2024.csv
"Long","Lat","Stn_Name","Clim_ID","Prov_or_Ter","Tm","DwTm","D","Tx","DwTx","Tn","DwTn","S","DwS","S%N","P","DwP","P%N","S_G","Pd","BS","DwBS","BS%","HDD","CDD"
"-82.432","52.928","ATTAWAPISKAT A","6010400","ON","-15.8","0","NA","0.2","0","-31.5","0","","","NA","3.6","0","NA","","0","","","NA","1048.0","0.0"
"-89.897","53.818","BIG TROUT LAKE A","6010735","ON","-17.7","1","NA","1.1","1","-33.3","1","","","NA","8.8","0","NA","","4","","","NA","1070.2","0.0"
"-89.892","53.816","BIG TROUT LAKE","6010740","ON","-17.5","0","NA","1.3","0","-34.4","0","","","NA","26.3","0","NA","43.0","9","","","NA","1099.6","0.0"
"-93.221","50.631","EAR FALLS (AUT)","6012199","ON","-12.6","0","NA","7.5","0","-31.2","0","","","NA","14.5","0","NA","21.0","6","","","NA","947.5","0.0"
"-87.676","56.019","FORT SEVERN A","6012501","ON","-19.2","0","NA","1.5","0","-34.7","0","","","NA","0.7","0","NA","","0","","","NA","1152.4","0.0"
"-87.934","52.196","LANSDOWNE HOUSE A","6014351","ON","-16.0","0","NA","1.1","0","-33.3","0","","","NA","4.5","0","NA","","1","","","NA","1053.6","0.0"
"-87.936","52.196","LANSDOWNE HOUSE (AUT)","6014353","ON","-16.0","0","NA","0.9","0","-33.4","0","","","NA","13.9","0","NA","10.0","5","","","NA","1054.0","0.0"
"-91.763","53.441","MUSKRAT DAM","6015026","ON","-17.2","0","NA","1.0","0","-35.8","0","","","NA","0.7","25","NA","","0","","","NA","1090.9","0.0"
"-85.433","54.983","PEAWANUCK (AUT)","6016295","ON","-19.4","5","NA","1.4","5","-37.2","5","","","NA","17.4","5","NA","5.0","5","","","NA","971.5","0.0"
```

```bash
ubuntu@LAPTOP-JBell:~/climate$ awk -F, 'NR>1 {for (i=1; i<=NF; i++) {gsub(/"/, "", $i); if ($i == "NA") $i = "NaN"; if (i != 3 && i != 4 && i != 5) printf "%s ", $i} printf "\n"}' en_climate_summaries_ON_01-2024.csv > en_climate_summaries_ON_01-2024.dat
ubuntu@LAPTOP-JBell:~/climate$ head en_climate_summaries_ON_01-2024.dat
-82.432 52.928 -15.8 0 NaN 0.2 0 -31.5 0   NaN 3.6 0 NaN  0   NaN 1048.0 0.0
-89.897 53.818 -17.7 1 NaN 1.1 1 -33.3 1   NaN 8.8 0 NaN  4   NaN 1070.2 0.0
-89.892 53.816 -17.5 0 NaN 1.3 0 -34.4 0   NaN 26.3 0 NaN 43.0 9   NaN 1099.6 0.0
-93.221 50.631 -12.6 0 NaN 7.5 0 -31.2 0   NaN 14.5 0 NaN 21.0 6   NaN 947.5 0.0
-87.676 56.019 -19.2 0 NaN 1.5 0 -34.7 0   NaN 0.7 0 NaN  0   NaN 1152.4 0.0
-87.934 52.196 -16.0 0 NaN 1.1 0 -33.3 0   NaN 4.5 0 NaN  1   NaN 1053.6 0.0
-87.936 52.196 -16.0 0 NaN 0.9 0 -33.4 0   NaN 13.9 0 NaN 10.0 5   NaN 1054.0 0.0
-91.763 53.441 -17.2 0 NaN 1.0 0 -35.8 0   NaN 0.7 25 NaN  0   NaN 1090.9 0.0
-85.433 54.983 -19.4 5 NaN 1.4 5 -37.2 5   NaN 17.4 5 NaN 5.0 5   NaN 971.5 0.0
-85.443 54.988 -19.0 1 NaN 1.3 1 -37.2 1   NaN 2.2 1 NaN  1   NaN 1108.5 0.0
```

```bash
ubuntu@LAPTOP-JBell:~/climate$ gnuplot
gnuplot> set title "Mean Temperature (°C)"
gnuplot> set xlabel "Longitude"
gnuplot> set ylabel "Latitude"
gnuplot> set style fill solid
gnuplot> set pointsize 2
gnuplot> set palette defined (0 "blue", 1 "white", 2 "red")
gnuplot> plot "en_climate_summaries_ON_01-2024.dat" using 1:2:3 with points pt 7 palette title ""
```

![image](https://github.com/jordanbell2357/how-to/assets/47544607/4d645f73-d132-4293-b6d1-d852bd60ac97)



## Process CSV files

```bash
ubuntu@LAPTOP-JBell:~/climate$ cat date_column.sh
#!/bin/bash

# Iterate over each file matching the pattern
for file in en_climate_summaries_ON_*.csv; do
    # Extract the year and month from the file name
    # 'cut' is used with '-' as the delimiter. We take the 2nd field for year and 1st field after last '_' for month
    year=$(echo "$file" | cut -d '-' -f 2 | cut -d '.' -f 1)
    month=$(echo "$file" | rev | cut -d '_' -f 1 | rev | cut -d '-' -f 1)

    # Define the new file name with year and month reordered
    new_file="climate_summaries_ON_${year}-${month}.csv"

    # Read the file, add the Date column, and output to the new file
    awk -v year="$year" -v month="$month" 'BEGIN {FS=OFS=","}
        NR==1 {print $0, "\"Date\""}
        NR>1 {print $0, "\"" year "-" month "\""}' "$file" > "$new_file"

    echo "Processed $file into $new_file"
done

echo "All files processed."

ubuntu@LAPTOP-JBell:~/climate$ bash date_column.sh
```

## Combine CSV files

```bash
ubuntu@LAPTOP-JBell:~/climate$ cat combine_csv.sh
#!/bin/bash

# Create an empty file for the final output
output_file="climate_summaries_ON.csv"
> "$output_file"

# Initialize a variable to track if the first file is being processed
first_file=1

# List and sort files, then iterate over them
for file in $(ls climate_summaries_ON_*.csv | sort -t '_' -k 3.1,3.4 -k 3.6,3.7); do
    if [ "$first_file" -eq 1 ]; then
        # For the first file, include the header
        head -n 1 "$file" > "$output_file"
        first_file=0
    fi

    # Append the file content without the header
    tail -n +2 "$file" >> "$output_file"

    echo "Processed $file"
done

echo "All files have been combined into $output_file"

ubuntu@LAPTOP-JBell:~/climate$ bash combine_csv.sh

ubuntu@LAPTOP-JBell:~/climate$ head climate_summaries_ON.csv
"Long","Lat","Stn_Name","Clim_ID","Prov_or_Ter","Tm","DwTm","D","Tx","DwTx","Tn","DwTn","S","DwS","S%N","P","DwP","P%N","S_G","Pd","BS","DwBS","BS%","HDD","CDD","Date"
"-82.432","52.928","ATTAWAPISKAT A","6010400","ON","-13.0","6","NA","3.4","6","-31.8","6","","","NA","13.4","6","NA","","2","","","NA","774.8","0.0","2017-01"
"-89.897","53.818","BIG TROUT LAKE A","6010735","ON","-14.6","19","NA","1.0","19","-31.8","19","","","NA","1.4","19","NA","","1","","","NA","391.2","0.0","2017-01"
"-89.892","53.816","BIG TROUT LAKE","6010740","ON","-16.1","0","NA","2.1","0","-34.9","0","","","NA","30.2","0","NA","38.0","7","","","NA","1057.6","0.0","2017-01"
"-93.221","50.631","EAR FALLS (AUT)","6012199","ON","-12.0","0","NA","6.3","0","-33.0","0","","","NA","22.0","0","NA","22.0","6","","","NA","929.8","0.0","2017-01"
"-87.676","56.019","FORT SEVERN A","6012501","ON","-18.4","9","NA","0.7","9","-33.1","9","","","NA","0.5","9","NA","","0","","","NA","800.0","0.0","2017-01"
"-87.936","52.196","LANSDOWNE HOUSE (AUT)","6014353","ON","-14.8","0","NA","4.3","0","-35.0","0","","","NA","29.2","0","NA","40.0","7","","","NA","1016.1","0.0","2017-01"
"-91.763","53.441","MUSKRAT DAM","6015026","ON","-12.2","9","NA","3.6","9","-34.5","9","","","NA","3.6","9","NA","","1","","","NA","664.3","0.0","2017-01"
"-85.433","54.983","PEAWANUCK (AUT)","6016295","ON","-17.6","0","NA","1.6","0","-33.4","0","","","NA","18.6","0","NA","13.0","6","","","NA","1103.4","0.0","2017-01"
"-90.218","51.449","PICKLE LAKE (AUT)","6016525","ON","-13.9","0","NA","4.8","0","-32.7","0","","","NA","38.1","0","NA","39.0","11","","","NA","987.4","0.0","2017-01"
```

## GeoJSON

```bash
ubuntu@LAPTOP-JBell:~/climate$ cat csv_to_geojson.sh
#!/bin/bash

input_file="climate_summaries_ON.csv"
output_file="climate_summaries_ON.geojson"

# Convert the CSV to GeoJSON and include description in each entry
ogr2ogr -f "GeoJSON" -oo X_POSSIBLE_NAMES="Long" -oo Y_POSSIBLE_NAMES="Lat" -oo KEEP_GEOM_COLUMNS="NO" "/vsistdout/" "$input_file" | \
jq '.features[].properties += {
    "description_Long": "Longitude (West - , degrees)",
    "description_Lat": "Latitude (North + , degrees)",
    "description_Stn_Name": "Station Name",
    "description_Clim_ID": "Climate Identifier",
    "description_Prov_or_Ter": "Province or Territory",
    "description_Tm": "Mean Temperature (°C)",
    "description_DwTm": "Days without Valid Mean Temperature",
    "description_D": "Mean Temperature difference from Normal (1981-2010) (°C)",
    "description_Tx": "Highest Monthly Maximum Temperature (°C)",
    "description_DwTx": "Days without Valid Maximum Temperature",
    "description_Tn": "Lowest Monthly Minimum Temperature (°C)",
    "description_DwTn": "Days without Valid Minimum Temperature",
    "description_S": "Snowfall (cm)",
    "description_DwS": "Days without Valid Snowfall",
    "description_S%N": "Percent of Normal (1981-2010) Snowfall",
    "description_P": "Total Precipitation (mm)",
    "description_DwP": "Days without Valid Precipitation",
    "description_P%N": "Percent of Normal (1981-2010) Precipitation",
    "description_S_G": "Snow on the ground at the end of the month (cm)",
    "description_Pd": "Number of days with Precipitation 1.0 mm or more",
    "description_BS": "Bright Sunshine (hours)",
    "description_DwBS": "Days without Valid Bright Sunshine",
    "description_BS%": "Percent of Normal (1981-2010) Bright Sunshine",
    "description_HDD": "Degree Days below 18 °C",
    "description_CDD": "Degree Days above 18 °C"
}' > "$output_file"

echo "Conversion complete: $output_file"

ubuntu@LAPTOP-JBell:~/climate$ bash csv_to_geojson.sh

ubuntu@LAPTOP-JBell:~/climate$ head climate_summaries_ON.geojson
{
  "type": "FeatureCollection",
  "name": "climate_summaries_ON",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "Stn_Name": "ATTAWAPISKAT A",
        "Clim_ID": "6010400",
        "Prov_or_Ter": "ON",
```

[climate_summaries_ON.geojson](https://github.com/jordanbell2357/how-to/blob/main/climate_summaries_ON.geojson)

We run `psql`

```bash
sudo -u postgres psql
```

We create PostgreSQL DB `monthly_climate_summaries` and in it execute `CREATE EXTENSION postgis;`

```sql
postgres=# CREATE DATABASE monthly_climate_summaries;
CREATE DATABASE
postgres=# \c monthly_climate_summaries;
You are now connected to database "monthly_climate_summaries" as user "postgres".
monthly_climate_summaries=# CREATE EXTENSION postgis;
CREATE EXTENSION
monthly_climate_summaries=# quit
```

Now we load `geojson` file into PostGIS.

Reference: https://gdal.org/drivers/vector/pg.html

We run the following

```bash
ogr2ogr -f "PostgreSQL" PG:"dbname='monthly_climate_summaries' host='localhost' port='5432' user='postgres' password='postgres'" climate_summaries_ON.geojson
```

This creates a table `climate_summaries_on` in the DB `monthly_climate_summaries`.

```sql
monthly_climate_summaries=# SELECT COUNT(*) FROM climate_summaries_on;
 count
-------
 14633
(1 row)
monthly_climate_summaries=# quit
```

