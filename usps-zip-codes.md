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

> -S, --export-file-per-sheet
> 
>              Export a file for each sheet if the exporter only supports one sheet at a time.   The  output
>              filename  is treated as a template in which sheet number is substituted for %n, sheet name is
>              substituted for %s, and sheet object name is substituted for %o in case of graph export.   If
>              there are no substitutions, a default of ".%n" is added.

```
ssconvert -S ZIP_Locale_Detail.xls
```
