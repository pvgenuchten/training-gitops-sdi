---
title: GitOps SDI
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

This tutorial presents a workshop on setting up a Spatial Data Infrastructure ([SDI](https://en.wikipedia.org/wiki/Spatial_data_infrastructure)) using [GitOps](https://www.opsmx.com/what-is-gitops) concepts. 
GitOps principles aim to replace manual steps for setting up and maintaining hardware and software by introducing automated delivery pipelines from versioned Git configurations. 
During the workshop we introduce you to the concepts of GitOps as well as a number of OSGeo tools.

SDIs are typically built as nodes, focused on a (spatial) data repository, a catalogue, a data access layer (APIs) and data visualization components.  GitOps concepts greatly help SDI development and implementation in the following ways:

- Containerization
- Reproducible and portable setup and deployment
- Low barrier access to modern tools for GIS, Earth observation, climate, Earth system prediction, and other data sciences

The SDI introduced in this tutorial is configured from scripts stored in Git and is delivered using a set
of Docker containers running locally, on a virtual machine (VM) or in a cloud infrastructure.

This tutorial introduces the following OSGeo tools:

- PyGeoDataCrawler (pygeometa/OWSLib/Mappyfile/GDAL)
- pycsw
- MapServer
- TerriaJS

Git and additional services provided by the common Git service providers (GitHub, GitLab, Bitbucket, ...) have
a central role in the training. 
They are, for example, suggested to facilitate software development and delivery,
content co-creation and management, as well as community feedback.

Experience with Git and Docker are required before starting this workshop. 

In the [first paragraph](./1-metadata-at-the-source.md) we will introduce you to our approach to integrated
metadata management, the core of any SDI. You can also access the [slides](./slides/) of the workshop.

We hope you enjoy the materials, 

Paul and Tom.
