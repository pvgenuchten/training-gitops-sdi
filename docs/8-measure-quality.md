---
title: Quality of Service
summary: Page content is derived from https://github.com/ejpsoil/soildata-assimilation-guidance
author: Paul van Genuchten
date: 2022-11-10
---
 
Quality of service monitoring supports data providers to understand strengths and weaknesses of a system. Aspects which are monitored are:

- [Availability](#availability-monitoring) (% of the time that the service has been available)
- [Performance and capacity](#performance--capacity) 
- [Usage](#usage-monitoring) (how (much) are services used)
- [Search engine optimisation](#search-engine-optimisation) (SEO)

Quality of service monitoring is a standard activity in IT. Therefore consult your IT department or hosting company if they have tools available to assess these aspects. Confirm with them on how to extend and/or share with you the registrations for the requested parameters. Combine these reports into monthly or quarterly reports to facilitate system maintenance and roadmap. 


---

## Availability Monitoring

To assess the availability of a service, it requires to monitor the availability of the service at intervals. A basic availability-test every 5 minutes is usually sufficient. Many software exists for availability monitoring, such as [Zabbix](https://zabbix.com/), [Nagios](https://nagios.org/), [CheckMK](https://checkmk.com/), [pingdom](https://www.pingdom.com/). 

In this workshop we're introducting the a Python based monitoring solution, called [GeoHealthCheck](https://geohealthcheck.org/). Geohealthcheck is designed to monitor spatial data services specifically.

The following exercise assumes docker desktop to be installed. Alternatively you can create a personal account at https://demo.geohealthcheck.org (click register in the login page). 


:::{.callout-tip}
- Start by setting up a local GeoHealthCheck container:

```bash
docker run -p80:80 geopython/geohealthcheck
```
- Visit http://localhost
- Login as user: admin, password: admin
- Click `ADD +` on the top bar right, select `WMS`
- Add a WMS url, for example https://maps.isric.org/mapserv?map=/map/wrb.map
- On the next screen add `WMS Drilldown` (so all layers are validated)
- Click `Save and test`
- When finished, click Details to see the test result
:::

This test is automatically repeated at intervals (as long the service is running). You can return to the test page to view a diagram of the availability over time. You can also configure a e-mail address to recieve notifications if a service is not available.

Read more at the [GeoHealthCheck website](https://docs.geohealthcheck.org/en/latest/),

An advanced aspect of availability is identifying broken links on your content. Many tools exist which can monitor wensites on broken links at intervals, for example [W3C linkcheck](https://validator.w3.org/checklink). Also [google search console](https://search.google.com/search-console/about) can be used to identify broken links. Another option is to verify in your website access logs if there are any requests which returned a 404 status, if such a request has a [referer url](https://en.wikipedia.org/wiki/HTTP_referer), you are able to identify which website incorrectly linked to one of your resources.

:::{.callout-tip}
Visit <https://validator.w3.org/checklink> and fill in a website you maintain or frequently visit. The `check` button starts a process to evaluate broken links.
:::

Sometimes a machine is not able to identify if a link is broken, for example if the target does not return a typical `404 Not found` message.

---

## Performance & Capacity

To know the capacity and performance of a service you can perform some load tests prior to moving to production. An alternative approach to evaluate performance is to extract the access logs of the service into an aggregation tool like [AWStats](https://awstats.sourceforge.io) or [Kibana](https://www.elastic.co/kibana) and evaluate the number of requests exceeding the limits.

:::{.callout-note}
A common challenge to service performance in spatial data is the provision of a WMS service on a big dataset. When requesting that dataset on a continental or national level, the server runs into problems drawing all the data at once. In such case consider to set up some cache/aggregation mechanism for larger areas. Setting proper min/max scale denominators can be a solution also.
:::

[jmeter](https://jmeter.apache.org/) is a utility which can run a series of performance and capacity tests on a webservice.
Jmeter is a java program, which can run on most platforms. 

- Download the latest version from the [apache website](https://jmeter.apache.org/download_jmeter.cgi). 
- Unzip the archive and run `jmeter.bat` from bin directory.
- Follow the [build web test plan](https://jmeter.apache.org/usermanual/build-web-test-plan.html) tutorial. 
- Customise the web test plan for your mapserver service

:::{.callout-note}
Do not perform a load test against a production url, it wil severely impact the performance of that service. 
:::

---

## Usage monitoring

To capture the usage of a service you can extract the usage logs and import them in a tool like [Kibana](https://www.elastic.co/kibana), [Splunk](https://www.splunk.com/), [Matomo](https://matomo.org/) or [AW stats](https://awstats.sourceforge.io/). For spatial data, it is interesting to define rules to extract the requested layer name from a WMS request. 

[AWStats](https://awstats.sourceforge.io/) is a basic utility to report on service usage. 

- Navigate to an empty folder, place a [sample log file](https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs) in the folder, rename the file to `access.log`.
- Start a container

::: {.panel-tabset}
# Linux
```bash
docker run -d --restart always --publish 3000:80 \
 --name awstats --volume $(pwd):/var/local/log:ro \
 pabra/awstats
```
# Powershell
```bash
docker run -d --restart always --publish 3000:80 `
 --name awstats -v "${PWD}:/var/local/log:ro" `
 pabra/awstats
```
:::

- Parse the logs:

```
docker exec awstats awstats_updateall.pl now
```

- View the dashboard at http://localhost:3000, navigate to `May 2015` to see the parsed logs from the sample.

---

### Search Engine Optimisation

As part of usage monitoring it is also of interest to understand how users find your data via search engines. The popular search engines offer [tooling](https://search.google.com/search-console) to report on how the crawlers navigate through your data and how users find your services. You need to [verify ownership](https://support.google.com/webmasters/answer/9008080?hl=en) of a domain either via a DNS property or by uploading an identification string to the website. 

You can also use [Google rich result test](https://search.google.com/test/rich-results) to extract structured data from a website. Or go to the [dataset search engine](https://datasetsearch.research.google.com/) to understand if and how the search engine finds your data.

---

## Summary

In this section you learned about various mechanisms to evaluate, report about and improve service quality. This concludes the main part of the workshop. In the [last section](./9-data-visualisation.md) we present a framework for data visualisation.