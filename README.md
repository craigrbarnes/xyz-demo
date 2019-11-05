# Visualize Data from a XYZ Space in harp.gl

We recently announced the beta release of harp.gl, our new 3D vector web rendering engine, and couldn't be happier for the positive feedback! Now that you've seen what harp.gl can do, let's start using it with other powerful tools we have introduced earlier this year. Our geospatial data storage and management solution, XYZ, integrates well with harp.gl. I'll show you how you can visualize a tiled data source from an XYZ Space inside harp.gl.

## Find data
The first thing we'll need to do is get a hold of some data. A great site for this is the Irish government's data portal. This data portal is an aggregated site for data set across many of the government's agencies. Let's take a look at the National Rail Segments data set, for example.

Download data from [https://data.gov.ie/dataset/rail-network-osi-national-250k-map-of-ireland](https://data.gov.ie/dataset/rail-network-osi-national-250k-map-of-ireland)


To download the data set, click on the Download button in the GeoJSON row. This will download an approximately 30mb sized file.

## Upload the data to an XYZ Space using the CLI.

If you don't already have a HERE Developer account and have used XYZ before, now would be a good time to sign up. It's free to get started with and doesn't require a credit card to sign up.

We'll be using the XYZ CLI to upload the data set to our XYZ Space. The CLI is great for uploading larger data sets.

If you don't already have the CLI installed, be sure to install it from npm and then log in. If you already have the CLI installed on your machine, please skip this step.

```
npm install -g @here/cli
```

```
here configure account
```

The first thing we'll want to do is create a new XYZ Space. We can do this with the following command:

```
here xyz upload -f Rail_Network__OSi_National_250k_Map_of_Ireland.geojson
```

You'll need to replace YOUR_SPACE_ID with the space ID that was outputted to you after you ran the here xyz create command in the previous step. Additionally, please replace PATH_TO_FILE with the path to the file downloaded from the data.gov site.

Your data has now been uploaded to an XYZ Space! To verify the data has been uploaded, you can run the following command
```
here xyz show YOUR_SPACE_ID
```
Set up the harp.gl map
There are two ways to consume the harp.gl api:

-  link a simple bundle as a <script> tag in your html
-  install a set of node modules from npm

In this blog post, we'll be using the simple bundle for simplicity's sake. For larger, more complex projects, we generally recommend using the node modules.

You'll want to create a new directory with two files in it, an index.html and an index.js.

```
mkdir rail-map
cd rail-map
touch index.js
touch index.html
```

You'll also want to set up a local server, for example in Python 2.x:
```
python -m SimpleHTTPServer 8888
```
and in Python 3.x:
```
python -m http.server 8888
```

Once you've created the new directory and the files within, open up index.html and add this code:

```html
<html>
   <head>
      <style>
         body, html { border: 0; margin: 0; padding: 0; }
         #map { height: 100vh; width: 100vw; }
      </style>
      <script src="https://unpkg.com/three/build/three.min.js"></script>
      <script src="https://unpkg.com/@here/harp.gl/dist/harp.js"></script>
   </head>
   <body>
      <canvas id="map"></canvas>
      <script src="index.js"></script>
   </body>
</html>
```

```javascript
const TOKEN = 'YOUR_XYZ_TOKEN';
const SPACE_ID = 'YOUR_SPACE_ID';
const map = new harp.MapView({
   canvas: document.getElementById('map'),
   theme: "https://unpkg.com/@here/harp-map-theme@latest/resources/berlin_tilezen_day_reduced.json",
});

window.onresize = () => map.resize(window.innerWidth, window.innerHeight);

map.setCameraGeolocationAndZoom(
   new harp.GeoCoordinates(-8.26250100000019,53.3331869990967),
   5
);

const controls = new harp.MapControls(map);
controls.maxPitchAngle = 90;
controls.setRotation(20, 50);

const omvDataSource = new harp.OmvDataSource({
   baseUrl: "https://xyz.api.here.com/tiles/herebase.02",
   apiFormat: harp.APIFormat.XYZOMV,
   styleSetName: "tilezen",
   authenticationCode: TOKEN,
});
map.addDataSource(omvDataSource);
```

The above code initializes our new harp.gl map with a default reduced day theme. We are also setting the map's default center and zoom, while also adding the HERE base map with the harp.OmvDataSource class.

Be sure to replace 'YOUR_XYZ_TOKEN' with your own XYZ token. To find your XYZ token, please run the following in your command line:

```
here xyz token
```

This command will output a few XYZ tokens to use. Copy and paste that into your code. We recommend using a read only token. This way, no one can overwrite your data set!

## Add the railways data set to the harp.gl map

Now that we've got the harp.gl base map configured, let's add the railways data set to the map.

We'll be using the OmvDataSource class again. We generally always use this class whenever we are adding vector tiles from a server.

```javascript
const xyzSpaceDataSource = new harp.OmvDataSource({
   baseUrl: `https://xyz.api.here.com/hub/spaces/${SPACE_ID}/tile/web`,
   apiFormat: harp.APIFormat.XYZSpace,
   authenticationCode: TOKEN,
});
```
We've initialized the data source, and now we'll add it to the map:
```javascript
map.addDataSource(xyzSpaceDataSource).then(() => {

   const styles = [{
         "when": `$geometryType ^= 'line'`,
         "renderOrder": 1000,
         "technique": "solid-line",
         "attr": {
            "color": "#FF3345",
            "metricUnit": "Pixel",
            "lineWidth": 3
         }
   }];
   xyzSpaceDataSource.setStyleSet(styles);
   map.update();
});
```
map.addDataSource() returns a promise, so we'll wait until the data has been added to the map. Once it's been added to the map, we will style the data with the harp.gl styling specification.

## Wrapping up
In just a few easy steps, we've created a great looking map of the different wild and scenic rivers across the United States. A natural next step of this map might be to create a map legend so viewers of the map can understand what lengths the different colors correlate to.

In this blog post you've learned how to:

- find and download public data sets from open data sites
- upload data to an XYZ Space with the command line interface
- initialize a basic harp.gl map
- add a tiled data set from an XYZ Space to a harp.gl map
