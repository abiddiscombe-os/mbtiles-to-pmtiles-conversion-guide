# MBTiles to PMTiles: Converting OS Open Zoomstack.

This guide explains how to create a **PM**Tiles version of the OS Open Zoomstack **MB**Tiles data.

## Introduction

**PM**Tiles ('Protomap Tiles' - [Official Website](https://protomaps.com/)) is a vector tile file format which supports random access, meaning induvidual tiles can be accessed as part of a single **PM**Tiles file when hosted on a server which supports [byte serving](https://en.wikipedia.org/wiki/Byte_serving). A JavaScript 'plugin' for mapping libraries (such as MapLibre) has been created by the Protomaps team; this provides the client functionality to access discreet parts of the **PM**Tiles file: requesting induvidual tiles per each request.

Another *de-facto* standard for storing tile data is the **MB**Tiles file format, which has an SQLite database at its core. Whilst not a replacement for **MB**Tiles, the **PM**Tiles format has several notable adventages in specific use-cases where site traffic is low and there is no requirement to authenticate clients:
- It is suitable for deployment to static sites or CDNs, which are often lower-cost and low-latency.
- There is no need for middleware/backend to read induvidual tiles via SQL and serve them to clients.
- There is no need to 'explode' the tiles into a series of physical directories following the `{z}/{x}/{y}` format.

The [Protomaps documentation](https://protomaps.com/docs/pmtiles/storage-providers) lists compatible storage providers for hosting PMTiles.

## Downloading OS Open Zoomstack MBTiles
Ordnance Survey's free [Open Zoomstack](https://www.ordnancesurvey.co.uk/business-government/products/open-zoomstack) is a great data source for creating simple and customisable web basemaps. Download the OS Open Zoomstack data from the [OS Data Hub](https://osdatahub.os.uk/downloads/open/OpenZoomstack) in `Vector Tiles (MBTiles)` format.

This is free and open data, [but you must provide attribution](https://github.com/OrdnanceSurvey/os-api-branding) to use the data in your mapping projects.

## Conversion Guide
Make sure you have an up-to-date copy of `python3` and `python3-pip` on your system, then install the `pmtiles` python package, which we will use to convert **MB**Tiles into **PM**Tiles. See the [official documentation](https://protomaps.com/docs/pmtiles#pmtiles-for-python) for further advice and support.

```bash
pip install pmtiles
```

Once installed, run the following command to convert the OS Open Zoomstack data into **PM**Tiles format.  

```bash
pmtiles-convert OS_Open_Zoomstack.mbtiles OS_Open_Zoomstack.pmtiles
```

*Running on WSL Ubuntu 20.04, my .pmtiles output was around 3.5 GB in size and took 30 seconds to produce.*

## Self-Hosting the OS Open Zoomstack PMTiles
This guide assumes you already have access to a CORS-enabled storage location which supports [byte serving](https://en.wikipedia.org/wiki/Byte_serving).

Download the `OS_Open_Zoomstack` directory included within this repository. This folder contains style data, sprites, and web fonts required to load OS Zoomstack. The `Road`, `Outdoor`, `Light`, and `Night` styles are all provided.

In each of the four style documents (e.g. `light.json`), modify the `http://localhost/` component of the URLs (under `tiles`, `sprite`, `glyphs`) to match the destination where you will host these files.

```json
"version": 8,
"name": "OS Open Zoomstack - Light",
"sources": {
    "composite": {
        "type": "vector",
        "tiles": [
            // edit me (1/3)
            "pmtiles://http://localhost/OS_Open_Zoomstack/OS_Open_Zoomstack.pmtiles/{z}/{x}/{y}"
        ]
    }
},
"sprite": "http://localhost/OS_Open_Zoomstack/sprites/sprites", // edit me too (2/3)
"glyphs": "http://localhost/OS_Open_Zoomstack/fonts/{fontstack}/{range}.pbf", // and me (3/3)
```
You can now upload these files to your hosting location.

## Testing access to Open Zoomstack PMTiles within MapLibre GL JS
*Sample code to help you get started is provided in the `demo.html` file in this repository.*  
Add the sources for MapLibre, and the PMTiles for MapLibre plugin to your HTML:

```html
<! MapLibre >
<link href='https://unpkg.com/maplibre-gl@2.1.9/dist/maplibre-gl.css' rel='stylesheet' />
<script src='https://unpkg.com/maplibre-gl@2.1.9/dist/maplibre-gl.js'></script>
<! PMTiles for MapLibre >
<script src="https://unpkg.com/pmtiles@1.0.4/dist/index.js"></script>
```

In your JavaScript, define a new Protocol Cache for PMTiles and connect it to MapLibre.  
Define a new MapLibre instance. The value for `style` should match the location of one of the four style documents you uploaded to your hosting location.

```javascript
let cache = new pmtiles.ProtocolCache();
maplibregl.addProtocol("pmtiles",cache.protocol);

const map = new maplibregl.Map({
    container: 'map',
    style: 'http://localhost/OS_Open_Zoomstack/light.json',
    // customise map placement to meet your needs
    center: [-2.9626, 54.4301],
    zoom: 12,
    bearing: 0,
    pitch: 40.00,
    padding: [0, 0, 0, 0],
});
```
You should now be able to open your HTML file and view the PMTiles data in MapLibre.

## Common Pitfalls
- Is the server CORS-enabled? Read more [here](https://protomaps.com/docs/pmtiles/storage-providers).
- Are the URLs for the style documents and the .pmtiles file correct?
- Are you using HTTPS for connections where required?
- OS_Open_Zoomstack only covers Great Britian - is your map centered in the right location?
