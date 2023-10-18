# Advanced-Vegetation-Index
 // Import an uploaded feature collection
var west = ee.FeatureCollection("users/dynamicriki1999/west");
Map.addLayer(west, {color: 'red'}, "West Tripura");
Map.centerObject(west, 10);


// Import sentinal 2 image collection

var s2 = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED");
print(s2.size());

// filter image collection by location, date, metadata
var filteredS2 = s2.filterBounds(west)
                   .filterDate("2020-01-01","2020-12-31")
                   .filterMetadata("CLOUDY_PIXEL_PERCENTAGE", "less_than", 10);
print(filteredS2.size());
print(filteredS2);

// sort cloud free image by asseding

var cloudFreeImage = filteredS2.sort("CLOUDY_PIXEL_PERCENTAGE")
                               .first();
print(cloudFreeImage);

var thirdImage = ee.Image("COPERNICUS/S2_SR_HARMONIZED/20200117T043121_20200117T043313_T46QCM");


Map.addLayer(thirdImage, {}, "Cloud Free Sentinal 2 Image");


 // Create median composite
var composite = filteredS2.median();
print(composite);
 var rgbVis = {
   min: 161,
   max: 1275,
   bands:["B4", "B3", "B2"]
 };
Map.addLayer(composite, rgbVis, "Median Composite Image");

// clip image with the region of interest
var clippedImage = composite.clip(west)
                             .select("B.*");
print(clippedImage);
Map.addLayer(clippedImage, rgbVis, "Clipped Image");


var avi = clippedImage.expression({
  expression: "(NIR * (1-RED) * (NIR-RED)) ** (1/3)", 
  map: {
    "NIR": clippedImage.select("B8").multiply(0.0001),
    "RED": clippedImage.select("B4").multiply(0.0001),
  }
}).rename("AVI");

var aviVis = {
  min: 0.17786922701253816,
  max: 0.39406195642449504,
  palette: ['#7b3294','#c2a5cf','#a6dba0','#008837']
};
Map.addLayer(avi, aviVis, "AVI_Image" );
