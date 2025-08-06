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
An important aspect is proper setup of authorizations for general public, partners and co-workers to access metadata as well as the actual data files behind the metadata. A general rule-of-thumb is that metadata can usually be widely shared, but data services with sensitive content should be properly protected. In some cases organizations even remove the data url from the public metadata, to prevent abuse of those urls. If a resource is not available to all, this can be indicated in metadata as 'access-constraints'.
:::

---

## pycsw catalogue 

Various catalogue frontends exist to facilitate dataset search, such as [Geonetwork OpenSource](https://geonetwork-opensource.org), [dataverse](https://dataverse.org), [CKAN](https://ckan.org). Selecting a frontend depends on metadata format, target audience, types of data, maintenance aspects, and personal preference.

For this workshop we are going to use [pycsw](https://pycsw.org). It is a catalogue software supporting various standardised query APIs, as well as providing a basic easy-to-adjust html web interface. 

For this exercise we assume you have [Docker Desktop](https://www.docker.com/get-started/) installed on your system and running.
Visit the [Docker get started tutorials](https://docs.docker.com/get-started/) in case you're new to docker.

pycsw is available as Docker image at the [github container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry), including an embedded SQLite database. In a production situation you will instead use a dedicated Postgres or MariaDB database for record storage. Notice that when you destroy the container, the SQLite database will be set to its default content. 

Pull and run the pycsw container locally using this command in a command line client (cmd, PowerShell, bash):

```bash
docker run -p 8000:8000 geopython/pycsw@sha256:2eb396798e40dfab3ad4cb839f30231b8c1023db148ed03ffcf3c4e6b388fc7c
```

Open your browser and browse to <http://localhost:8000> to see pycsw in action.

Return to the command line, press ctrl-C to stop the Docker container process.

## Docker Compose

[Compose](https://docs.docker.com/compose/) is a utility of docker, enabling setup of a set of containers using a composition script. A composition script can automate the manual startup operations of the previous paragraph. We've prepared a composition script for this workshop. The script includes, besides the pycsw container, other containers from next paragraphs.

Clone the workshop repository to a local folder (You don't have Git installed? You can also download the repository as a [zip file](https://github.com/pvgenuchten/training-gitops-sdi/archive/refs/heads/main.zip)).

```bash
git clone https://github.com/pvgenuchten/training-gitops-sdi.git
```

On the cloned repository in the `docker` folder there are 2 alternatives:

- [docker-compose.yml](https://github.com/pvgenuchten/training-gitops-sdi/blob/main/docker/docker-compose.yml) is the full orchestration including PostGIS and TerriaJS
- [docker-compose.sqlite.yml](https://github.com/pvgenuchten/training-gitops-sdi/blob/main/docker/docker-compose.sqlite.yml) is a minimal orchestration without terria and based on a file based SQLite database

On both orchestrations a library is used called [Traefik](https://traefik.io) to facilitate 
[path-routing](https://doc.traefik.io/traefik/routing/routers/#path-pathprefix-and-pathregexp) to the relavant containers. 

Also notice that some [layout templates are mounted](https://github.com/pvgenuchten/training-gitops-sdi/blob/0621ba5b8ede4b84a4bd41b5922126e3a02f7b49/docker/docker-compose.yml#L45-L46) into the pycsw container. These templates override the default layout of pycsw.

Some environment variables should be set in a .env file. Rename the `.env-template` file to `.env`.

Then open a shell and navigate to the Docker folder in the cloned repository and run:

```bash
docker compose -f docker-compose.sqlite.yml up
```

A lot of logs are produced by the various containers. You can also run in the background (`-d` or `--detach`) using:

```bash
docker compose -f docker-compose.sqlite.yml up -d
```

When running in the background, use `docker compose down`, `docker ps`, `docker logs pycsw` to stop, see active containers and see the logs of a container. Or interact with the containers from Docker Desktop.

## Load some records

Make sure the Docker setup is running in the background (`-d`), or open a second shell window.

Much of the configuration of pycsw (title, contact details, database connection, url) is managed in [a config file](https://github.com/geopython/pycsw/blob/master/docker/pycsw.yml). You will find a copy of this file in `./docker/pycsw`. In this file, adjust the catalogue title and restart the orchestration. Notice the updated title in your browser.

For administering the contents of the catalogue a utility called `pycsw-admin.py` is available in the pycsw container.
You can either open a shell in the container (via Docker Desktop) and type the commands directly, or use `docker exec` to run the commands from the host.

First clear the existing database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py delete-records -c /etc/pycsw/pycsw.yml
```
# PowerShell
```bash
docker exec -it pycsw bash -c "pycsw-admin.py delete-records -c /etc/pycsw/pycsw.yml"
```
:::

Notice at <http://localhost:8000/collections/metadata:main/items> that all records are removed.

We exported MCF records as iso19139 in the [previous section](./2-interact-with-data-repositories.md).
Copy the ISO XML documents to the `./docker/data/export` folder in the Docker project. This folder will be mounted into the container, so the records can be loaded into the pycsw database.

Use pycsw-admin.py to load the records into the catalogue database:

::: {.panel-tabset}
# Container terminal
```bash
pycsw-admin.py load-records -p /etc/data/export -c /etc/pycsw/pycsw.yml -y -r
```
# PowerShell
```bash
docker exec -it pycsw bash -c `
 "pycsw-admin.py load-records -p /srv/data/export -c /etc/pycsw/pycsw.yml -y -r"
```
:::

Validate at <http://localhost/collections/metadata:main/items> if the records are loaded, else check logs to identify a problem.


## Customise the catalogue skin

pycsw uses [Jinja templates](https://jinja.palletsprojects.com/en/3.1.x/) to build the web frontend. These are HTML documents including template language to substitute parts of the page.

You find 2 template files in `./docker/pycsw/`. Notice in the orchestration file how the files are mounted into the container:

- `templates/landing_page.html` represents the home page of pycsw
- `templates/_base.html` is a main layout template which contains page header, footer and menu and wraps around all other templates

Open a template file and make some changes (colors, text, logos).

Restart the orchestration and view the result at <http://localhost>. 

Have a look at [the other templates](https://github.com/geopython/pycsw/tree/master/pycsw/ogc/api/templates) available in pycsw, which can be tailored in a similar way.

## Summary

In this paragraph you learned how datasets can be published into a catalogue. In the next paragraph, we'll look at [importing metadata from external sources](./4-bulk-import.md).
