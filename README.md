# Spatially interpolated non-smoke and smoke PM2.5 concentrations for the US from 2006-2024

[https://doi.org/10.5061/dryad.k0p2ngfhv](https://doi.org/10.5061/dryad.k0p2ngfhv)

## Description of the data and file structure

**We estimated daily smoke and non-smoke PM2.5 across the contiguous US (CONUS) for 2006-2023 using Environmental Protection Agency (EPA) ground monitors and NOAA Hazard Mapping System (HMS) smoke polygons.**

The daily PM2.5 data from EPA ground monitors are interpolated to ~15 km resolution to create a total PM2.5 estimate. The HMS smoke polygons are then used to identify locations where there is likely smoke somewhere in the atmospheric column. The seasonal mean or median background is calculated using pixels within the season where an HMS (dense, medium, or light) smoke polygon is not located. The seasonal background can then be subtracted from the total PM2.5 to estimate smoke PM2.5.

In recent years, an active fire season has resulted in some regions having HMS smoke polygons during the entire season or almost the entire season. To account for this, a minimum number of 15 days is required to estimate the background concentration. If there are not 15 days at a pixel, additional days just prior to and after season are included until the minimum threshold is met.

This dataset was produced using code written by Katelyn O'Dell and updated by Jennifer McGinnis. The original code can be found in the Software files.

### Files and variables

The file named “2006_2023_Mean_Background.zip” includes the 2006-2023 kriged data (.nc files) where the background PM2.5 concentration was calculated using the mean value of days with no HMS smoke polygon. The “2006_2023_Median_Background.zip” file includes the 2006-2023 kriged data (.nc files) where the background PM2.5 concentration was calculated using the median value of days with no HMS smoke polygon. For most applications, we recommend using the mean background concentration for calculating the contribution of PM2.5 from smoke. Our method of subtracting the mean or median non-smoke PM2.5 from the total PM2.5 to estimate smoke PM2.5 may result in some negative smoke PM2.5 values. To ensure your analysis accurately represents the data, we recommend including these negative values rather than omitting or altering them.

The file named “Intermediary_Files_Figures” includes the intermediate datasets (.npz files) and summary figures (.png files) created in the process of creating the final kriged data. The intermediary datasets can be used to calculate the background concentrations differently to how it was explained above. More information about the files and how they were made can be found in the Supplemental  and Software materials.

"2024_Mean_Background.zip" and "2024_Median_Background.zip" were also added, in the same format. 

The kriged data is a netCDF (.nc) file with the variables:

“doy”: day of year

“lon”: degrees longitude for data grid centers

“lat”: degrees latitude for data grid centers

“we_lon”: degrees longitude for data grid west-east borders

“we_lat”: degrees latitude for data grid west-east borders

“ns_lon”: degrees longitude for data grid north-south borders

“ns_lat”: degrees latitude for data grid north-south borders

“PM25”: 24 hour average PM2.5 concentration (µg/m3)

“Background_PM25”: seasonal PM2.5 background concentration (µg/m3) (JFM, AMJ, JAS, OND)

“HMS_Smoke”: Binary HMS Smoke (1=smoke, 0=no smoke)

“testing_sites_longitude”: longitudes for EPA AQS sites used for kriging LOOCV

“testing_sites_latitude”: latitudes for EPA AQS sites used for kriging LOOCV

“r_squared”: r-squared for LOOCV

“mean_bias”: mean bias for LOOCV

“mean_absolute_error”: mean absolute error for LOOCV

“slope”: linear regression slope

“nobs”: number of monitor observations for LOOCV

## Code/software

See the Zenodo links for Software.

## Access information

Other publicly accessible locations of the data:

* [https://mountainscholar.org/items/89896cf7-1a64-43be-a61b-44cd460e9632](https://mountainscholar.org/items/89896cf7-1a64-43be-a61b-44cd460e9632)
* [https://mountainscholar.org/items/cf8053c6-6a16-49e7-8b0f-1044322d867e](https://mountainscholar.org/items/cf8053c6-6a16-49e7-8b0f-1044322d867e)
* [https://mountainscholar.org/items/370ac2d9-422f-4256-a764-b75a5a84e724](https://mountainscholar.org/items/370ac2d9-422f-4256-a764-b75a5a84e724)

Data was derived from the following sources:

* Environmental Protection Agency (EPA) 24-hour PM2.5 ground monitor data
* Hazard Mapping System (HMS) smoke and fire product

