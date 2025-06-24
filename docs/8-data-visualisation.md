---
title: Advanced options
author: Paul van Genuchten
date: 2023-05-09
---

## TerriaJS

[TerriaJS](https://terria.io) is a modern web gis application, which includes an option to query a CSW catalogue. 

The terriamap docker image is available via Github ContainerRegistry (ghcr.io)

```
docker run -p 3001:3001 ghcr.io/terriajs/terriamap:0.4.2
```

Visit http://localhost:3001 to see TerriaJS in action. 

In our docker compose terriamap is routed via [localhost/map](http://localhost/map).

Two files are mounted into the container:

- config.json contains the configuration of the interface, you should add a personalised google/bing/cesium key here.
- simple.json is the configuration of the inital map (you can add more map configurations in a smilar way). A reference to mapserver and pycsw are included.

Notice that from terria layer details, you can access the catalogue again (link is shared via WMS-getcapabilities)

--- 

## Summary

With this topic we conclude our training on data management. We hope you enjoyed the materials. Notice that the training can act as a starting point to a number of other resources. Let us know via [Git issues](https://github.com/lsc-hubs/hub-core/issues) if you have improvement suggestions for the materials.
