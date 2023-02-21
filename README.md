@abiddiscombe | [abiddiscombe.dev](https://abiddiscombe.dev)

# PMTiles Conversion Guide for Ordnance Survey Open Data Products

[Protomap Tiles](https://protomaps.com/docs/pmtiles) (**PM**Tiles) is a single-file vector or raster tile storage specification which stores a complete tileset within a single file using a compressed form of Hilbert Ordering. Tiles can be accessed from within a PMTiles archive using [HTTP range requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests) which return a specified portion of the resource.

[![](/media/demo-button.png)](https://abiddiscombe-os.github.io/mbtiles-to-pmtiles-conversion-guide)

Another *de-facto* standard for storing tile data is the **MB**Tiles file format. MBTiles files are loosely modelled on an SQLite database; tiles are returned via SQL queries. Whilst not intended to replace **MB**Tiles, the **PM**Tiles format has several notable advantages in specific use-cases where site traffic is low:

- It is suitable for deployment to either static file buckets, CDNs, or blob stores (including those compatible with the S3 transfer protocol), which are low-cost and low-latency. A PMTiles file can sit on the network edge, and serve data to clients with minimal overhead and no cloud compute requirements. MBTiles cannot be stored in this form because they must be queried via SQL.

- Unlike MBTiles, there is no need for a service backend to query tiles from a database and serve them onwards to clients, as this process can be handled on the client. Alternatively, edge functions or workers can be utilised to fetch data from a blob store and serve it onwards in the universal `z/x/y` format from a geographically perferable position on the network edge. The latter approach is similar to existing architectual mechanisms for serving MBTiles and provides finer controls over security.

- There is no requirement to 'explode' the tiles into a series of physical directories following the `{z}/{x}/{y}` format on a server. The transfer of many-thousand tiles to a server as discreet files takes time and is challenging to maintain.

The [Protomaps site provides a convenient method](https://app.protomaps.com/downloads/small_map) to select a cut of OpenStreetMap data to convert and download in the PMTiles format on-the-fly. This guide explains how to replicate this capability with third-party datasets including data from Ordnance Survey. Please be advised that more detailed instructions on these processing steps have been prepared in the [PMTiles documentation](https://protomaps.com/docs/pmtiles).

In this guide, OS' Open Data products (OS Open Zoomstack, OS BoundaryLine, etc.) are converted and hosted publicly, however the PMTiles format is also suitable for premium datasets where access is restricted. **When using Ordnance Survey's datasets [you must provide attribution](https://github.com/OrdnanceSurvey/os-api-branding) where you use it, including on digital maps or demonstrations.**

## MBTiles to PMTiles Conversion Process
The following section will walk you through the steps required to create your own PMTiles data from an MBTiles archive. These instructions have been written for Windows, but are replicable on MacOS and Linux.

1. Please download the relevant data products from the [OS Data Hub](https://osdatahub.os.uk/downloads/open/) in the `.mbtiles` format.

2. Download the latest PMTiles CLI utility for your system. You can download the latest binaries from [Protomaps' GitHub](https://github.com/protomaps/go-pmtiles/releases) page.

3. Once fully downloaded, extract the `.zip` archive to a location on your system where you can easily access the MBTiles file, and then open a new PowerShell prompt within the directory.

4. The following command will use the PMTiles utility to convert an existing `.mbtiles` file into a new `.pmtiles` file, the time required will vary depending on the size of the source data, for OS Open Zoomstack, it takes about 40 seconds:

``` bash
.\pmtiles.exe convert OS_Open_Zoomstack.mbtiles OS_Open_Zoomstack.pmtiles
```

5. You now have a ready-to-serve PMTiles archive which can be uploaded to a supported blob store or static site. When the source data is updated you can re-generate the PMTiles file and re-upload it.

Whilst you now have a base vector tileset ready for use in a mapping library, you will still need to provide the relevant map style documents for each layer, plus any fontstack or sprite resources used by any particular layer. This is no different to supplying resources when using a standard `z/x/y` tile server.

The resources for OS Open Zoomstack have been provided in this repository. Please upload these resources to your blob store and update the relevant URLs to match your configuration.

## Deployment
There are two different methods of serving tiles to clients. In all deployment scenarios the hosting service must have an appropriate CORS configuration to enable tiles to be fetched from a specific domain.

### Option 1: Edge-Based CDN Tile Relay
The first method introduces an additional proccessing step on the cloud. It involves configuring a serverless function to query the PMTiles file and forward tiles (if found) via HTTP to clients. This approach provides greater control over the HTTP connection: authentication (such as API keys or bearer tokens) can be applied, and detailed CORS configurations can be administered.

Detailed instructions on the hosting of PMTiles data in this way is available on the [Protomaps Documentation](https://protomaps.com/docs/cdn). Protomaps encourages the use of Cloudflare's R2 and Workers, or the use of AWS' Cloudfront.

### Option 2: PMTiles Client Conversion
The second option involves a PMTiles file deployed to a blob store or Content Delivery Network (CDN). The [Protomaps documentation](https://protomaps.com/docs/pmtiles/cloud-storage) lists some compatible storage providers.

Plugins for JavaScript mapping libraries such as [MapLibre](https://maplibre.org/maplibre-gl-js-docs/api/) and [Leaflet](https://leafletjs.com/), have been created by the Protomaps team: [these plugins](https://protomaps.com/docs/frontends/basemap-layers) enable clients to resolve and ingest tiles as if they were served from a standard `z/x/y` API endpoint. This requires an additional processing step on the client.

**The following examples are for use with [MapLibre](https://maplibre.org/maplibre-gl-js-docs/api/).**

If you are developing your application in NodeJS, you can install the `pmtiles` plugin directly; otherwise the CDN version hosted on `unpkg.org` can be imported:

``` html
<!-- more current versions may be available -->
<script src='https://unpkg.com/pmtiles@2.7.0/dist/index.js'></script>
```

Once installed, you must define the PMTiles protocol and add it to MapLibre or Leaflet. The plugin enables PMTiles sources to be added to MapLibre's style document:

``` javascript
const pmtilesProtocol = new pmtiles.Protocol()
maplibregl.addProtocol('pmtiles', pmtilesProtocol.tile);

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

Please note, your style document must contain all relevant styling for each layer within the PMTiles file (just as with any other vector tile data source). Vectors without style information will not be visible. To help you get started several example stylesheets for OS Open Data have been included within this repository.

## Common Pitfalls
If you have following the above instructions and still encounter issues, please run through this checklist:

- Is the server CORS-enabled? Read more [here](https://protomaps.com/docs/pmtiles/storage-providers).

- Are the URLs for the style documents and the `.pmtiles` file correct?

- Are you using HTTPS for connections where required? Some hosting providers and domains requires HTTPS by default.

- OS' Open Data products only cover Great Britain (so your map may appear empty beyond this extent) - please check your map is centred in the right location.

- If you are using a cloud function, is the connection between the pmtiles file and the compute infastructure valid? You may wish to check your access keys.