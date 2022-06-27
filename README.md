# Guide: OS Open Zoomstack to PMTiles
✨ I'm writing this guide - it's not ready to use yet!


This guide explains the process of converting OS Open Zoomstack data into PMTiles.

Ordnance Survey's free [Open Zoomstack](https://www.ordnancesurvey.co.uk/business-government/products/open-zoomstack) data is fantastic for creating lightweight basemaps. It's provided in two file formats:
- GeoPackage: good for traditional desktop GIS usage.
- MBTiles: for storing vector tile data for use on the web.

**MB**Tiles ('Mapbox Tiles') data can be disseminated by a server via two techniques:
- Via a server application, which serves induvidual tiles per request, but requires the developer to manage a backend environment.
- As an 'exploded' file structure, where the /{x}/{y}/{z}/ file path consists of physical paths, but often costs more in serverless compute environments.

**PM**Tiles ('Protomap Tiles') is a suitable alternative to MBTiles. It can be stored as a static file on a web server (that supports byte serving), and the .js client can access induvidual tiles within the archive. There's no need to explode the file structure, or manage a backend environment; simply upload the file and serve. The [PMTiles docs](https://protomaps.com/docs/pmtiles/storage-providers) list a number of compatible storage providers.

## Downloading OS Zoomstack Data
First, let's [download the Open Zoomstack data from the OS Data Hub](https://osdatahub.os.uk/downloads/open/OpenZoomstack) - it's a free download, so there's no need to login, but you must acknowledge the copyright and the source of the data by including the following attribution statement: `Contains OS data © Crown copyright and database right 2022`

Download the data in `Vector Tiles (MBTiles)` format.

## Installing the conversion library
