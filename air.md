https://psl.noaa.gov/mddb2/makePlot.html?variableID=1603

https://psl.noaa.gov/data/gridded/data.ghcncams.html

https://www.earthdata.nasa.gov/learn/find-data/near-real-time/hazards-and-disasters/air-quality

https://lpdaac.usgs.gov/products/mod11c3v061/#tools

MOD11C3 v061

MODIS/Terra Land Surface Temperature/Emissivity Monthly L3 Global 0.05 Deg CMG

```python
#Import packages
import earthaccess

#Authentication with Earthdata Login
auth = earthaccess.login(strategy="netrc")

results = earthaccess.search_data(short_name="OSCAR_L4_OC_third-deg_YEARLY",
                                  version="1",
                                  cloud_hosted=True,
                                  temporal = ("2022-01-01","2022-07-31"),
                                  bounding_box = (-51.96423,68.10554,-48.71969,70.70529))

earthaccess.download(results, "./local_folder")
```

```python
import xarray as xr

ds = xr.open_mfdataset("local_folder/oscar_vel2022.nc")

ds.info()

import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt

plt.figure(figsize=(18, 9))

ax = plt.axes(projection=ccrs.PlateCarree()) # plate carrée projection

dec = 2

lon = ds.longitude.values[::dec]

lon[lon > 180] = lon[lon > 180] - 360

mymap = plt.streamplot(
    lon,
    ds.latitude.values[::dec],
    ds.u.values[0, 0, ::dec, ::dec],
    ds.v.values[0, 0, ::dec, ::dec],
    8,
    transform = ccrs.PlateCarree()
)

ax.coastlines()

plt.title('Sea surface currents derived from OSCAR')

plt.savefig("currents18x9E.png", dpi=150)

plt.show()
```
