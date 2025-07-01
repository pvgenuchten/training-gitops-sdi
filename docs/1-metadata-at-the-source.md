---
title: Metadata at the source
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

## Introduction

Many organizations organise their documents and datasets at a central network location or database. These resources are usually clustered in organizational units, projects and/or years. Some files and database tables in that central network location contain embedded metadata, such as the name, size, date, author, location etc. This information supports users in understanding the context of the data source. Especially if that data at some point is migrated from its original context.

## Sidecar metadata

For those formats which lack embedded metadata, or in order to capture additional metadata aspects, you may consider creating a `sidecar` metadata file for every resource. A dedicated metadata file sharing the name of the datasource. This approach is for example common in the Esri community, where a `.shp.xml` is created alongside any `.shp` file, which captures relevant metadata elements.

:::{.callout-tip}
Locate on your local computer or network drive a random shapefile. Does the file have a .shp.xml sidecar file? Else find another shape or tiff file (look for `*.shp.xml`). The contents of the xml file may be very minimal, but in most cases at least some processing information and the data model of the shapefile are mentioned. 
:::

**Through the embedded metadata and sidecar concept, we endorse data scientists to document their data at the source. Since the data producers are often best informed how the data is produced and how it should be used.** 

## Standards and interoperability

For optimal interoperability, it is important to agree within your group on the metadata standard(s) to use in sidecar files. Esri software for example provides an option to select the model of the metadata as documented in the [ArcGIS Pro documentation]](https://pro.arcgis.com/en/pro-app/latest/help/metadata/create-iso-19115-and-iso-19139-metadata.htm). QGIS has various plugins, such as [GeoCat Bridge](https://plugins.qgis.org/plugins/geocatbridge/), to work with various metadata models.

:::{.callout-tip}
Does your organization or community endorse a metadata model to describe data sources?
Are you aware of tooling which can support you in creation of metadata in this model?
:::

## Getting started

Within the [geopython community](https://geopython.github.io), the [pygeometa](https://geopython.github.io/pygeometa) library provides a metadata format called the [metadata control file](https://geopython.github.io/pygeometa/reference/mcf) (MCF). The aim of MCF is ease of use, while providing export options to various metadata models. Many metadata models are based on XML, which makes them quite challenging to read by humans. MCF is based on [YAML](https://www.yaml.io/spec/), a text-based format using indents to group elements. In this workshop we are using the MCF format for its simplicity and natural fit with the use cases. A minimal sample of an MCF is:

```yaml
mcf:
    version: 1.0

metadata:
    identifier: 9c36a048-4d28-453f-9373-94c90e101ebe
    hierarchylevel: dataset
    date: 2023-05-10

identification:
    title: My favourite dataset
    abstract: A sample dataset record to highlight the options of MCF
    ...
```

If you are comfortable with Python, consider to try the following experiment.

:::{.callout-tip}
Save the above file as `md.yml`. Then open a shell and set up a virtual Python (or COnda) environment, then:

```bash
pip3 install pygeometa
pygeometa metadata info path/to/md.yml
pygeometa metadata generate path/to/md.yml --schema=iso19139 --output=md.xml
```
:::

Read more about pygeometa at <https://geopython.github.io/pygeometa/tutorial/>.

When describing a resource, consider which user groups are expected to read the information. This analyses will likely impact the style of writing in the metadata. The UK Geospatial Commission has published some [practical recommendations](https://www.gov.uk/government/publications/search-engine-optimisation-for-publishers-best-practice-guide) on this topic.

When tagging the dataset with keywords, preferably use keywords from controlled vocabularies like Agrovoc, Eurovoc, etc. A benefit of controlled vocabularies is that the term is not ambigue and it can be processed in multiple languages. 

## MCF editing

MCF documents can be written in a text editor like [Visual Studio Code](https://code.visualstudio.com). Consider to install the [YAML plugin](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) for instant YAML validation. 

Another option to create and update mcf files is via [MDME](https://github.com/osgeo/mdme). MDME is a web based software package providing a dynamic metadata edit form. An operational package is available at [osgeo.github.io](https://osgeo.github.io/mdme). Notice that if you install the package locally, you can customize the metadata model to your organizational needs.

:::{.callout-tip}
Imagine a dataset you have recently worked with. Then open [mdme](https://osgeo.github.io/mdme) and populate the form, describing that dataset. Now save the MCF file so we can later place it in a sample data repository. 

Notice that MDME also offers capabilities to export directly as iso19139, it uses a webservice based on the tools used in this workshop.
:::

## Summary

In this section, you are introduced to a data management approach which maintains metadata at the location where the datasets are maintained, using a minimal, standards complient approach. You are introduced to the MCF metadata format. In the [next section](./2-interact-with-data-repositories.md), we will go into more detail on interacting with the MCF format.
