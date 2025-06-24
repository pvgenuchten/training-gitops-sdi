---
title: Catalogue publication
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

Catalogues facilitate data discovery in 3 ways:

- Users can go to the catalogue website and search for data
- Applications such as QGIS and TerriaJS can let users query the catalogue, evaluate the metadata, and directly add the related data to their project
- Search engines and partner catalogues crawl the catalogue and include the records in their search results

:::{.callout-note}
An important aspect is proper setup of authorisations for general public, partners and co-workers to access metadata as well as the actual data files behind the metadata. A general rule-of-thumb is that metadata can usually be widely shared, but data services with sensitive content should be properly protected. In some cases organisations even remove the data url from the public metadata, to prevent abuse of those urls. If a resource is not available to all, this can be indicated in metadata as 'access-constraints'.
:::

---

## pycsw catalogue 

Various catalogue frontends exist to facilitate dataset search, such as [Geonetwork OpenSource](https://geonetwork-opensource.org), [dataverse](https://dataverse.org), [CKAN](https://ckan.org). Selecting a frontend depends on metadata format, target audience, types of data, maintenance aspects, and personal preference.

For this workshop we are going to use [pycsw](https://pycsw.org). It is a catalogue software supporting various standardised query APIs, as well as providing a basic easy-to-adjust html web interface. 

For this exercise we assume you have [Docker Desktop](https://www.docker.com/get-started/) installed on your system and running.

pycsw is available as Docker image at DockerHub, including an embedded SQLite database. In a production situation you will instead use a dedicated Postgres or MariaDB database for record storage. 

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
- Much of the configuration of pycsw (title, contact details, database connection, url) is managed in [a config file](https://github.com/geopython/pycsw/blob/master/docker/pycsw.yml). Download the file to the current folder, adjust the title and restart docker with:

::: {.panel-tabset}
# Linux
```bash
docker run -p 8000:8000 \
   -d --rm --name=pycsw \
   -v $(pwd):/etc/data \
   -v $(pwd)/pycsw.cfg:/etc/pycsw/pycsw.yml \
   geopython/pycsw
```
# Powershell
```bash
docker run -p 8000:8000 `
   -d --rm --name=pycsw `
   -v "${PWD}:/etc/data" `
   -v "${PWD}/pycsw.cfg:/etc/pycsw/pycsw.yml" `
   geopython/pycsw
```
:::

:::{.callout-note}
Notice `-d` starts the Docker in the background, so we can interact with the running container. To see which instances are running (in the background) use `docker ps`. `docker logs pycsw` shows the logs of a container and `docker stop pycsw` stops the container. The `-rm` option removes the container at stop, so we can easily recreate it with additional options at next runs.
:::

- For administering the instance we use a utility called `pycsw-admin.py`. Notice on the calls below a reference to a relevant config file. 
- First clear the existing database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py delete-records -c /etc/pycsw/pycsw.yml
```
# Powershell
```bash
docker exec -it pycsw bash -c "pycsw-admin.py delete-records -c /etc/pycsw/pycsw.yml"
```
:::

- Notice at <http://localhost:8000/collections/metadata:main/items> that all records are removed.
- Load the records, which we exported as iso19139 in the [previous section](./2-interact-with-data-repositories.md), to the database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py load-records -p /etc/data/export -c /etc/pycsw/pycsw.yml -y -r
```
# Powershell
```bash
docker exec -it pycsw bash -c `
 "pycsw-admin.py load-records -p /etc/data/export -c /etc/pycsw/pycsw.yml -y -r"
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

- We will now replace the default template in the Docker image with our template.

::: {.panel-tabset}
# Linux
```bash
docker run -p 8000:8000 \
   -d --rm --name=pycsw \
   -v $(pwd):/etc/data \
   -v $(pwd)/pycsw.yml:/etc/pycsw/pycsw.yml \
   -v $(pwd)/landing_page.html:/etc/pycsw/ogc/api/templates/landing_page.html \
   geopython/pycsw
```
# Powershell
```bash
docker run -p 8000:8000 `
   -d --rm --name=pycsw `
   -v "${PWD}:/etc/data" `
   -v "${PWD}/pycsw.yml:/etc/pycsw/pycsw.yml" `
   -v "${PWD}/landing_page.html:/usr/local/lib/python3.10/site-packages/pycsw/ogc/api/templates/landing_page.html" `
   geopython/pycsw
```
:::

- View the result at <http://localhost:8000> 
- Have a look at [the other templates](https://github.com/geopython/pycsw/tree/master/pycsw/ogc/api/templates) in pycsw
- We published a tailored set of templates as a [pycsw skin on GitHub](https://github.com/pvgenuchten/pycsw-skin). This skin has been used as a starting point for the lsc-hubs catalogue skin.

## SDI setup using docker compose

[Compose](https://docs.docker.com/compose/) is a utility of docker, enabling setup of a set of containers using a composition script.
A composition script can automate the manual operations of the previous paragraph. We've prepared a composition script for this workshop. The script includes, besides the pycsw container, other containers from next paragraphs.

Clone the repository to a local folder (You don't have git installed? You can also download the repository as a [zip file](https://github.com/pvgenuchten/training-gitops-sdi/archive/refs/heads/main.zip)).

```
git clone https://github.com/pvgenuchten/training-gitops-sdi.git
```

On the cloned repository in the `docker` folder there are 2 alternatives:

- [docker-compose.yml](https://github.com/pvgenuchten/training-gitops-sdi/blob/main/docker/docker-compose.yml) is the full orchestration including postgis and terria
- [docker-compose-sqlite.yml](https://github.com/pvgenuchten/training-gitops-sdi/blob/main/docker/docker-compose-sqlite.yml) is a minimal orchestration without terria and based on a file based sqlite database

On both orchestrations a library is used called [traefik](https://traefik.io) to facilitate 
[path-routing](https://doc.traefik.io/traefik/routing/routers/#path-pathprefix-and-pathregexp) to the relavant containers. 

Also notice that some [layout templates are mounted](https://github.com/pvgenuchten/training-gitops-sdi/blob/0621ba5b8ede4b84a4bd41b5922126e3a02f7b49/docker/docker-compose.yml#L45-L46) into the pycsw container.

Some environment variables should be set in a .env file. Rename the `.env-template` file to `.env`.

Then open a shell and navigate to the docker folder in the cloned repository and run:

```bash
docker compose -f docker-compose-sqlite.yml up
```

A lot of logs are produced by the various containers. You can also run in the background using:

```bash
docker compose -d -f docker-compose-sqlite.yml up
```

When running in the background, use `docker compose down`, `docker ps`, `docker logs pycsw` to stop, see active containers and see the logs of a container. Or interact with the containers from docker desktop.

You can now use `pycsw-admin.py` in a similar way as above to load records into the catalogue.

## Summary

In this paragraph you learned how datasets can be published into a catalogue. In the next paragraph, we'll look at [importing metadata from external sources](./4-bulk-import.md).
