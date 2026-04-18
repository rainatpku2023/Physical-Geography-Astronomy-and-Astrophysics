[遥感原理与应用-GEE3.pdf](https://dengbaiyu2023.yuque.com/attachments/yuque/0/2024/pdf/2371504/1733726720676-4a1b6fb0-2fe8-432a-ad9b-3d6a9edee181.pdf)

### 循环
+ for循环无法进行并行运算，因为递推时只能用一个服务器

```javascript
for(variable = initial_val; condition; increment_of_car){
  code...
}
```

+ map方法可以实现并行运算，可以分配好多个服务器。

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733885201683-366c1b7e-20e9-424e-a26c-3468acb41b18.png?x-oss-process=image%2Fcrop%2Cx_0%2Cy_0%2Cw_1036%2Ch_570" width="691" title="" crop="0,0,1,0.9948" id="uc037bec7" class="ne-image">

### Landsat去云 .simpleCloudScore()
+ 常见的遥感影像，如MODIS、Landsat、Sentinel-2等，使用数据时一般需要做去云操作。
+ 基于QA（质量控制图层）去云
+ 基于算法去云

```javascript
function rmCloud(image) { 
  var mask = image.select("cloud").lte(30); // 设置判断为云的阈值
  return image.updateMask(mask); 
 } 
var rawImage = ee.Image("LANDSAT/LC08/C01/T1_TOA/LC08_123032_20210705");
Map.centerObject(rawImage, 7);
var visParams = { 
 bands: ['B4', 'B3', 'B2'], 
  min: 0, 
  max: 0.3 
}; 

//print("rawImage", rawImage); 
//Map.addLayer(rawImage, visParams, "rawImage"); 

var cleanImage = ee.Algorithms.Landsat.simpleCloudScore(rawImage); 
print("cleanImage", cleanImage); 
cleanImage = rmCloud(cleanImage); 
Map.addLayer(cleanImage, visParams, "cleanImage"); 
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733884818974-d9f82062-0b6d-4eb6-b4c8-98268eaf1fed.png" width="1648" title="" crop="0,0,1,1" id="uefe715e1" class="ne-image">

### 计算NDVI
```javascript
var roi = ee.Geometry.Polygon( 
 [[[116.3782558270866, 40.02023566292988], [116.3782558270866, 39.9984096033757], [116.41069982733075, 39.9984096033757], [116.41069982733075, 40.02023566292988]]]); 
Map.addLayer(roi, {color: "red"}, "roi"); Map.centerObject(roi, 13); 
var l8Col = ee.ImageCollection("LANDSAT/LC08/C01/T1_TOA") 
 .filterBounds(roi) 
 .filterDate("2018-1-1", "2018-6-1") 
 .map(function(image){  var ndvi = image.normalizedDifference(["B5", "B4"])  .rename("NDVI");  return image.addBands(ndvi);  }) 
 .select("NDVI") 
 .map(function(image) {  var dict = image.reduceRegion({  reducer: ee.Reducer.mean(),  geometry: roi,  scale: 30 
 });
 var ndvi = ee.Number(dict.get("NDVI"));  image = image.set("ndvi", ndvi);  return image;  }); 
print("l8Col", l8Col); 
var visParam = { 
min: -0.2, 
max: 0.8, 
palette: ["FFFFFF", "CE7E45", "DF923D", "F1B555", "FCD163", 
 "99B718", "74A901", "66A000", "529400", "3E8601",  "207401", "056201", "004C00", "023B01", "012E01",  "011D01", "011301"] }; 
Map.addLayer(l8Col.first().clip(roi), visParam, "l8Col"); 
```

### 缨帽变换（K-T变换）
```javascript
var coefficients = ee.Array([ 
  [0.3037, 0.2793, 0.4743, 0.5585, 0.5082, 0.1863], 
  [-0.2848, -0.2435, -0.5436, 0.7243, 0.0840, -0.1800], 
  [0.1509, 0.1973, 0.3279, 0.3406, -0.7112, -0.4572], 
  [-0.8242, 0.0849, 0.4392, -0.0580, 0.2012, -0.2768], 
  [-0.3280, 0.0549, 0.1075, 0.1855, -0.4357, 0.8085], 
  [0.1084, -0.9022, 0.4120, 0.0573, -0.0251, 0.0238]  //变换矩阵
]); 

//影像转换为array 
var image = ee.Image("LANDSAT/LT05/C01/T1_TOA/LT05_044034_20081011") 
              .select(['B1', 'B2', 'B3', 'B4', 'B5', 'B7']);  //B6是热红外波段，不需要使用
                              //在每个像元上进行矩阵运算，所以要创建每个像元都储存转换矩阵的Image
var arrayImage1D = image.toArray();
var arrayImage2D = arrayImage1D.toArray(1); 

//相乘变换 
var componentsImage = ee.Image(coefficients) 
                        .matrixMultiply(arrayImage2D) 
                        .arrayProject([0]) 
                        .arrayFlatten([[ 
                        'brightness', 'greenness', 'wetness', 
                        'fourth', 'fifth', 'sixth' 
                        ]]); 
var vizParams = { 
"bands": ['brightness', 'greenness', 'wetness'], 
"min": -0.1, 
"max": [0.5, 0.1, 0.1] 
}; 

Map.centerObject(image, 11); 
Map.addLayer(componentsImage, vizParams, "componentsImage"); 
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733885059512-07bf43c1-92f4-409f-95cc-25d7a2c45324.png" width="1567" title="" crop="0,0,1,1" id="u052f529f" class="ne-image"><img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733885095457-6d83da68-5cfc-4eeb-a78f-32030a862ea9.png" width="550.6666666666666" title="" crop="0,0,1,1" id="u2ff30097" class="ne-image">


