---
title: Catalogue publication
author: Paul van Genuchten
date: 2023-05-09
---

Catalogues facilitate data discovery in 3 ways:

- Users can go to the catalogue website and search for data
- Applications such as QGIS and TerriaJS can let users query the catalogue, evaluate the metadata, and directly add the related data to their project
- Search engines crawl public catalogues and include the records in their search results

:::{.callout-note}
An important aspect is proper setup of authorisations for general public, partners and co-workers to access metadata as well as the actual data files behind the metadata. A general rule-of-thumb is that metadata can usually be widely shared, but data services with sensitive content should be properly protected. In some cases organisations even remove the data url from the public metadata, to prevent abuse of those urls. If a resource is not available to all, this can be indicated in metadata as 'access-constraints'.
:::

---

## Catalogue frontend

Various catalogue frontends exist to facilitate dataset search, such as [geonetwork](https://geonetwork-opensource.org), [dataverse](https://dataverse.org), [ckan](https://ckan.org). Selecting a frontend depends on metadata format, target audience, types of data, maintenance aspects, and personal preference.

For this workshop we are going to use [pycsw](https://pycsw.org). It is a catalogue software supporting various standardised query APIs, as well as providing a basic easy-to-adjust html web interface. 

For this exercise we assume you have [docker-desktop](https://www.docker.com/get-started/) installed on your system and running.

pycsw is available as docker image at docker hub, including an embedded SQLite database. In a production situation you will instead use a dedicated Postgres or MariaDB database for record storage. 

- Navigate your shell to the temporary folder containing iso-xml documents. This folder will be mounted into the container, in order to load the records to the pycsw database.

::: {.panel-tabset}
# Linux
```bash
docker run -p 8000:8000 \
   -v $(pwd):/etc/data \
   geopython/pycsw
```
# Powershell
```bash
docker run -p 8000:8000 `
   -v "${PWD}:/etc/data" `
   geopython/pycsw
```
:::

- Visit <http://localhost:8000> 
- Much of the configuration of pycsw (title, contact details, database connection, url) is managed in [a config file](https://github.com/geopython/pycsw/blob/master/docker/pycsw.cfg). Download the file to the current folder, adjust the title and restart docker with:

::: {.panel-tabset}
# Linux
```bash
docker run -p 8000:8000 \
   -d --rm --name=pycsw \
   -v $(pwd):/etc/data \
   -v $(pwd)/pycsw.cfg:/etc/pycsw/pycsw.cfg \
   geopython/pycsw
```
# Powershell
```bash
docker run -p 8000:8000 `
   -d --rm --name=pycsw `
   -v "${PWD}:/etc/data" `
   -v "${PWD}/pycsw.cfg:/etc/pycsw/pycsw.cfg" `
   geopython/pycsw
```
:::

:::{.callout-note}
Notice `-d` starts the docker in the background, so we can interact with the running container. To see which instances are running (in the background) use `docker ps`. `docker logs pycsw` shows the logs of a container and `docker stop pycsw` stops the container. The `-rm` option removes the container at stop, so we can easily recreate it with additional options at next runs.
:::

- For administering the instance we use a utility called `pycsw-admin.py`. Notice on the calls below a reference to a relevant config file. 
- First clear the existing database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py delete-records -c /etc/pycsw/pycsw.cfg
```
# Powershell
```bash
docker exec -it pycsw bash -c "pycsw-admin.py delete-records -c /etc/pycsw/pycsw.cfg"
```
:::

- Notice at <http://localhost:8000/collections/metadata:main/items> that all records are removed.
- Load the records, which we exported as iso19139 in the [previous section](./2-interact-with-data-repositories.md), to the database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py load-records -p /etc/data/export -c /etc/pycsw/pycsw.cfg -y -r
```
# Powershell
```bash
docker exec -it pycsw bash -c `
 "pycsw-admin.py load-records -p /etc/data/export -c /etc/pycsw/pycsw.cfg -y -r"
```
:::

- Validate at http://localhost:8000/collections/metadata:main/items if our records are loaded, else check logs to identify a problem.

---


## Customise the catalogue skin

pycsw uses [jinja templates](https://jinja.palletsprojects.com/en/3.1.x/) to build the web frontend. These are html documents including template language to substitute parts of the page.

- Save the template below as a file 'landing_page.html' in the current directory

```html
{% extends "_base.html" %}
{% block title %}{{ super() }} Home {% endblock %}
{% block body %}
<h1>Welcome to my catalogue!</h1>
<p>{{ config['metadata:main']['identification_abstract'] }}</p>
Continue to the records in this catalogue
<a title="Items" 
    href="{{ config['server']['url'] }}/collections/metadata:main/items">
    Collections</a>, or have a look at the  
<a title="OpenAPI" 
      href="{{ config['server']['url'] }}/openapi?f=html">Open API Document</a>
{% endblock %}
```

- We will now replace the default template in the docker image with our template.

::: {.panel-tabset}
# Linux
```bash
docker run -p 8000:8000 \
   -d --rm --name=pycsw \
   -v $(pwd):/etc/data \
   -v $(pwd)/pycsw.cfg:/etc/pycsw/pycsw.cfg \
   -v $(pwd)/landing_page.html:/etc/pycsw/ogc/api/templates/landing_page.html \
   geopython/pycsw
```
# Powershell
```bash
docker run -p 8000:8000 `
   -d --rm --name=pycsw `
   -v "${PWD}:/etc/data" `
   -v "${PWD}/pycsw.cfg:/etc/pycsw/pycsw.cfg" `
   -v "${PWD}/landing_page.html:/home/pycsw/pycsw/pycsw/ogc/api/templates/landing_page.html" `
   geopython/pycsw
```
:::

- View the result at <http://localhost:8000> 
- Have a look at [the other templates](https://github.com/geopython/pycsw/tree/master/pycsw/ogc/api/templates) in pycsw
- We published a tailored set of templates as a [pycsw skin on github](https://github.com/pvgenuchten/pycsw-skin). This skin has been used as a starting point for the lsc-hubs catalogue skin.


## Summary

In this paragraph you learned how datasets can be published into a catalogue. In the next paragraph, we'll look at [importing metadata from external sources](./4-bulk-import.md).
