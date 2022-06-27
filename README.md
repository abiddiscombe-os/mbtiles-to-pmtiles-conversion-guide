# Guide: OS Open Zoomstack to PMTiles
✨ I'm writing this guide - it's not ready to use yet!


This guide explains the process of converting OS Open Zoomstack data into PMTiles.

Ordnance Survey Open Zoomstack data is a fantastic resource for creating a simple cartographic basemap, it's free to use and can be downloaded from the Ordnance Survey Data Hub.

The Open Zoomstack data can be downloaded in either GeoPackage or MBTiles file formats. GeoPackage is great for traditional desktop GIS usage. MBTiles ('Mapbox Tiles') is a great way to store vector tile data, hosted in one of two ways:
- Via a server application, which serves induvidual tiles per request.
- As an 'exploded' file structure, where the /{x}/{y}/{z}/ file path consists of a physical path on the disk.

These are great, but have issues:
- Using a server application requires managing a backend environment.
- Using induvidual files often costs more on serverless compute, and is a hassle to manage.

Another file format, PMTiles is a great alternative. It can be stored as a static file on a web server (that supports byte serving), and the .js client can access induvidual tiles within the archive.

## Downloading OS Zoomstack Data
