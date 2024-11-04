
var dataset = ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')

var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']};

// Applies scaling factors.
function applyScaleFactors(image) {
  var opticalBands = image.select('SR_B.').multiply(0.0000275).add(-0.2);
  var thermalBands = image.select('ST_B.*').multiply(0.00341802).add(149.0);
  return image.addBands(opticalBands, null, true)
              .addBands(thermalBands, null, true);
}

dataset = dataset.map(applyScaleFactors);

var viz = {
  bands: ['SR_B4', 'SR_B3', 'SR_B2'],
  min: 0.0,
  max: 0.3, }

var Img =  dataset.filterBounds(study)
                 .filterDate("2016-06-01","2016-06-28").filterMetadata("CLOUD_COVER", "less_than", 10).sort("CLOUD_COVER",true).first()

var date = ee.Date(Img.get('system:time_start'));
print('Timestamp: ', date)

// Map.addLayer(Clipped_Img, viz, 'lansat8 SR 2016_11:59:18')

var NIR = Img.select('SR_B5');
var RED = Img.select('SR_B4');

var NDVI = NIR.subtract(RED).divide(NIR.add(RED)).rename('NDVI')

var Clipped_NDVI = NDVI.clip(study)

var ndviMin =  ee.Number(Clipped_NDVI.reduceRegion({
                
                reducer   : ee.Reducer.min(),
                geometry: study,
                scale     : 30,
                maxPixels : 1e9
                }).values().get(0))

var ndviMax =  ee.Number(Clipped_NDVI.reduceRegion({
  
                reducer   : ee.Reducer.max(),
                geometry: study,
                scale     : 30,
                maxPixels : 1e9
                }).values().get(0))
                
print(ndviMin)
print(ndviMax)

// Calculate the proportion of vegetation using the NDVI values within the specified range.

// Formula: ((NDVI - NDVI_min) / (NDVI_max - NDVI_min))^2

var Pv = Clipped_NDVI.subtract(ndviMin).divide(ndviMax.subtract(ndviMin)).pow(ee.Number(2)).rename('Pv')
             
var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']}

// Map.addLayer(Pv, ndviParams, 'Proportion of Vegetation');

// Map.addLayer(Clipped_NDVI, ndviParams, 'NDVI');

// Calculate Land Surface Emissivity (EM) using the Fraction of Vegetation (FV).

var Emi_W = ee.Image.constant(0.991);  //Emissivity when NDVI is less than 0, and it is generally water.
var Emi_S = ee.Image.constant(0.996); //Emissivity when NDVI is NDVI values between 0 and 0.2, and it is generally soil.
var Emi_V = ee.Image.constant(0.973); //Emissivity when NDVI is greater than 0.5, and it is generally vegetation area.
   

var emi = ee.Image(0).where(NDVI.gte(-1).and(NDVI.lt(0)), Emi_W)  // water for NDVI < 0
                     .where(NDVI.gte(0).and(NDVI.lt(0.2)), Emi_S)   // Water for 0 <= NDVI < 0.2
                     .where(NDVI.gte(0.2).and(NDVI.lt(0.5)), Emi_V.multiply(Pv).add(Emi_S.multiply(ee.Image(1).subtract(Pv))).add(ee.Image(0.004)))
                      //Emissivity when NDVI is NDVI values between 0.2 and 0.5, and it is generally soil) 
                     .where(NDVI.gte(0.5).and(NDVI.lt(1)), Emi_W) // Vegetation for NDVI >= 0.5


var Clipped_emi =  emi.clip(study)

var Clipped_Img = Img.clip(study)

// Map.addLayer(Clipped_emi, ndviParams, 'EMI image');

var em = Pv.multiply(ee.Number(0.004)).add(Clipped_emi).rename('EM')

var em2 = Pv.multiply(ee.Number(0.004)).add(0.986).rename('EM')

// Select Thermal Band (Band 10) and Rename It
var thermal = Clipped_Img.select('ST_B10').rename('thermal')


// Formula: (TB / (1 + (λ * (TB / 1.438)) * ln(em))) - 273.15
var lst = thermal.expression(
  '(TB / (1 + (0.00115 * (TB / 1.438)) * log(em))) - 273.15', {
    'TB': thermal.select('thermal'), // Select the thermal band (TB)
    'em': em // Assign emissivity (em)
  }).rename('LST');

var lst_2 = thermal.expression(
  '(TB / (1 + (0.00115 * (TB / 1.438)) * log(em))) - 273.15', {
    'TB': thermal.select('thermal'), // Select the thermal band (TB)
    'em': em2 // Assign emissivity (em2)
  }).rename('LST_2')

// var ndviParams = {min: -1, max: 1, palette: ['blue', 'white', 'green']};
// Map.addLayer(NDVI, ndviParams, 'NDVI image')

var minTemp =  ee.Number(lst.reduceRegion({
                
                reducer   : ee.Reducer.min(),
                geometry: study,
                scale     : 30,
                maxPixels : 1e9
                }).values().get(0));

var maxTemp =  ee.Number(lst_2.reduceRegion({
  
                reducer   : ee.Reducer.max(),
                geometry: study,
                scale     : 30,
                maxPixels : 1e9
                }).values().get(0));
                

print(minTemp)
print(maxTemp)

// Add the LST Layer to the Map with Custom Visualization Parameters
Map.addLayer(lst, {
  min: 32.974, // Minimum LST value
  max: 55.350, // Maximum LST value
  
  palette: [
    '040274', '040281', '0502a3', '0502b8', '0502ce', '0502e6',
    '0602ff', '235cb1', '307ef3', '269db1', '30c8e2', '32d3ef',
    '3be285', '3ff38f', '86e26f', '3ae237', 'b5e22e', 'd6e21f',
    'fff705', 'ffd611', 'ffb613', 'ff8b13', 'ff6e08', 'ff500d',
    'ff0000', 'de0101', 'c21301', 'a71001', '911003'
  ]}, 'Land Surface Temperature 2016')
  
Map.addLayer(lst_2, {
  min: 32.974, // Minimum LST value
  max: 55.350, // Maximum LST value
  
  palette: [
    '040274', '040281', '0502a3', '0502b8', '0502ce', '0502e6',
    '0602ff', '235cb1', '307ef3', '269db1', '30c8e2', '32d3ef',
    '3be285', '3ff38f', '86e26f', '3ae237', 'b5e22e', 'd6e21f',
    'fff705', 'ffd611', 'ffb613', 'ff8b13', 'ff6e08', 'ff500d',
    'ff0000', 'de0101', 'c21301', 'a71001', '911003'
  ]}, 'Land Surface Temperature 2016 v2')

Export.image.toDrive({
  image: lst,
  description: 'LST_Charlotte_2016',
  scale: 30,
  region: study,
  crs: 'EPSG:6543', // NAD 1983 (2011) StatePlane North Carolina FIPS 3200 (US Feet)
  fileFormat: 'GeoTIFF',
  formatOptions: {
    cloudOptimized: true
  }
})


var empty = ee.Image().byte();
var outline = empty.paint({
  featureCollection: study,
  color: 1,
  width: 0.2
})

// Display Municipal Boundary
Map.addLayer(outline, {palette: 'purple'}, 'Study Area')
//Center Map with Municipality
Map.centerObject(study,12)
