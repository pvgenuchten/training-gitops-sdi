---
title: Providing convenience APIs
author: Paul van Genuchten
date: 2023-05-09
---

For spatial datasets it is of interest to share them via convenience APIs, so the datasets can be downloaded in parts or easily be visualised in common tools such as [QGIS](https://qgis.org), [OpenLayers](https://openlayers.org) & [Leaflet](https://leaflet.org). The standards on data services of the [Open Geospatial Consortium](https://www.ogc.org/) are designed with this purpose. These APIs give direct access to subsets or map visualisations of a dataset. 
 
In this paragraph you will be introduced to various standardised APIs, after which we introduce an approach to publish datasets, which builds on the data management approach introduced in the previous paragraphs.  

---

## Standardised data APIs 

Standardised mapping APIs, such as Web Map Service (WMS), Web Feature service (WFS) and Web Coverage Service (WCS), originate from the beginning of this century. In recent years several challenges have been identified around these standards, which led to a series of [Spatial data on the web best practices](https://www.w3.org/TR/sdw-bp/). OGC then initiated a new generation of standards based on these best practices.

An overview of both generations:

| OWS | OGC-API | Description |
| --- | --- | --- |
| Web Map Service ([WMS](https://www.ogc.org/standard/wms/)) | [Maps](https://ogcapi.ogc.org/maps/) | Provides a visualisation of a subset of the data |
| Web Feature Service ([WFS](https://www.ogc.org/standard/wfs/)) | [Features](https://ogcapi.ogc.org/features/) | API to request a subset of the vector features |
| Web Coverage Service ([WCS](https://www.ogc.org/standard/wcs/)) | [Coverages](https://ogcapi.ogc.org/coverages/) | API to interact with grid sources |
| Sensor Observation Service ([SOS](https://www.ogc.org/standard/sos)) | [Sensorthings](https://www.ogc.org/standard/sensorthings/) | Retrieve subsets of sensor observations |
| [Web Processing Service](https://www.ogc.org/standard/wps) (WPS) | [OGCAPI:Processes](https://ogcapi.ogc.org/processes) | Run processes on data ]
| [Catalogue Service for the web](https://www.ogc.org/standard/cat) (CSW) | [OGCAPI:Records](https://ogcapi.ogc.org/records) | Retrieve and filter catalogue records |

Notice that most of the mapping software supports the standards of both generations. However, due to their recent
introduction, expect incidental challenges in the implementations of OGC APIs. 

---

## Setting up an API

[Mapserver](https://mapserver.org) is server software which is able to expose datasets through various APIs. 
Examples of similar software are [QGIS server](https://docs.qgis.org/3.28/en/docs/server_manual/introduction.html), 
[ArcGIS Server](https://enterprise.arcgis.com/en/server/), [Geoserver](https://geoserver.org) and 
[pygeoapi](https://pygeoapi.io).
 
We've selected mapserver for this training, because of its robustness and low resource consumption.
Mapserver is configured using a configuration file: called the [mapfile](https://www.mapserver.org/mapfile/). 
The mapfile defines metadata for the dataset and how users interact with the dataset, mainly the colour 
scheme (legend) to draw a map of the dataset.  

Various tools exist to write these configuration files, such as [Mapserver studio](https://mapserverstudio.net/), 
[GeoStyler](https://www.osgeo.org/projects/geostyler/), [QGIS Bridge](https://www.geocat.net/docs/bridge/qgis/latest), 
up to a [visual studio plugin to edit mapfiles](https://marketplace.visualstudio.com/items?itemName=chicoff.mapfile).

The [GeoDataCrawler](https://pypi.org/project/geodatacrawler/), introduced in a 
[previous paragraph](./2-interact-with-data-repositories.md), also has an option to generate mapfiles. 
A big advantage of this approach is the integration with existing metadata. 
GeoDataCrawler will, during mapfile generation, use the existing metadata, but also update the metadata 
so it includes a link to the mapserver service endpoint. This step enables a typical workflow of: 

- Users find a dataset in a catalogue 
- Then open the dataset via the linked service

But also vice versa; from a mapping application, access the metadata describing a dataset.

---

## Mapfile creation exercise

- Navigate with shell to a folder with data files.
- Verify if mcf's are available for the files, if not, create initial metadata with `crawl-metadata --mode=init --dir=.`
- Add a index.yml file to the folder. This metadata is introduced in the mapfile to identify the service.

```yaml
mcf:
   version: 1.0
identification:
    title: My new mapservice
    abstract: A map service for data about ...
contact:
  pointOfContact:
    organization: ISRIC
    email: info@isric.org
    url: https://www.isric.org
```

- Set some environment variables

::: {.panel-tabset}
# Linux
```bash
export pgdc_md_url="https://kenya.lsc-hubs.org/collections/metadata:main/items/{0}"
export pgdc_ms_url="http://localhost"
export pgdc_webdav_url="https://example.com/data"
```
# Powershell
```bash
$pgdc_md_url="https://kenya.lsc-hubs.org/collections/metadata:main/items/{0}"
$pgdc_ms_url="http://localhost"
$pgdc_webdav_url="https://example.com/data"
```
:::

- Generate the mapfile

::: {.panel-tabset}
# Local
```bash
crawl-maps --dir=.
```
# Docker & Linux
```bash
docker run -it --rm -v$(pwd):/tmp \
  org/metatraining crawl-maps --dir=/tmp 
```
# Docker & Powershell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  org/metatraining crawl-maps --dir=/tmp 
```
:::

- Index.yml may include a "robot" property, to guide the crawler in how to process the folder. This section can be used to add specific crawling behaviour.

```yaml
mcf:
    version: 1.0
robot:
    skip-subfolders: True # indicates the crawler not to proceed in subfolders
```

You can test this mapfile locally if you have mapserver installed. On windows, consider using conda or [ms4w](https://www.ms4w.com/).

```bash
conda install -c conda-forge mapserver
```

Mapserver includes a [map2img](https://mapserver.org/utilities/map2img.html) utility, which enables to render a map image from any mapfile.

```bash
map2img -m=./mymap.map -o=test.png
```

---

## Setup mapserver via Docker 

For this exercise we're using a [mapserver image](https://hub.docker.com/r/camptocamp/mapserver) available from Docker hub.

```bash
docker pull camptocamp/mapserver:master  
```

First create a config file, which we'll [mount as a volume](https://docs.docker.com/storage/volumes/) into the container. On this config file we list all the mapfiles we aim to publish on our container. Download [the default config file](https://github.com/camptocamp/docker-mapserver/blob/master/runtime/etc/mapserver.conf). Open the file and unescape and populate the maps section:

```yaml
MAPS
     "data" "/srv/data/data.map"
END
```

Also unescape the OGCAPI templates section

```yaml
OGCAPI_HTML_TEMPLATE_DIRECTORY "/usr/local/share/mapserver/ogcapi/templates/html-bootstrap4/"
```

In the next statement we mount the data folder, including the config file and indicate on which port and with which config file the container will run:

::: {.panel-tabset}
# Linux
```bash
docker run -p 80:80 \
    -e MAPSERVER_CONFIG_FILE=/srv/data/mapserver.conf \
    -v $(pwd):/srv/data  \
    camptocamp/mapserver:master 
```
# Powershell
```bash
docker run -p 80:80 `
    -e MAPSERVER_CONFIG_FILE=/srv/data/mapserver.conf `
    -v "${PWD}:/srv/data" `
    camptocamp/mapserver:master 
```
:::

Check http://localhost/data/ogcapi in your browser. If all has been set up fine it should show the OGCAPI homepage of the service. 
If not, check the container logs to evaluate any errors. 

You can also try the url in QGIS. Add a WMS layer, of service http://localhost/data?request=GetCapabilities&service=WMS.

Notice the links to metadata when you open GetCapabilities in a browser.

:::{.callout-note}
In recent years browsers have become more strict, to prevent abuse. For that reason it is important to carefully consider common connectivity aspects, when setting up a new service. Websites running at https can only embed content from other https services, so using https is relevant. [CORS](https://en.wikipedia.org/cors) and [CORB](https://en.wikipedia.org/corb) can limit access to embedded resources from remote servers. Using proper CORS headers and Content type identification, is relevant to prevent CORS and CORB errors. 
:::

GeoDataCrawler uses default (gray) styling for vector and an average classification for grids. You can finetune the styling of layers through the [robot section in index.yml](https://github.com/pvgenuchten/pyGeoDataCrawler?tab=readme-ov-file#layer-styling).

---

## Summary

In this paragraph the standards of Open Geospatial Consortium have been introduced and how you can publish your data according to these standards using Mapserver. In the [next section](./8-measure-quality.md) we'll look at measuring service quality.



