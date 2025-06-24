---
title: Interact with data repositories
author: Paul van Genuchten
date: 2023-05-09
---

In this section a crawler tool is introduced which let's you interact with the metadata in a file based data repository. For this exercise we've prepared a minimal data repository containing a number of Excel-, Shape- and Tiff-files. Unzip the repository to a location on disk. 

In the root folder of the repository already exists a minimal MCF file, `index.yml`. This file contains some generic metadata properties which are used if a file within the repository does not include them. The tool we use is able to inherit metadata properties from this index.yml file through the file hierarchy. 

:::{.callout-tip}
Open index.yml and customise the contact details. Later you will notice that these details will be applied to all datasets which themselves do not provide contact details. 
:::

Consider to add additional index.yml files in other folders to override the values of index.yml at top level.

## Setup environment

The tool we will use is based on Python. For this workshop we offer 2 approaches, if you are familiar with python or are interested to learn more about it, you can setup a conda or python virtual environment and run the scripts in that environment. Else, we recommend to run the python tools in docker containers. Each of the exercises indicates an option to run the Python script directly, or from a container. 

### Conda
 
It has some specific dependencies which are best installed via [Conda](https://conda.io). Conda creates for each project a virtual environment, so any activity will not interfere with the base environment of your machine. Conda manages both the python libraries as well as any other dependencies, such as GDAL. 

If you don't use Conda yet, consider to install [mamba](https://mamba.readthedocs.io/en/latest/), a lighter alternative to conda. Or alternatively run the scripts in a python [virtual environment](https://docs.python.org/3/library/venv.html). 

Now start a commandline or powershell with mamba enabled (or add mamba to your PATH). On windows look for the `Mamba prompt` in start menu. First we will navigate to the folder in which we unzipped the sample data repository. Make sure you are not in the `data` directory but one above.

```bash
cd {path-where-you-unzipped-zipfile}
```

We will create a virtual environment `pgdc` (using Python 3.10) for our project and activate it.

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

Install the crawler tool, [GeoDataCrawler](https://pypi.org/project/geodatacrawler/). The tool is under active development at ISRIC and facilitates many of our data workflows. It is powered by some popular metadata and transformation libraries; [OWSLib](https://github.com/geopython/OWSLib), [pygeometa](https://github.com/geopython/pygeometa) and [GDAL](https://gdal.org). If you want to learn more about these libraries, consider to read the 'tutorials' of these libraries ([OWSlib](https://owslib.readthedocs.io/en/latest/), [pygeometa](https://geopython.github.io/pygeometa/tutorial/), [gdal](https://gdal.org/en/stable/tutorials/index.html)), or enjoy the jupyter based [geopyton workshop](https://github.com/geopython/geopython-workshop).

```
pip install pyGeoDataCrawler
```

Verify the different crawling options by typing:

```
crawl-metadata --help
```

### Python/GDAL via Docker

In case you have difficulties setting up Python with GDAL on your local machine (or just want to try out), an alternative approach is available using Python via Docker. [Docker](https://www.docker.com) is a virtualisation technology which runs isolated containers within your computer. 

- First [install docker](https://docs.docker.com/desktop/install/windows-install/). 
- Start the Docker Desktop tool.
- Now navigate to the folder where you unzipped the data repository and use the Docker image to run the crawler:

```bash
docker run -it --rm pvgenuchten/geodatacrawler crawl-metadata --help
```

For advanced Docker statements there are some differences between Windows commandline, Windows powershell and Linux bash. Use the relevant syntax for your system. 

## Initial MCF files 

The initial task for the tool is to create for every data file in our repository a sidecar file based on embedded metadata from the resource.

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=init --dir=data
```
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=init --dir=tmp/data
```
# Docker & Powershell
```bash
docker run -it --rm -v ${PWD}:/tmp `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=init --dir=/tmp/data
```
:::


Notice that for each resource a {dataset}.yml file has been created. Open a .yml file in a text editor and review the content.
Notice on the docker statements that we mount the local folder into the container, before we can run commands into it. To verify if the correct folder was mounted, run a `ls` command to see the folder contents.

::: {.panel-tabset}
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp pvgenuchten/geodatacrawler ls tmp
```
# Docker & Powershell
```bash
docker run -it --rm -v ${PWD}:/tmp pvgenuchten/geodatacrawler ls /tmp
```
:::

## Update MCF

The `update` mode is meant to be run at intervals, it will update the MCF files if changes have been made on a resource. 

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=update --dir=data
```
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=update --dir=tmp/data
```
# Docker & Powershell
```bash
docker run -it --rm -v ${PWD}:/tmp `
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
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp \
 pvgenuchten/geodatacrawler crawl-metadata \
 --mode=export --dir=tmp/data \
 --dir-out=export --dir-out-mode=flat
```
# Docker & Powershell
```bash
docker run -it --rm -v ${PWD}:/tmp `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=export --dir=/tmp/data `
  --dir-out=/tmp/export --dir-out-mode=flat
```
:::

Open one of the xml files and evaluate if the contact information from step 1 is available.

## Summary

In this paragraph you have been introduced to the pyGeoDataCrawler library. In the [next section](./3-catalog-publication.md) we are looking at catalogue publication.
