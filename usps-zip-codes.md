# USPS ZIP Codes

ZIP Codes by Area and District codes

<https://postalpro.usps.com/ZIP_Locale_Detail>

> The Excel spreadsheet lists ZIP Codes with the associated Area and District Codes.

```bash
wget https://postalpro.usps.com/mnt/glusterfs/2025-12/ZIP_Locale_Detail.xls
```

```bash
sudo apt install gnumeric
```

We use ssconvert, from gnumeric.

> -S, --export-file-per-sheet
>
> Export a file for each sheet if the exporter only supports one sheet at a time.   The  output
>              filename  is treated as a template in which sheet number is substituted for %n, sheet name is
>              substituted for %s, and sheet object name is substituted for %o in case of graph export.   If
>              there are no substitutions, a default of ".%n" is added.

```
ssconvert -S ZIP_Locale_Detail.xls ZIP_Locale_Detail.csv
```

```console
$ wc -l ZIP_Locale_Detail.csv.0
44359 ZIP_Locale_Detail.csv.0
$ wc -l ZIP_Locale_Detail.csv.1
2045 ZIP_Locale_Detail.csv.1
$ wc -l ZIP_Locale_Detail.csv.2
1567 ZIP_Locale_Detail.csv.2
```

```bash
head -n 5 ZIP_Locale_Detail.csv.0
```

```csv
"AREA NAME","AREA CODE","DISTRICT NAME","DISTRICT NO","DELIVERY ZIPCODE","LOCALE NAME","PHYSICAL DELV ADDR","PHYSICAL CITY","PHYSICAL STATE","PHYSICAL ZIP","PHYSICAL ZIP 4"
SOUTHERN,4G,"PUERTO RICO",006,00601,ADJUNTAS,"37 CALLE MUNOZ RIVERA",ADJUNTAS,PR,00601,9998
SOUTHERN,4G,"PUERTO RICO",006,00602,AGUADA,"5 AVE NATIVO ALERS",AGUADA,PR,00602,9998
SOUTHERN,4G,"PUERTO RICO",006,00603,AGUADILLA,"50 CARR 459 STE 1",AGUADILLA,PR,00603,9998
SOUTHERN,4G,"PUERTO RICO",006,00604,RAMEY,"100 AVE BORINQUEN",AGUADILLA,PR,00603,9996
```

We parse the file using awk:

```bash
awk -F ',' '$5 == "90210"' ZIP_Locale_Detail.csv.0
```

```console
WESTPAC,4E,"CALIFORNIA 5",900,90210,"BEVERLY HILLS CARRIER ANNEX","820 N SAN VICENTE BLVD","BEVERLY HILLS",CA,90209,9997
```

