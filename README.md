# mta bus archive

Download archived NYC MTA bus position data, and scrape gtfs-realtime data from the MTA.

Bus position data is archived at [data.mytransit.nyc](http://data.mytransit.nyc).

Requirements:
* Python 3.x
* PostgreSQL 9.5+

## Set up

Create a set of tables in the Postgres database `dbname`:
```
make install PG_DATABASE=dbname
```

This command will create a number of whose tables that begin with `rt_`, notable `rt_vehicle_positions`, `rt_alerts` and `rt_trip_updates`. It will also install the Python requirements, including the [Google Protobuf](https://pypi.python.org/pypi/protobuf/3.3.0) library.

You can specify a remote table using the `PSQLFLAGS` or `MYQSLFLAGS` variables:
```
make install PG_DATABASE=dbname PSQLFLAGS="-U psql_user"
```

## Download an MTA Bus Time archive file

Download a (UTC) day from [data.mytransit.nyc](http://data.mytransit.nyc), and import into the Postgres database `dbname`:
```
make download DATE=20161231 PG_DATABASE=dbname
```

The same, for MySQL:
```
make mysql_download DATE=20161231 PG_DATABASE=dbname
```

## Scraping

Scrapers have been tested with Python 3.4 and above. Earlier versions of Python (e.g. 2.7) won't work.

### Scrape

The scraper depends assumes an environment variable, `BUSTIME_API_KEY`, contains an MTA BusTime API key. [Get a key from the MTA](http://bustime.mta.info/wiki/Developers/Index).

```
export BUSTIME_API_KEY=xyz123
```

Download the current positions from the MTA API and save a local PostgreSQL database named `mtadb`:
```
make positions PG_DATABASE=dbname
```

Download current trip updates:
```
make tripupdates PG_DATABASE=dbname
```

Download current alerts:
```
make alerts PG_DATABASE=dbname
```

## Scheduling

The included `crontab` shows an example setup for downloading data from the MTA API. It assumes that this repository is saved in `~/mta-bus-archive`. Fill-in the `PG_DATABASE` and `BUSTIME_API_KEY` variables before using.

# Setting up Postgres in CentOS

Pick a database name.  In this example it's `mydbname`.

```
sudo make install  # downloads requirements
sudo make create  # initializes postgresql
sudo make init PG_DATABASE=mydbname PG_HOST= PG_USER=myusername
```

## Uploading files to Google Cloud

# Setup

[Create a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) in the Google API Console. Make sure to enable the "Google Cloud Storage API" for your application. Then [set up a service account](https://cloud.google.com/storage/docs/authentication#generating-a-private-key). This will download a file containing credentials named something like `myprojectname-3e1f812da9ac.json`. 

Then run the following (on the machine you'll be using to scrape and upload) and follow instructions:
```
gsutil config -e
```

Next, create a bucket for the data using the [Google Cloud Console](https://console.cloud.google.com/storage/browser).

You've now authenticated yourself to the Google API. You'll now be able to run a command like:
```
make -e gcloud DATE=2017-07-14 PG_DATABASE=mydbname
```

By default, the Google Cloud bucket will have the same name as the database. Use the variable `GOOGLE_BUCKET` to customize it.

# License

Available under the Apache License.
