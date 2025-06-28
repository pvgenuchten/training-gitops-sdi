---
title: Data publication
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

## Introduction

In order to share a dataset with colleagues, partners or the wider public. The file should be published in a shared environment. Various technologies are available to share a file on a network. To select a relevant location mainly depends on which type of users are going to access the dataset.

The following options exist:

- A data repository such as [Zenodo](https://zenodo.org/) or [Dataverse](https://dataverse.org). With this option metadata of the resource is automatically collected and searchable. 
- A cloud service such as Google Drive, Microsoft Sharepoint, Dropbox, Amazon Webservices, GitHub. Such a service can also be setup locally. A minimal solution would be to set up a Webdav service.
- A shared folder on a central server on the local intranet. Notice that this location is usually not available by remote partners. 


## Persistent identification

It is important that datasets made available for reuse remain available at a persistent location. So any documentation, which reference the dataset as a source, doesn't break. Repositories typically provide a persistent identification layer between the deposited dataset and the users (such as [DOI](https://doi.org) or [ePIC](http://www.pidconsortium.net)). In case a file is moved, providers can update the DOI reference to the new location.


## Include metadata

For optimal discoverability, it is important to combine data publication with metadata. Either via embedded metadata in the file, else with a separate metadata file. In case of a shared folder or cloud service, embed or place the metadata side by side with the data files, so people browsing through the system can easily find it.

The embedded or sidecar metadata can be ingested by catalogue software, to make it searchable for the targeted audience. This process is further described at [catalogue publication](./3-catalogue-publication.md).


## Summary

Various technologies exist to share data on a network. When selecting a mechanism, evaluate if you can facilitate identifier persistence and share metadata along with the files. In the next section we'll setup [convenience APIs](./7-providing-map-services.md) on data to facilitate reuse of the data.
