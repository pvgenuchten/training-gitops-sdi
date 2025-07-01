---
title: Advanced options
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

## TerriaJS

[TerriaJS](https://terria.io) is a modern Web GIS application, which includes an option to query a CSW catalogue. 

The terriamap Docker image is available via the GitHub Container Registry (ghcr.io)

```bash
docker run -p 3001:3001 ghcr.io/terriajs/terriamap:0.4.2
```

Visit http://localhost:3001 to see TerriaJS in action. 

## TerriaJS within Docker Compose

In our Docker Compose setup, terriamap is routed via [localhost/map](http://localhost/map).

Two files are mounted into the container:

- `config.json` contains the configuration of the interface, you should add a personalised Google/Bing/Cesium key here.
- `simple.json` is the configuration of the inital map (you can add more map configurations in a smilar way). A reference to MapServer and pycsw are included.
- In `index.html` the title/abstract of the page can be customized. Also, because TerriaJS is hosted at `/map`, the base url is set to `/map`.

Notice that from terria layer details, you can access the record in the catalogue (link is shared via WMS Capabilities)

## Summary

With this topic we conclude our training on data management. We hope you enjoyed the materials. Notice that the training can act as a starting point to a number of other resources. Let us know via [GitHub issues](https://github.com/lsc-hubs/hub-core/issues) if you have improvement suggestions for the materials.
