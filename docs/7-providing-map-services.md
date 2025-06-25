---
title: Providing convenience APIs
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

For spatial datasets it is of interest to share them via convenience APIs, so the datasets can be downloaded in parts or easily be visualised in common tools such as [QGIS](https://qgis.org), [OpenLayers](https://openlayers.org) & [Leaflet](https://leaflet.org). The standards on data services of the [Open Geospatial Consortium](https://www.ogc.org/) are designed with this purpose. These APIs give direct access to subsets or map visualisations of a dataset. 
 
In this paragraph you will be introduced to various standardised APIs, after which we introduce an approach to publish datasets, which builds on the data management approach introduced in the previous paragraphs.  

---

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

---

## Setting up an API

[MapServer](https://mapserver.org) is server software which is able to expose datasets through various APIs. 
Examples of similar software are [QGIS server](https://docs.qgis.org/3.28/en/docs/server_manual/introduction.html), 
[ArcGIS Server](https://enterprise.arcgis.com/en/server/), [GeoServer](https://geoserver.org) and 
[pygeoapi](https://pygeoapi.io).
 
We've selected mapserver for this training, because of its robustness and low resource consumption.
MapServer is configured using a configuration file: called the [mapfile](https://www.mapserver.org/mapfile/). 
The mapfile defines metadata for the dataset and how users interact with the dataset, mainly the colour 
scheme (legend) to draw a map of the dataset.  

Various tools exist to write these configuration files, such as [MapServer studio](https://mapserverstudio.net/), 
[GeoStyler](https://www.osgeo.org/projects/geostyler/), [QGIS Bridge](https://www.geocat.net/docs/bridge/qgis/latest), 
up to a [Visual Studio plugin to edit mapfiles](https://marketplace.visualstudio.com/items?itemName=chicoff.mapfile).

The [pyGeoDataCrawler](https://pypi.org/project/geodatacrawler/), introduced in a 
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
    title: My new map service
    abstract: A map service for data about ...
contact:
  pointOfContact:
    organization: ISRIC
    email: info@isric.org
    url: https://www.isric.org
```

- Set the environment variables in the `.env file`; `pgdc_md_url`,`pgdc_ms_url`,`pgdc_webdav_url="https://example.com/data"`

- Generate the mapfile

::: {.panel-tabset}
# Local
```bash
crawl-maps --dir=.
```
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp \
  pvgenuchten/geodatacrawler crawl-maps --dir=/tmp 
```
# Docker & PowerShell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  pvgenuchten/geodatacrawler crawl-maps --dir=/tmp 
```
:::

- Index.yml may include a "robot" property, to guide the crawler in how to process the folder. This section can be used to add specific crawling behaviour.

```yaml
mcf:
    version: 1.0
robot:
    skip-subfolders: True # indicates the crawler not to proceed in subfolders
```

---

## MapServer via Docker 

For this exercise we're using a [mapserver image](https://hub.docker.com/r/camptocamp/mapserver) provided by Camp to Camp available from DockerHub.

```bash
docker pull camptocamp/mapserver:8.4  
```

First update the config file, which is [mounted as a volume](https://docs.docker.com/storage/volumes/) into the container. On this config file we will list all the mapfiles we aim to publish on our container. Open the file `./data/ms.conf` and populate the maps section. You may have to move all content of the data repository to the docker folder.

```yaml
MAPS
     "data" "/srv/data/data.map"
END
```

Notice that our local `./docker/data` folder is mounted into the mapserver container as `/srv/data`. 

Run or restart the docker compose.

Check http://localhost/ows/data/ogcapi in your browser. If all has been set up fine it should show the OGCAPI homepage of the service. If not, check the container logs to evaluate any errors. 

You can also try the url in QGIS. Add a WMS layer, of service http://localhost/ows/data?request=GetCapabilities&service=WMS.

GeoDataCrawler uses default (gray) styling for vector and an average classification for grids. You can finetune the styling of layers through the [robot section in index.yml](https://github.com/pvgenuchten/pyGeoDataCrawler?tab=readme-ov-file#layer-styling) or by providing an [Styled Layer Descriptor](https://www.ogc.org/standards/sld/) (SLD) file for a layer, as `{name}.sld`. Sld files can be created using QGIS (export style as SLD).

---

## Summary

In this paragraph the standards of Open Geospatial Consortium have been introduced and how you can publish your data according to these standards using MapServer. In the [next section](./8-data-visualisation.md) we'll look at measuring service quality.



