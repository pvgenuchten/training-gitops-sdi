---
title: Advanced options
author: Paul van Genuchten
date: 2023-05-09
---

Various extensions are possible to tailor the system to your organisation needs.

## TerriaJS

[TerriaJS](https://terria.io) is a modern web gis application, which includes a widget to query a catalogue. From the catalogue search results the data can be added to the TerriaJS map.

The main [Docker image definition](https://github.com/TerriaJS/TerriaMap/blob/main/deploy/docker/Dockerfile) can be used to build and run terriaJS locally.

```
git clone https://github.com/TerriaJS/TerriaMap
cd TerriaMap
docker build -t local/terria .
docker run -p 3001:3001 local/terria
```

Visit http://localhost:3001 to see TerriaJS in action. 

--- 

## Summary

With this topic we conclude our training on data management. We hope you enjoyed the materials. Notice that the training can act as a starting point to a number of other resources. Let us know via [Git issues](https://github.com/lsc-hubs/hub-core/issues) if you have improvement suggestions for the materials.
