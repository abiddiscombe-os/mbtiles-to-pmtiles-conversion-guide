@abiddiscombe | [abiddiscombe.dev](https://abiddiscombe.dev)

# PMTiles Conversion Guide for Ordnance Survey Open Data Products

[Protomap Tiles](https://protomaps.com/docs/pmtiles) (**PM**Tiles) is a single-file vector or raster tile storage specification which stores a complete `z/x/y` tileset within a single archive, using a compressed form of Hilbert Ordering. Tiles can be accessed individually from within the PMTiles archive using [HTTP range requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests) which can return a specified portion of a static resource.

Another *de-facto* standard for storing tile data is the **MB**Tiles file format which was originally developed at Mapbox. MBTiles are loosely modelled upon an SQLite database thus tiles are queried via SQL. Whilst not intended to be a replacement for **MB**Tiles, the **PM**Tiles format has several notable advantages in specific use-cases where site traffic is low and there is no requirement to authenticate clients on the server.

- It is suitable for deployment to either static sites or S3-compatible data stores, which are low-cost and low-latency. A PMTiles file can sit on the network edge, and serve data to clients with minimal overhead nor compute infrastructure.

- Unlike a 'traditional' tile storage mechanism such as MBTiles, there is no need for a service backend or compute instance to query tiles from a database and serve them onwards to clients.

- There is no need to 'explode' the tiles into a series of physical directories following the `{z}/{x}/{y}` format on a server (a common alternative to the use of MBTiles). The transfer of many-thousand tiles to a server as discreet files takes a considerable amount of time, and is challenging to maintain across updates to the entire, or a portion of, the tileset.

The [Protomaps site provides a convenient method](https://app.protomaps.com/downloads/small_map) to select a cut of OpenStreetMap data to convert and download in the PMTiles data on-the-fly. The guide explains how to replicate this capability with third-party datasets, including Open Data from Ordnance Survey. Please be advised that more detailed instructions on these steps have been prepared in the [PMTiles documentation](https://protomaps.com/docs/pmtiles). The PMTiles format is also suitable for premium data when used in internal systems.

## Deployment
A PMTiles file can be deployed to a static site, blob store, or CDN. The [Protomaps documentation](https://protomaps.com/docs/pmtiles/cloud-storage) lists some compatible storage providers. The authors of this document have successfully deployed PMTiles to Azure Blob Containers and DigitalOcean Spaces. To access data, the service must have an appropriate CORS configuration to enable tiles to be fetched from a web map on a specific domain, and support for [byte serving](https://en.wikipedia.org/wiki/Byte_serving).

## Client Access to PMTiles Data
JavaScript plugins for web-based mapping libraries, such as [MapLibre](https://maplibre.org/maplibre-gl-js-docs/api/) and [Leaflet](https://leafletjs.com/) , have been created by the Protomaps team: [these plugins](https://protomaps.com/docs/frontends/basemap-layers) enable clients to resolve and ingest tiles as if they were served from a standard `z/x/y` API endpoint. *This places an additional processing overhead on the client, if this causes issues in your ecosystem, Protomaps does suggest an [alternative method which moves some of the compute onto cloud-based edge functions](https://protomaps.com/docs/cdn).*

To use these plugins you must first install the PMTiles JavaScript library, and connect this to your mapping library. The following examples are for use with [MapLibre](https://maplibre.org/maplibre-gl-js-docs/api/).

### Installation
If you are developing your application in NodeJS:
``` bash
npm install pmtiles
```
Otherwise, a script tag is available via unpkg.org which is suitable for simpler deployments:
``` html
<!-- more current versions may be available -->
<script src='https://unpkg.com/pmtiles@2.7.0/dist/index.js'></script>
```

### Usage
In your JavaScript code, you need to define the PMTiles protocol, and then add it to your web mapping library. For MapLibre:
``` javascript
const pmtilesProtocol = new pmtiles.Protocol()
maplibregl.addProtocol('pmtiles', pmtilesProtocol.tile);
```
Once added, the rest of the code required to create a basic functioning web map is very simple, MapLibre uses a `style.json` file to define the map's sources and layers, the following example explains how to integrate the PMTiles protocol into a basic map template. It is worth noting that MapLibre will also accept a URL to an external `.json` file as a style reference.
``` javascript
const styleDocument = {
	"version": 8,
	"name": "Generic MapLibre Styling Document",
	"sources": {
		"boundaryline": {
		"type": "vector",
		"tiles": [ "pmtiles://https://example.com/bdline_gb.pmtiles/{z}/{x}/{y}" ]
	},
	layers: [
		{
			"id": "westminster_const",
			"type": "line",
			"source": "boundaryline",
			"source-layer": "westminster_const",
			"layout": {},
			"paint": {
				"line-color": "#ef2199"
			}
		}
	]
}

const map = new maplibregl.Map({
	container: 'map',
	style: styleDocument,
	center: [-2.9626, 54.4301],
	zoom: 7,
	maxZoom: 14,
	minZoom: 5,
});
```
Please note, your style document must contain all relevant styling for each layer within the PMTiles file, just as with any other vector tile data source. Vectors without styles will not be visible on the map. **To help you get started, several PMTiles example stylesheets for OS Open Data have been included in this repository.**

## Conversion Guide (for OS Open Data)
The following section will walk you through the steps required to create your own PMTiles data from an MBTiles archive. These instructions have been written for MS Windows users, but should be replicable on MacOS and Linux Distributions. These steps are also replicable for non-OS data.

1. Please download the relevant data products from the [OS Data Hub](https://osdatahub.os.uk/downloads/open/) in the `.mbtiles` format.

> ℹ **Attribution Required**  
> Ordnance Survey's Open Data is free to use in personal and commercial projects, [but you must provide attribution](https://github.com/OrdnanceSurvey/os-api-branding) where you use it.

2. Download the latest PMTiles CLI utility for your system. You can find and download the latest binaries from [Protomaps' GitHub](https://github.com/protomaps/go-pmtiles/releases) page.

3. Once fully downloaded, extract the `.zip` archive to a location on your system where you can easily access the MBTiles file, and open PowerShell to the same directory.

4. The following command will use the PMTiles utility to convert a named `.mbtiles` file into a new `.pmtiles` file, the time required will vary depending on the size of the source data, for OS Open Zoomstack, it took me about 40 seconds:

``` bash
.\pmtiles.exe convert OS_Open_Zoomstack.mbtiles OS_Open_Zoomstack.pmtiles
```

5. You now have a ready-to-serve PMTiles archive prepared on your system. Whenever the source data is uploaded, it is a case of re-generating the PMTiles file and pushing it to your cloud hosting provider.

## Common Pitfalls

- Is the server CORS-enabled? Read more [here](https://protomaps.com/docs/pmtiles/storage-providers).

- Are the URLs for the style documents and the `.pmtiles` file correct?

- Are you using HTTPS for connections where required? Some hosting providers and domains requires HTTPS by default or have it preregistered in the HSTS preload list.

- OS' Open Data products only cover Great Britain (so your map may appear empty beyond this extent) - please check your map is centred in the right location.