# OOI Data Team Portal Backend Scripts

This repository includes a number of Python scripts that update metadata and other information provided via the [OOI Data Team portal](https://github.com/ooi-data-review/datateam-portal).

## Requirements

* Installation of the OOI Data Team Portal CakePHP application
* MySQL Database (initialized via the CakePHP app)
* Customized Python or Conda environment (see below)

## Installation

1. Clone the repo onto you machine (This repo should not be web accessible.)

2. Setup the custom environment
 
```
conda create --name datateam python=2.7
source activate datateam
# conda install mysql-python # May be needed on linux distributions
# pip install --upgrade --force-reinstall pip==9.0.3 # If you run into a distutils bug
pip install -r requirements.txt
```

3. Edit the Configuration file
  * `cp config-default.cfg config.cfg`
  * Edit the database information (this should match what you used for the CakePHP app).  Note, by default the scripts use the development server, or you can override that (see below)
  * Add your ooinet credentials.  (This is used by some script to pull information from OOI Net.)

4. Clone additional repos
In order to load the necessary metadata into the portal, you will need to clone the following repos into your working directory.
```
mkdir repos
cd repos
git clone https://github.com/oceanobservatories/preload-database.git
git clone https://github.com/ooi-integration/asset-management.git
git clone https://github.com/ooi-integration/ingestion-csvs.git
cd repos/preload-database/
pip install -r requirements.txt
```

## Initial Import
To populate the Data Team Portal database, run the following scripts.  Note, the -s option is used to specify the database configuration option (specified in the config file) that should be used. 

```
# System Infrastructure Information (from this repo)
./load-data.py -s production -o regions
./load-data.py -s production -o sites
./load-data.py -s production -o nodes
./load-data.py -s production -o instruments
./load-data.py -s production -o instrument_classes
./load-data.py -s production -o instrument_models

# Additional Metadata (from other repos)
./load-data.py -s production -o assets
./load-data.py -s production -o cruises
./load-data.py -s production -o deployments
./load-data.py -s production -o calibrations
./load-data.py -s production -o preload
./load-data.py -s production -o data_streams
./load-data.py -s production -o ingestions

# Annotations (loaded directly from OOI Net)
./load-annotations.py -s production 
```

## Daily Statistics
To load the daily statistics, you will need to install and run the ??? repo.  Because the statistics calculations take a few days to run, we recommend running this script weekly.  Once the statistics have been calculated, you can use the `load-dailystats.py` script to load the results into the database/portal.  Simply specify the output directory as one of the inputs as follows:

```
./load-dailystats.py -s production -d /home/knuth/ooi_stats/stats/output/
```

## Regular Database Updates
Typically, basic system infrastructure information will not change much.  The most common update is adding additional nodes, instruments (reference designators) and data_streams when new gliders are added.  After the appropriate csv files in this repo have been updated, you can easily update the necessary database tables by running the necessary load-data.py commands again.

Updates to preload, asset-management, and ingestions will occur more often.  As a result, it is suggested that these updates be regularly made on a cron.  A number of example cron scripts are included in the repo and can be adapted for your system.  Here is a suggested crontab that updates the statistics and metadata information weekly, while updating the annotations on a daily basis.

```
0 8 * * 1 /home/sage/data-team-python/cron_stats.sh > out_stats.txt 2>&1
0 9 * * 1 /home/sage/data-team-python/cron_update.sh > out_update.txt 2>&1
0 8 * * * /home/sage/data-team-python/cron_annotations.sh > out_annotations.txt 2>&1
```


## Credits

Developed by Sage Lichtenwalner, Rutgers University

With help from the OOI Data Team: Mike Vardaro, Leila Belabassi, Lori Garzio, Friedrich Knuth,  Michael Smith�& Michael Crowley

�2018 OOI Data Team, Rutgers University