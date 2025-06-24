# SDI-hub deployment project

This project contains the configuration of a typical SDI setup

The hub infrastructure is based on docker-compose, internally using traefic, terria, mapserver, pycsw, postgres/sqlite 

---

## Getting started

This setup assumes a local computer or Virtual machine, with docker and installed

### Environment

Copy the environment file from `.env-template` to `.env` 

Update the environment file to local needs
 
```
vi .env
```

## Catalogue

Implemented as ([pycsw](https://www.pycsw.org)). A postgres database backend is configured via environment variables (.env), in case the database is empty, the required tables are created by pycsw.

Import metadata records (pycsw container should be running)

```
docker exec pycsw pycsw-admin.py load-records -c /etc/pycsw/pycsw.yml -p /home/records -r -v WARNING
```

- expects records in xml format at /home/records (mounted into the container)
- expects config file at /etc/pycsw/pycsw.yml (containing database connection details)


### Metadata crawler

Initialize metadata for files in webdav (once)
```
docker run -it --rm -v$(pwd):/geodata pvgenuchten/geodatacrawler crawl-metadata -
-dir=/geodata --mode=init
```
Update metadata in case new verions are uploaded
```
docker run -it --rm -v$(pwd):/geodata pvgenuchten/geodatacrawler crawl-metadata -
-dir=/geodata --mode=init
```

## Mapserver

Mapserver, runs OGC WMS/WFS/WCS services on data, is configured using [mapfiles](https://www.mapserver.org/mapfile/) and a [global-config](https://www.mapserver.org/mapfile/config.html)

### Generate mapfiles

A number of environment variables is required as part of mapserver generation.
The variables can best be passed in using a .env file.

Create a .env file at ./webdav with content:
```
pgdc_md_url=https://example.com/cat/collections/metadata:main/items/{0}
pgdc_ms_url=http://example.com/ows/
pgdc_webdav_url=http://example.com/files/
```

A mapfile is typically generated per folder. Navigate into the folder and then generate the mapfile.
```
docker run -it --rm --env-file=../.env -v=$(pwd):/geodata pvgenuchten/geodatacrawler crawl-maps -
-dir=/geodata 
```
### Run mapserver

- An alias to each mapfile (folder) needs to be placed in ./webdav/ms.conf (rename from ms.conf.template)
- Also check the environment variables in docker compose
- ./webdav is mounted into container as /share/data

Test mapserver using [map2img](https://mapserver.org/utilities/map2img.html) utility

```
docker exec -it mapserver map2img --help
docker exec -it mapserver map2img -m /srv/data/cec.map
```

## Terria mapviewer

Mapviewer ([TerriaJS](https://terria.io)), a javascript based map viewer, some config files are mounted into the container

## Database

Database (PostGreSQL), initial startup is slow, configured via environment variables


## For development

In case you want to run a minimal setup for development, you can run 'docker-compose-sqlite.yml' which uses a local sqlite database with some dummy data. By default TerriaJS is also disabled. 

```cmd
docker compose -f docker-compose-sqlite.yml up
```



