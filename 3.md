// Lesson 3 — Buffers, Reduction functions, and Exporting .csv files

// Import your point dataset (rename your uploaded asset to 'point_count')
var point_count = /* your uploaded table */;

// Load Landsat 8 Surface Reflectance & Temperature dataset for May 2024
var may_images = ee.ImageCollection("LANDSAT/LC08/C02/T1_L2")
  .filterDate('2024-05-01', '2024-06-01');

// Apply scale factors to convert to temperature in Kelvin
function applyScaleFactors(image) {
  var thermal = image.select('ST_B10').multiply(0.00341802).add(149.0);
  return image.addBands(thermal.rename('LST'));
}

var temp_images = may_images.map(applyScaleFactors);

// Reduce the whole image collection into a **mean image**
var mean_LST_image = temp_images.select('LST').mean();

// Create 100-meter buffers around each point
var buffers = point_count.map(function(feature) {
  return feature.buffer(100);
});

// Reduce the buffered zones using the mean LST image
var withLST = mean_LST_image.reduceRegions({
  collection: buffers,
  reducer: ee.Reducer.mean(),
  scale: 30  // Landsat resolution
});

// Export to Google Drive
Export.table.toDrive({
  collection: withLST,
  description: 'Mean_LST_per_Point',
  fileFormat: 'CSV'
});
