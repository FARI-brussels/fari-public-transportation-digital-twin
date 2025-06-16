# fari-public-transportation-digital-twin

The goal is to reimplement the Unity public transportation digital twin in a browser.


## Maps
### Bruxelles mobilité
WMS link : https://data.mobility.brussels/geoserver/ows
Layer : 
- bm_public_transport:stib_lines
- bm_public_transport:infrabel_lines
- bm_public_transport:delijn_lines

Visualise in cesium sandcastle 
NOT WORKING WHY ??
```javascript
const viewer = new Cesium.Viewer("cesiumContainer", {
  timeline: false,
  animation: false,
  baseLayerPicker: true,
});

// Brussels bounding box (approximate)
viewer.camera.setView({
  destination: Cesium.Rectangle.fromDegrees(4.2, 50.75, 4.45, 50.9)
});

// Base WMS URL
const wmsUrl = "https://data.mobility.brussels/geoserver/ows?";

// Function to add a WMS layer
function addWmsLayer(layerName, title, color) {
  viewer.imageryLayers.addImageryProvider(
    new Cesium.WebMapServiceImageryProvider({
      url: wmsUrl,
      layers: layerName,
      parameters: {
        transparent: true,
        format: "image/png"
      },
      credit: "data.mobility.brussels"
    })
  );
}

// Add layers
addWmsLayer("bm_public_transport:stib_lines", "STIB Lines");
addWmsLayer("bm_public_transport:infrabel_lines", "Infrabel Lines");
addWmsLayer("bm_public_transport:delijn_lines", "De Lijn Lines");
```

### Urbis
WMS link :https://geoservices-urbis.irisnet.be/geoserver/ows?version=1.3.0
Layer : 
- BaseMaps:UrbISNotLabeledColor


Visualise in cesium sandcastle
```javascript
const viewer = new Cesium.Viewer("cesiumContainer", {
  baseLayerPicker: false,
  timeline: false,
  animation: false
});

const wmsLayer = viewer.imageryLayers.addImageryProvider(
  new Cesium.WebMapServiceImageryProvider({
    url: 'https://geoservices-urbis.irisnet.be/geoserver/ows',
    layers: 'BaseMaps:UrbISNotLabeledColor',
    parameters: {
      service: 'WMS',
      transparent: true,
      format: 'image/png'
    },
    credit: 'Land Cover - URBIS'
  })
);

// Optionally zoom to Brussels
viewer.camera.setView({
  destination: Cesium.Rectangle.fromDegrees(4.25, 50.75, 4.45, 50.95) // Approximate bounds of Brussels
});
```



## 3D assets
- bus  : 3465321
- tram : 3465325
- train : 3465324
- bus stop : 3465322



Visualise in cesium sandcastle
```javascript
// Set your ion token
Cesium.Ion.defaultAccessToken = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJhY2E3ZDhlNC03Yjc0LTQzM2QtYmI5My0zYWQ3NjIwOTk0OTciLCJpZCI6Mjc4NzM4LCJpYXQiOjE3NDA0ODg1MjB9.VsZjL6pbKSwR_SBbxUq-KRweOU_P3R8DKjSpeD0EICY';

const viewer = new Cesium.Viewer("cesiumContainer", {
  timeline: false,
  animation: false
});

// Asset info
const assets = [
  { id: 3465321, name: "Bus" },
  { id: 3465325, name: "Tram" },
  { id: 3465324, name: "Train" },
  { id: 3465322, name: "Bus Stop" },
];

// Load all assets
assets.forEach(asset => {
  Cesium.IonResource.fromAssetId(asset.id)
    .then(resource => {
      const entity = viewer.entities.add({
        name: asset.name,
        position: Cesium.Cartesian3.fromDegrees(4.35, 50.85, 0), // Approximate position over Brussels
        model: {
          uri: resource,
          scale: 1.0,
          minimumPixelSize: 64
        }
      });
    })
    .catch(error => {
      console.error(`Failed to load ${asset.name}:`, error);
    });
});

// Fly to the location
viewer.camera.flyTo({
  destination: Cesium.Cartesian3.fromDegrees(4.35, 50.85, 1000),
});
```


https://github.com/FARI-brussels/Fari-Brussels-Digital-Twin-Unity/blob/main/ServerExt/update_geoJsonDeLijn.py This is used to convert the data from delijn to the same format as stib and sncb data.
The api should run in the background for now. We will implement that in the global backend after.

https://github.com/FARI-brussels/Fari-Brussels-Digital-Twin-Unity/blob/main/DigitalTwin/Assets/Script/API/ApiClient.cs This is getting all position from the different api 

https://github.com/FARI-brussels/Fari-Brussels-Digital-Twin-Unity/blob/main/DigitalTwin/Assets/Script/Other%20Manager/Stib/Visualization/VehicleMover.cs this is effectively moving the vehicules
