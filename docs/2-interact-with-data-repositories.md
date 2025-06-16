---
title: Interact with data repositories
author: Paul van Genuchten
date: 2023-05-09
---

In this section a crawler tool is introduced which let's you interact with the metadata in a file based data repository. For this exercise we've prepared a minimal data repository containing a number of Excel-, Shape- and Tiff-files. Unzip the repository to a location on disk. 

In the root folder of the repository already exists a minimal mcf file, `index.yml`. This file contains some generic metadata properties which are used if a file within the repository does not include them. The tool we use is able to inherit metadata properties from this index.yml file through the file hierarchy. 

:::{.callout-tip}
Open index.yml and customise the contact details. Later you will notice that these details will be applied to all datasets which themselves do not provide contact details. 
:::

Consider to add additional index.yml files in other folders to override the values of index.yml at top level.

## Setup environment

The tool we will use is based on python. It has some specific dependencies which are best installed via [Conda](https://conda.io). Conda creates a virtual python environment, so any activity will not interfere with the base python environment of your machine. Notice that the [next paragraph](#pythongdal-via-docker) describes an alternative approach without installing python locally.

If you don't have Conda, we suggest to install [mamba](https://mamba.readthedocs.io/en/latest/), or alternatively run the scripts from a python docker container. Each of the exercises below indicates an option to run the python script directly, or from a container. 

Now start a commandline or powershell with mamba or docker enabled (or add mamba to your PATH). On windows look for the `Mamba prompt` in start menu.
First we will navigate to the folder in which we unzipped the sample data repository. Make sure you are not in the `data` directory but one above.

```bash
cd {path-where-you-unzipped-zipfile}
```

### Setup in mamba

For mamba, we will create a virtual environment `pgdc` (using Python 3.10) for our project and activate it.

```bash
manba create --name pgdc python=3.10 
mamba activate pgdc
```

Notice that you can deactivate this environment with: `mamba deactivate` and you will return to the main Python environment. The tools we will install below, will not be available in the main environment.

Install the dependencies for the tool:

```
mamba install -c conda-forge gdal
mamba install -c conda-forge pysqlite3
```

Now we will install the crawler tool, [GeoDataCrawler](https://pypi.org/project/geodatacrawler/). The tool is under active development at ISRIC and facilitates many of our data workflows. It is powered by some popular metadata and transformation libraries; [OWSLib](https://github.com/geopython/OWSLib), [pygeometa](https://github.com/geopython/pygeometa) and [GDAL](https://gdal.org).

```
pip install geodatacrawler
```

Verify the different crawling options by typing:

```
crawl-metadata --help
```

### Python/GDAL via Docker

In case you have difficulties setting up python with gdal on your local machine (or just want to try out), an alternative approach is available, using python via Docker. [Docker](https://www.docker.com) is a virtualisation technology which runs isolated containers within your computer. 

- First [install docker](https://docs.docker.com/desktop/install/windows-install/). 
- Start the docker desktop tool.
- Now navigate to the folder where you unzipped the data repository and use the docker image to run the crawler:

```bash
docker run -it --rm pvgenuchten/geodatacrawler crawl-metadata --help
```

For advanced docker statements there are some differences between Windows commandline, Windows powershell and Linux. Use the relevant syntax for your system. 

## Initial MCF files 

The initial task for the tool is to create for every data file in our repository a sidecar file based on embedded metadata from the resource.

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=init --dir=data
```
# Dckr & Linux
```bash
docker run -it --rm -v$(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=init --dir=tmp/data
```
# Dckr & Powershell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=init --dir=/tmp/data
```
:::


Notice that for each resource a {dataset}.yml file has been created. Open a .yml file in a text editor and review the content.

## Update MCF

The `update` mode is meant to be run at intervals, it will update the mcf files if changes have been made on a resource. 

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=update --dir=data
```
# Dckr & Linux
```bash
docker run -it --rm -v$(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=update --dir=tmp/data
```
# Dckr & Powershell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=update --dir=/tmp/data
```
:::

In certain cases the update mode will also import metadata from remote url's. This happens for example if the dataset-uri is a [DOI](https://www.doi.org/the-identifier/what-is-a-doi/). The update mode will ten fetch metadata of the DOI and push it into the MCF. 

## Export MCF

Finally we want to export the MCF's to actual iso19139 metadata to be loaded into a catalogue like pycsw, GeoNetwork, CKAN etc.

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=export --dir=data --dir-out=export --dir-out-mode=flat
```
# Dckr & Linux
```bash
docker run -it --rm -v$(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=export --dir=tmp/data \
 --dir-out=export --dir-out-mode=flat
```
# Dckr & Powershell
```bash
docker run -it --rm -v "${PWD}:/tmp" `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=export --dir=/tmp/data `
  --dir-out=/tmp/export --dir-out-mode=flat
```
:::

Open one of the xml files and evaluate if the contact information from step 1 is available.

## Summary

In this paragraph you have been introduced to the geodatacrawler library. In the [next section](./3-catalog-publication.md) we are looking at catalogue publication.
