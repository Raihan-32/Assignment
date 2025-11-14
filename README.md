# Assignment_Unsupervised_code
var upazila = table.filter(ee.Filter.and(
    ee.Filter.eq("NAME_3", "Lohagara"),
    ee.Filter.eq("NAME_2", "Chittagong")
  )
)

// There are two upazila named as "Lohagara" in Bangladesh 
print(upazila)
var sentinel2 = ee.ImageCollection("COPERNICUS/S2")
                .filterBounds(upazila)
                .filterDate("2023-01-01","2023-12-31")
                .filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE",10))
                .median()
                .clip(upazila)
                
var bands = ["B2","B3","B4","B8"] 

var image = sentinel2.select(bands)

Map.centerObject(upazila,10)

var training = image.sample({
  region: upazila.geometry(),
  scale : 10,
  numPixels : 5000
})

var cluster = ee.Clusterer.wekaKMeans(5).train(training)

var result = image.cluster(cluster)
print(result)

Map.addLayer(result.randomVisualizer(),{},"Clustered_dataset")


var ground_point = Settlement.merge(Vegetation).merge(Waterbody).merge(Cropland).merge(Bareland)
 print(ground_point)

 var training =result.sampleRegions({
   collection :  ground_point,
   properties : ["lc"],
   scale : 10
 })
 
 var errorMatrix = training.errorMatrix('lc','cluster');
print('Confusion Matrix:', errorMatrix);
print('Overall Accuracy:', errorMatrix.accuracy())

