# Converting OS Open Zoomstack into the PMTiles vector tile format.

**IMPORTANT:  
This technique and the instructions written here are still experimental, please don't use it for production just yet!**

This guide explains the steps to deploy a PMTiles version of the OS Open Zoomstack data for use in a MapLibre GL JS map.  
Ordnance Survey's free [Open Zoomstack](https://www.ordnancesurvey.co.uk/business-government/products/open-zoomstack) is a useful source of data for creating simple and highly customisable web basemaps. It is provided in both `GeoPackage`, and `MBTiles` formats.

## Why use PMTiles?

**MB**Tiles ('Mapbox Tiles') are a data format for storing vector tiles in a single archive. They can be served to clients via a middleware application which reads induvidual tiles from the database per each request, or by expanding file structure on the server, whereby the `/{x}/{y}/{z}/` file path corresponds to a physical path.

Using middleware to read the database requires a backend to your web app, a hassle and cost to maintain (you could choose to use the OS Maps or Vector Tile APIs instead). Similarly, expanding the file structure is a simple way of getting tiles online, but is more complex to update and maintain. 

**PM**Tiles ('Protomap Tiles') is an alternative to MBTiles which can be stored as one static file on a server (one which supports [byte serving](https://en.wikipedia.org/wiki/Byte_serving)), using a JavaScript plugin for MapLibre GL JS, the client can access parts of this single file per request, extracting specific tiles each time.

Whilst not a replacement for MBTiles, the PMTiles format is suitable for deployment to static sites or CDNs, which are often cheaper and easier to run, and have lower latencies. The [Protomaps documentation](https://protomaps.com/docs/pmtiles/storage-providers) list some compatible storage providers for hosting PMTiles.

## Instructions

### Downloading the OS Zoomstack Data
Please start by downloading the OS Open Zoomstack data from the [OS Data Hub](https://osdatahub.os.uk/downloads/open/OpenZoomstack) in Vector Tiles (MBTiles) format. At the time of writing, this file is around 2.5 GB in size.

### Installing the PMTiles Conversion Library
[Protomaps](https://github.com/protomaps), the team behind PMTiles have released the `pmtiles` python package which converts MBTiles into PMTiles. You can read the documentation [here](https://github.com/protomaps/PMTiles).

Make sure you have an up-to-date copy of `python3` and `python3-pip` on your system, then install the `pmtiles` python package:

```
pip install pmtiles
```

### Converting from MBTiles into PMTiles
Once installed, run the following command to convert the OS Open Zoomstack data, it might take some time to run.
I've tested this on a WSL installation of `Ubuntu 20.04`, my PMTiles output was around 3.5 GB in size.

```
pmtiles-convert TILES.mbtiles TILES.pmtiles
```

You can check the output file is functional using [Protomap's PMTiles Viewer](https://protomaps.github.io/PMTiles/).  
When ready, upload the PMTiles file to a static location or CDN which supports [byte serving](https://en.wikipedia.org/wiki/Byte_serving).

### Integrating PMTiles into MapLibre GL JS
*Quick tip - take a look at `demo.html` in this repo for an example of how the code below should look.*

We can now setup our MapLibre GJ LS map, and add the JavaScript module to read PMTiles:

```
<script src="https://unpkg.com/pmtiles@1.0.4/dist/index.js"></script>
```

In your JavaScript, define a new `protocolCache` for PMTiles and add it as a protocol to MapLibre. Then, define the map instance.

The value for `style` should be the URL where you'll put the stylesheets and supporting files. I'd reccomend you keep them with the PMTiles file. You'll need to specify the type of style you wish to use, in this example that is `_Road`.

```
let cache = new pmtiles.ProtocolCache();
maplibregl.addProtocol("pmtiles",cache.protocol);

const map = new maplibregl.Map({
    container: 'map',
    style: 'https://abiddiscombe-os/zoomstack2pmtiles/assets/OS_Open_Zoomstack_Road/style.json',
    center: [-2.9626, 54.4301],
    zoom: 12,
    bearing: 0,
    pitch: 40.00,
    padding: [0, 0, 0, 0],
});
```

### Updating the Style JSON
Inside the `assets` directory of this repo are a series of folders for each OS Open Zoomstack theme (Light, Night, Road, and Outdoor). They have all been modified to work with PMTiles. Please download the folders relating to the map themes you wish to use.

Before you upload these to a server, we need to update the contents of the `style.json` file in each theme folder. The values for `tiles`, `sprite`, and `glyphs` all need to correspond to the URL where you will host the respective files.

```
"version": 8,
"name": "OS Open Zoomstack - Road",
"sources": {
    "composite": {
        "type": "vector",
        "tiles": [
            "pmtiles://https://abiddiscombe-os.github.io/zoomstack2pmtiles/OS_Open_Zoomstack.pmtiles/{z}/{x}/{y}"
        ]
    }
},
"sprite": "https://abiddiscombe-os.github.io/zoomstack2pmtiles/assets/OS_Open_Zoomstack_Road/sprites/sprites",
"glyphs": "https://abiddiscombe-os.github.io/zoomstack2pmtiles/assets/OS_Open_Zoomstack_Road/fonts/{fontstack}/{range}.pbf",
...
```

For example, if we were hosting at the root of `cdn.example.com`, the code above would now look like this:

```
"version": 8,
"name": "OS Open Zoomstack - Road",
"sources": {
    "composite": {
        "type": "vector",
        "tiles": [
            "pmtiles://https://cdn.example.com/OS_Open_Zoomstack.pmtiles/{z}/{x}/{y}"
        ]
    }
},
"sprite": "https://cdn.example.com/OS_Open_Zoomstack_Road/sprites/sprites",
"glyphs": "https://cdn.example.com/OS_Open_Zoomstack_Road/fonts/{fontstack}/{range}.pbf",
...
```

We're done! You can now upload these files to the server, and check it all works in a sample map.
