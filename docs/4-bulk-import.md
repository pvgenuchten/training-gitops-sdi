---
title: Bulk import
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

This paragraph describes approaches to import metadata from existing repositories. Including an option to import metadata from records of a spreadsheet.

## Bulk import from a spreadsheet

Many metadata initiatives tend to start from a spreadsheet. Each of the columns representa a metadata property and the rows are the individual records describing a resource. Spreadsheets have proven to be an effective medium to populate a catalogue with records initially. To facilitate this use case the pyGeoDataCrawler software provides an `import spreadsheet` method. The spreadsheet is parsed and a MCF document is generated for every row.

Since every metadata initiative tends to have dedicated columns. A templating approach is used to convert from row to MCF. A default template is available, matching a default spreadsheet layout. If your spreadsheet layout is different, you need to adjust the template accordingly. 

- For this exercise we'll use index.csv and index.j2 file in .data/csv. Notice that the template has the same filename, but with extension `.j2`. Navigate to the folder called `csv`, in your working directory.
- From your shell environment run this command:

::: {.panel-tabset}
# Local
```bash
crawl-metadata --mode=import-csv --dir="./csv"
```
# Docker & Linux
```bash
docker run -it --rm -v $(pwd):/tmp \
  pvgenuchten/geodatacrawler crawl-metadata \
  --mode=import-csv --dir="/tmp" --sep=";"
```
# Docker & PowerShell
```bash
docker run -it --rm -v ${PWD}:/tmp `
  pvgenuchten/geodatacrawler crawl-metadata `
  --mode=import-csv --dir="/tmp" --sep=";"
```
:::


- If there are errors, check the paths and consider to open the CSV in Google Sheets and export it again or open it in a text editor to look for special cases. A known issue with this approach is that the crawler tool can not manage `newline` characters in text fields.
- Open one of the generated MCF files to evaluate its content.
- A common spreadsheet tool is [Microsoft Excel](https://www.microsoft.com/en-gb/microsoft-365/excel). If you open and export a spreadsheet from Excel, the CSV will use the ';' character as column separator. Use the --sep=';' parameter to indicate pyGeoDataCrawler to use this separator.


## Bulk import from an online location

Many resources are already described elsewhere which may be of interest to add to our catalogue. For this use case some options exist to import remote metadata. 

In case you want to harvest the full set of a remote catalogue, you can create a basic MCF in a new folder `undrr` and add a distribution of a CSW endpoint.

```
metadata:
    hierarchylevel: service
    identifier: riskprofilesundrr
distribution:
  csw:
    name: csw
    url: http://riskprofilesundrr.org/catalogue/csw
    type: OGC:CSW
```

Now use the crawler to fetch the remote records of the catalogue.

```
crawl-metadata --mode=update --dir="./undrr" --resolve=true
```

You can repeat this harvest at intervals to keep your catalogue up to date with the remote.

## Summary

We've seen a number of options to import metadata from external sources. In the [next section](5-git-cicd.md) we'll have a look at Git, a versioning system.
