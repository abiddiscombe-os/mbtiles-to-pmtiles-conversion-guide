# Guide: OS Open Zoomstack to PMTiles
ℹ This guide explains the process of converting OS Open Zoomstack data into PMTiles.

---

Ordnance Survey's free [Open Zoomstack](https://www.ordnancesurvey.co.uk/business-government/products/open-zoomstack) data is a great tool for creating self-hosted basemaps for use in web maps. It's provided as both `GeoPackage`, and `MBTiles` - the latter is essentially an SQLite database holding vector tiles suitable for use with web mapping libraries

**MB**Tiles ('Mapbox Tiles') data can be served to clients via two techniques:
- A server capable of reading the SQLite MBTiles database fetches induvidual tiles per each request. **Requires a backend environment**.
- As an expanded file structure, where the `/{x}/{y}/{z}/` file path allows for direct access to each tile.

**PM**Tiles ('Protomap Tiles') is an alternative to MBTiles. It can be stored as a static file on a web server (that supports [byte serving](https://en.wikipedia.org/wiki/Byte_serving)), and the Protomaps .js client can access induvidual tiles within the archive, allowing mapping engines to read the data. The PMTiles format is suitable for deployment to static sites or CDNs, which are often cheaper and have a lower latency. The [docs](https://protomaps.com/docs/pmtiles/storage-providers) list some compatible storage providers for hosting PMTiles.

## 1. Downloading OS Zoomstack Data
Download the Open Zoomstack data from the [OS Data Hub](https://osdatahub.os.uk/downloads/open/OpenZoomstack) in `Vector Tiles (MBTiles)` format.

Please acknowledge the copyright and the source of the data by including the following attribution: `Contains OS data © Crown copyright and database right 2022`

## 2. Installing the PMTiles Conversion Library
*You will need Python to run this tool.*
