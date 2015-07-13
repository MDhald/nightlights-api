# Summary Data

The monthly summary data needed by the nightlights-api are as follows:

**`villages.csv`:** the villages, with location and region info.
This file *should* have a header, and the following structure:
```
villagecode,longitude,latitude,state,district,acid
```

The following files, containing the time series, should *not* have headers.

**`districts.csv`**
```
state, district, year, month, satellite, num_observations, vis_mean, vis_sd, vis_min, vis_median, vis_max
```

**`states_months.csv`**
```
state, year, month, satellite, num_observations, vis_mean, vis_sd, vis_min, vis_median, vis_max
```

**Column Info:**
latitude, longitude: decimal degrees
state: state name (e.g. 'Uttar Pradesh')
district: district name (e.g., 'Hardoi')
year: numeric, e.g. 2011
month: numeric, month of year, 1 == January
satellite: string (e.g. 'F10')
num_observations, vis_mean, vis_sd, vis_min, vis_median, vis_max: numeric

Instructions for generating these data from the raw nightly TIF files are below.

## Dependencies

The following pipeline depends on:

 - R >= v3.0
 - csvcut

## Extract tabular data from nightly TIFs

Use [this R script](DMSP_extract.R) to extract data for village points like so:

```sh
r --slave --no-restore --file=DMSP_extract.R --args /absolute/path/to/village-points.shp input-dir output-dir
```

Note that for the entire 20-year timespan, the final output is ~1.2 TB, so that doing the entire extraction in serial is infeasible.


## Clean the CSV

[clean-csv.sh](clean-csv.sh) does a bit of cleanup on the CSV, preparing it for
use with Redshift (or another SQL) db.  Use it with `./clean-csv.sh
the-csv-file.csv`, or parallelize with GNU parallel with `parallel
./clean-csv.sh ::: your/csv/files/*.csv`.

## Generate summaries

For generating the monthly summaries of the dataset, we used Amazon Redshift to
deal with the large volume of data, but the scripts could be relatively easily
adapted to another SQL database.

To use redshift to create summaries, put the cleaned CSV data onto an S3
bucket, and then use [summaries.sql](summaries.sql) to generate the summaries,
replacing `credentials` in `summaries.sql` with your AWS credentials.

