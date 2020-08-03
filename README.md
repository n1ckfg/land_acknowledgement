# Land Acknowledgement SMS App
Land Acknowledgement is a simple api that links data about native lands to current locations. 

# Data Sources
Data about boundaries of Native land is from: [native-land.ca](https://native-land.ca)   
Locations lookups are determined using data files from GeoNames (https://www.geonames.org) [zipcode data](http://download.geonames.org/export/zip/)

# Running locally
**Requirements**  
[Docker](https://www.docker.com/get-started)

To install an run from docker container

```
git clone https://github.com/codeforanchorage/land_acknowledgement.git  
cd land_acknowledgement  
docker-compose up
```

This will create docker images and start containers for postgreSQL and a simple python web server. Once running you should be able to make POST requests to `localhost:8000/` with something like Postman or curl:

```
curl -XPOST localhost:8000/ -d Body=Anchorage%2C+ak
```

In development the webserver, gunicorn, will relaod on changes to the code. 

To bring service down and remove containers use:

```
docker-compose down
```


While the containers are running you can execute and attach a shell environment with:

```
docker exec -it landak_web sh 
```



# Deploying


Heroku:
Add a postgres database add-on. The number of row in the database exceeds the limits for the free tier postgres. This requires at least the `Hobby-basic` ($9.00/month).

#### Add postgis extension
Connect to heroku postgres via the cli and add extension with `create extension postgis`

```
% heroku pg:psql --app landak
landak::DATABASE=> create extension postgis;
```

Exit psql with `\q`

#### Create backup of local database to restore to heroku
The compressed format `--format=c` is required for Heroku.

```
pg_dump --format=c --compress=9 -U postgres postgres  > local_data/data_dump.sql
```

Upload the compressed sql file to S3(or some other accessible place) and pass it to the Heroku cli
 
 ```
 heroku pg:backups:restore https://landak-sql-dump.s3-us-west-2.amazonaws.com/data_dump.sql postgresql-opaque-24019 --app landak
 ```

The Dockerfile on the top-level directory will build an image appropriate for Heroku. To build a push this use:

```
heroku stack:set container --app landak
heroku container:push web --app landak
heroku container:release web --app landak
```

