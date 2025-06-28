---
title: Providing convenience APIs
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

## Introduction

For spatial datasets it is of interest to also share them via convenience APIs, so the datasets can be downloaded in parts or easily be visualised in common tools such as [QGIS](https://qgis.org), [OpenLayers](https://openlayers.org) or [Leaflet](https://leaflet.org). The standards on data services of the [Open Geospatial Consortium](https://www.ogc.org/) are designed with this purpose. These APIs give direct access to subsets or map visualisations of a dataset. 
 
In this paragraph you will be introduced to various standardised APIs, after which we introduce an approach to publish datasets, which builds on the data management approach introduced in the previous paragraphs. 

These days novel ways to share data over the web arrive, where the data formats itself allow requesting subsets of the data, enabling efficient consumption of the data straight from a repository or cloud storage service, the Cloud Optimized GeoTiff ([COG](https://cogeo.org/)) and [GeoZarr](https://github.com/zarr-developers/geozarr-spec) formats for grid data and for vector data there is [GeoParquet](https://geoparquet.org/).


## Standardised data APIs 

Standardised mapping APIs, such as Web Map Service (WMS), Web Feature service (WFS) and Web Coverage Service (WCS), originate from the beginning of this century. In recent years several challenges have been identified around these standards, which led to a series of [Spatial Data on the Web Best Practices](https://www.w3.org/TR/sdw-bp/). Combined with the [OGC Open Geospatial APIs - White Paper](https://docs.ogc.org/wp/16-019r4/16-019r4.html), OGC then initiated a new generation of standards based on these best practices.

An overview of both generations:

| OWS | OGC-API | Description |
| --- | --- | --- |
| Web Map Service ([WMS](https://www.ogc.org/standard/wms/)) | [Maps](https://ogcapi.ogc.org/maps/) | Provides a visualisation of a subset of the data |
| Web Feature Service ([WFS](https://www.ogc.org/standard/wfs/)) | [Features](https://ogcapi.ogc.org/features/) | API to request a subset of the vector features |
| Web Coverage Service ([WCS](https://www.ogc.org/standard/wcs/)) | [Coverages](https://ogcapi.ogc.org/coverages/) | API to interact with grid sources |
| Sensor Observation Service ([SOS](https://www.ogc.org/standard/sos)) | [SensorThings](https://www.ogc.org/standard/sensorthings/) | Retrieve subsets of sensor observations |
| Web Processing Service ([WPS](https://www.ogc.org/standard/wps)) | [Processes](https://ogcapi.ogc.org/processes) | Run processes on data ]
| Catalogue Service for the Web ([CSW](https://www.ogc.org/standard/cat)) | [Records](https://ogcapi.ogc.org/records) | Retrieve and filter catalogue records |

Notice that most of the mapping software supports the standards of both generations. However, due to their recent
introduction, expect incidental challenges in the implementations of OGC APIs. 


## Setting up an API

[MapServer](https://mapserver.org) is server software which is able to expose datasets through various APIs. 
Examples of similar software are [QGIS server](https://docs.qgis.org/latest/en/docs/server_manual/index.html), 
[ArcGIS Server](https://enterprise.arcgis.com/en/server/), [GeoServer](https://geoserver.org) and 
[pygeoapi](https://pygeoapi.io).
 
We've selected mapserver for this training, because of its robustness, ease of configuration and low resource consumption.
MapServer is configured using a configuration file: called the [mapfile](https://www.mapserver.org/mapfile/). 
The mapfile defines metadata for the dataset and how users interact with the dataset, mainly the colour 
scheme (legend) to draw a map of a dataset.  

Various tools exist to write these configuration files, such as [MapServer studio](https://mapserverstudio.net/), [QGIS Bridge](https://geocat.github.io/qgis-bridge-plugin/latest/server_configuration.html#mapserver), 
up to a [Visual Studio plugin to edit mapfiles](https://marketplace.visualstudio.com/items?itemName=chicoff.mapfile).

The [pyGeoDataCrawler](https://pypi.org/project/geodatacrawler/), introduced in a 
[previous paragraph](./2-interact-with-data-repositories.md), also has an option to generate mapfiles. 
A big advantage of this approach is the integration with existing metadata. 
GeoDataCrawler will, during mapfile generation, use the existing metadata, but also update the metadata 
so it includes a link to the mapserver service endpoint. This toolset enables a typical workflow of: 

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
    title: My new map service
    abstract: A map service for data about ...
contact:
  pointOfContact:
    organization: example
    email: info@example.com
    url: https://www.example.com
```

- Set some environment variables in the `.env` file; `pgdc_md_url`,`pgdc_ms_url`,`pgdc_webdav_url`

- Generate the mapfile

::: {.panel-tabset}
# Local
```bash
crawl-maps --dir=.
```
# Docker & Linux
```bash
cd ./docker/
docker run -it --rm -v $(pwd):/tmp \
  pvgenuchten/geodatacrawler crawl-maps --dir=/tmp/data 
```
# Docker & PowerShell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  pvgenuchten/geodatacrawler crawl-maps --dir=/tmp/data 
```
:::

Test your mapserver configuration. The mapserver container includes a test tool for this purpose.
With the docker composition running, try:

::: {.panel-tabset}
# Local
```bash
map2img 
```
# Docker & Linux
```bash
docker exec mapserver map2img -m /srv/data/data/data.map \
  -l cities -o /srv/data/data/test.png
```

# Docker & PowerShell
```bash
docker exec mapserver map2img -m /srv/data/data/data.map `
  -l cities -o /srv/data/data/test.png
```
:::

Replace -l (layer) for a layer in your mapfile. Notice a file `test.png` being written to the data folder.


## MapServer via Docker 

For this workshop we're using a [mapserver image](https://hub.docker.com/r/camptocamp/mapserver) provided by Camp to Camp available from [Docker Hub](https://hub.docker.com/).

```bash
docker pull camptocamp/mapserver:8.4  
```

First update the config file `./data/ms.conf`. On this config file list all the mapfiles wihich are published on the container. Open the file `./data/ms.conf` and populate the maps section. The maps section are key-value pairs of alias and path to the mapfile, the alias is used as http://localhost/ows/{alias}/ogcapi (for longtime mapserver users, the alias replaces the `?map=example.map` syntax).

Notice that our local `./docker/data` folder is mounted into the mapserver container as `/srv/data`. 
You may have to move all content of the data repository to the ./docker/data folder. 

```yaml
MAPS
     "data" "/srv/data/data.map"
END
```

Run or restart the docker compose.

Check http://localhost/ows/data/ogcapi in your browser. If all has been set up fine it should show the OGCAPI homepage of the service. If not, check the container logs to evaluate any errors. 

You can also try the url in QGIS. Add a WMS layer, of service http://localhost/ows/data?request=GetCapabilities&service=WMS.

GeoDataCrawler uses default (gray) styling for vector and an average classification for grids. You can finetune the styling of layers through the [robot section in index.yml](https://github.com/pvgenuchten/pyGeoDataCrawler?tab=readme-ov-file#layer-styling) or by providing an [Styled Layer Descriptor](https://www.ogc.org/standards/sld/) (SLD) file for a layer, as `{name}.sld`. Sld files can be created using QGIS (export style as SLD).


## Summary

In this paragraph the standards of Open Geospatial Consortium have been introduced and how you can publish your data according to these standards using MapServer. In the [next section](./8-data-visualisation.md) we'll look at measuring service quality.



