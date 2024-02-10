# Monthly Climate Summaries

https://climate.weather.gc.ca/prods_servs/cdn_climate_summary_e.html

```bash
ubuntu@LAPTOP-JBell:~/climate$ dos2unix en_climate_summaries_ON*.csv

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

Make coordinates file `coords.dat` of first 200 entries in `.csv`:

```bash
ubuntu@LAPTOP-JBell:~/climate$ awk -F, 'NR==1 {next} NR<=201 {gsub(/"/, "", $1); gsub(/"/, "", $2); print $1, $2}' climate_summaries_ON.csv > coords.dat
```

```bash
ubuntu@LAPTOP-JBell:~/climate$ head coords.dat
-82.432 52.928
-89.897 53.818
-89.892 53.816
-93.221 50.631
-87.676 56.019
-87.936 52.196
-91.763 53.441
-85.433 54.983
-90.218 51.449
-90.214 51.446
```

Gnuplot:

```bash
ubuntu@LAPTOP-JBell:~/climate$ gnuplot
gnuplot> plot 'coords.dat' using 1:2 with points
```

![image](https://github.com/jordanbell2357/how-to/assets/47544607/9695ae7d-9fb7-4940-bd42-17860d965354)

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
```
