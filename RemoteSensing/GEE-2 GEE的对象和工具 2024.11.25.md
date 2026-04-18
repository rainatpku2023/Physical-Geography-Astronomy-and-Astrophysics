[遥感原理与应用-GEE2.pdf](https://dengbaiyu2023.yuque.com/attachments/yuque/0/2024/pdf/2371504/1732511033679-2aab32d4-9c78-4405-840b-d864bb3d6b78.pdf)

[https://code.earthengine.google.com/](https://code.earthengine.google.com/)

### 写在前面
1. 官方教程

[https://developers.google.com/earth-engine/apidocs](https://developers.google.com/earth-engine/apidocs)

2. 对象语法可以在Docs处查找。

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732630664835-63814b57-7192-45e5-a4bc-a063f1baa6e8.png" width="1920" title="" crop="0,0,1,1" id="u8cc70571" class="ne-image">

### ee.Map
```javascript
//Prints the viewport of the map
print(Map.getBounds())

//Center map at lon,lat,zoom level
Map.setCenter(116.3097,39.9936) //将画面中心点设置为(116.3097,39.9936)

// Create a rectangle, center it and display it on map
var rectangle = ee.Geometry.Rectangle(116.3065,39.9960, 116.3146,39.9889) //左上角和右下角坐标
Map.centerObject(rectangle) //移动视野，使矩形区域位于视野中央，而且比例尺合适
Map.addLayer(rectangle) //显示


Map.setOptions("SATELLITE"); // 卫星影像作为背景进行显示
var image = ee.Image("LANDSAT/LC08/C02/T1_TOA/LC08_123032_20210705"); 
  // landsat文件需要日期和row和path -> 即位置
Map.centerObject(image, 7); // 7为缩放指标
var visParam = { 
 min: 0, // 数据拉伸，0*255成为最暗
 max: 0.3,  // 数据拉伸，0.3*255成为最亮
 bands: ["B4", "B3", "B2"]  //真彩色
  //bands: ["B5", "B4", "B3"]  //假彩色
}; 
Map.addLayer(image, visParam, "rawImage");
```

##### 
```javascript
Map.addLayer(eeObject, visParams, name, shown, opacity)
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732512362290-55f92ade-6b6a-4758-a39b-01f9e6a74b58.png" width="1074" title="" crop="0,0,1,1" id="uf81fd2c1" class="ne-image">

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732512684492-e613b665-8ec5-4edb-9c1b-2376292f008c.png" width="1070" title="" crop="0,0,1,1" id="uc94b5586" class="ne-image">

### ee.Image
+ ⽤来将结果展示或下载结果
+ 可以是单⼀波段或多个波段
+ ⼀张影像可以覆盖全球

```javascript
Map.addLayer(eeObject, visParams, name, shown, opacity)
```

```javascript
var img1 = ee.Image('MOD09GA/MOD09GA_005_2012_03_09'); // From name 输入文件
var img2 = ee.Image(1) //Constant value 将每个栅格亮度都改成1
var img3 = ee.Image() //Empty transparent Image 空值
print (img1)
Map.addLayer(img2,{},'equal_to_1');
Map.addLayer(img1.select(['sur_refl_b01','sur_refl_b04','sur_refl_b03']) // 分别显示蓝绿红波段
             ,{gain:"0.0255,0.0255,0.0255"},
             'MOD09GA');
Map.addLayer(img3,{},'transparent');
```

+ 获取波段名称的方法：在搜索框处查找
+ modis的scaler问题，比例。

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732513244806-ac108824-f5a9-47fe-8934-6d0c9343c70c.png" width="1920" title="" crop="0,0,1,1" id="u39b8886b" class="ne-image">

+ Image的一些运算
    - Algebraic: add, Subtract, Multiply, Divide
    - Trigonometric: sin, cos, tan, asin, acos, atan
    - Binary: eq, gt, gte, lte, and, or, not, leftShift, rightShift
    - Type: toInt16, toInt32, toLong, toString, toFloat, ...
    - Other: where, mask, normalizedDifference
    - 更多的函数可以参见说明文档（侧面Docs辅助查询）

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732513552801-dced6349-414f-40b8-8bfc-fb061bf70029.png" width="1920" title="" crop="0,0,1,1" id="u2d7536c6" class="ne-image">

```javascript
var image1 = ee.Image('MODIS/MOD09A1/MOD09A1_005_2010_02_02') //Load image 1
var image2 = ee.Image('MODIS/MOD09A1/MOD09A1_005_2011_02_10') //load image 2
var ndvi1 = image1.normalizedDifference(['sur_refl_b02','sur_refl_b01']) //

//Shortcut
var ndvi2 = image2.expression('(NIR-RED)/(RED+NIR)',{ //Alternative way，使用expression方法
 RED:image2.select(['sur_refl_b01']), //在后面制定变量值
  NIR:image2.select(['sur_refl_b02']) 
});

var ndviDifference = ndvi2.subtract(ndvi1) //Calculate their difference

//This palette will help giving colors to visualization
var palette = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
               '74A901', '66A000', '529400', '3E8601', '207401', '056201',
               '004C00', '023B01', '012E01', '011D01', '011301'];
Map.addLayer(ndviDifference,{min:-1,max:1,palette:palette})
// 'FFFFFF'对应最暗，'011301'对应最亮，中间等距分割
```

```javascript
//错误的写法
var image1 = ee.Image('MODIS/MOD09A1/MOD09A1_005_2010_02_02') //Load image 1

var EVI1 = image1.expression('2.5*(NIR-RED)/(6*RED+NIR-7.5*BLUE+1)',{
  //错误的写法，没有考虑到比例问题
  RED:image1.select(['sur_refl_b01']), //在后面制定变量值
  NIR:image1.select(['sur_refl_b02']),
  BLUE:image1.select(['sur_refl_b03'])
});

Map.addLayer(EVI1,{min:0,max:1})


//正确的写法
var image1 = ee.Image('MODIS/MOD09A1/MOD09A1_005_2010_02_02') //Load image 1

var EVI1 = image1.expression('2.5*(NIR-RED)/(6*RED+NIR-7.5*BLUE+10000)',{
  //正确的写法，需要考虑到scale的问题
  RED:image1.select(['sur_refl_b01']), //在后面制定变量值
  NIR:image1.select(['sur_refl_b02']),
  BLUE:image1.select(['sur_refl_b03'])
});

Map.addLayer(EVI1,{min:0,max:1})
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732515320981-a70615dc-47ef-4328-8a08-3d0434ff9582.png" width="1920" title="" crop="0,0,1,1" id="uef8fb8ce" class="ne-image">

```javascript
var image = ee.Image('USGS/SRTMGL1_003') //SRTM Digital Elevation Data 30m，全球30m分辨率高程信息

var altNaiveClass = ee.Image(0)
//Start with an image of zeros，新建全部以0值填充的影像

altNaiveClass = altNaiveClass.where(image.lt(500),1) //1: <500m；小于500m的位置设置成1
altNaiveClass = altNaiveClass.where(image.gte(500).and(image.lt(1000)),2) //.and语句表示“并且”
//2: [500m-1000m)；大于等于500m且小于1000的位置设置成2

altNaiveClass = altNaiveClass.where(image.gte(1000),3)
// 3: >=1000m，大于1000m的位置设置成3

altNaiveClass = altNaiveClass.mask(altNaiveClass) 
//Removes all Zeros，将0的位置设成空值，即透明

Map.addLayer(altNaiveClass)
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732515260204-b92c4c86-f03f-4402-b8bd-debfcb9c3f6f.png" width="1920" title="" crop="0,0,1,1" id="u7844b818" class="ne-image">

### ee.ImageCollection
+ ImageCollection 的⼀些操作
    - Statistical aggregation: min, max, mean, median, mode, count
    - Binary: and, or, not
    - Algebraic Aggregation: sum, product, ...
    - Other: merge, map, reduce, ﬁlterDate, ﬁlterBounds, cast, ﬁrst, …

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732515420618-5988a7db-2fa3-4535-b68a-f13f6cc79d21.png?x-oss-process=image%2Fcrop%2Cx_0%2Cy_4%2Cw_1114%2Ch_624" width="743" title="" crop="0,0.006,1,1" id="OlTku" class="ne-image">

```javascript
var myCollection = ee.ImageCollection('MODIS/MOD11A2')
  .filterDate(new Date(2000,1,1),new Date(2001,1,1))//按照时间过滤

var medianLST = myCollection.select(['LST_Night_1km'])
  .median() 
  // Select median value, now we have an ee.Image object；对ImageCollection中的每个Image的每个像素取中间值
  .toFloat() // Transform to float，转换为float数据类型
  .multiply(ee.Image(0.02))
  //this band has a 0.02 scale，调整scale使合适，和全部为0.02的Image各个像元做乘法
  .subtract(ee.Image(273.15))
  //Transform Kelvin to Celsius，数值减去273.15得到摄氏度读数，和全部为273.15的做减法
print(myCollection,{min:-30,max:30})
print(medianLST) //Just for debugging
Map.addLayer(medianLST,{min:-30,max:30}) 
```

```javascript
function getNDVI (image){
  return image.normalizedDifference(['sur_refl_b02','sur_refl_b01'])
   .select([0],['ndvi']) //Give name to the band and return
}

var year = 2008;
var collection = ee.ImageCollection("MODIS/MOD09A1")
  .filterDate(new Date(year,1,1),new Date(year+1,1,1)) //按照日期选择
print(collection)
collection = collection.map(getNDVI);
//对每个pixel进行求最大值，把collection中每个图片的不同波段压缩成一个只剩下NDVI
//map方法对collection中每个日期的图片每个像素求NDVI，压缩成365个单层NDVI图像
//原来的数据丢掉了
print(collection)

var maxNDVI = collection.max()
print(maxNDVI)
Map.addLayer(maxNDVI)
//选出最大NDVI，作为最终图像像素灰度


//另外一种写法
function getNDVI(image){
  var Img = image.normalizedDifference(['sur_refl_b02','sur_refl_b01']) 
  return image.addBands(Img.rename('ndvi'))
}
//这种方法实际上新建了一个波段，所以没有数据丢失
```

```javascript
//如何向Map方法传入数据
function getNdviFromThreshold(min,max){
  return function(image){
    image = image.normalizedDifference(['sur_refl_b02','sur_refl_b01']) 
    image = image.where(image.gt(ee.Image(max)),max) //Put zeros in values out of range 
    image = image.where(image.lt(ee.Image(min)),min) //Put zeros in values out of range 
    return image.select([0],['ndvi']) //Give name to the band and return its value }
}
var year = 2008, min = 0.1, max = 1;
var collection = ee.ImageCollection("MODIS/MOD09A1")
.filterDate(new Date(year,1,1),new Date(year+1,1,1));
print(collection);
collection = collection.map(getNdviFromThreshold(min,max));
print(collection);
var maxNDVI = collection.mean();
print(maxNDVI);
Map.addLayer(maxNDVI,{max:max,min:min});
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732517254264-d68a0d8f-a1d1-49d3-ae5a-dffbe92eda17.png" width="808" title="" crop="0,0,1,1" id="ubc1d6d54" class="ne-image">

### ee.Geometry
+ Geometry是几何形状的统称
    - Point / MultiPoint
    - Polygon, Multipolygon
    - Rectangle (useful shortcut)
    - from geoJSON

```javascript
var rectangle = ee.Geometry(ee.Geometry.Rectangle(115.5,39.5,117.5,41),'EPSG:4326',false)
var image = ee.Image(('LANDSAT/LC08/C01/T1_TOA/LC08_123032_20210705'))
var ndvi = image.normalizedDifference(['B5','B4'])

ndvi = ndvi.clip(rectangle)
var palette = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718',
               '74A901', '66A000', '529400', '3E8601', '207401', '056201',
               '004C00', '023B01', '012E01', '011D01', '011301'];

Map.addLayer(ndvi,{min:0,max:.1,palette:palette},'ndvi in the polygon')
```

```javascript
var rectangle = ee.Geometry(ee.Geometry.Rectangle(115.5,39.5,117.5,41),'EPSG:4326',false)

var myImages = ee.ImageCollection((‘LANDSAT/LC08/C01/T1_TOA'))
  .filterDate('2021-01-01', '2021-06-01').filterBounds(rectangle) 
  //.filterBounds()是ImageCollection的方法

print(myImages)
```

### ee.Feature
```javascript
//Sample layer NLCD
var a = ee.Image('USGS/NLCD/NLCD2001')

//Select only water and snow/ice pixels (11 or 12)
var b = ee.Image(0).where(
        a.select('landcover').eq(11).or(a.select('landcover').eq(12)),
        1)//将属于水和雪的部分都设置为1，其他的保持为0。

Map.addLayer(b.mask(b),{palette:'0000FF'}) //把值为1处保留，其他的都mask掉
Map.addLayer(a)
var point1 = ee.Geometry.Point([-87.863438, 43.393651])  
var point2 = ee.Geometry.Point([-87.8637768, 43.3936994])
Map.addLayer(point1,{color:"00ff00"},'Point1')
Map.addLayer(point2,{},'Point2')
Map.centerObject(point1)
print('Point1 (water)');print(a.sampleRectangle(point1))
print('Point2 (no_water)');print(a.sampleRectangle(point2))
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733721657184-fb37cb03-e50c-4487-9878-1849e2bc3fdb.png" width="1920" title="" crop="0,0,1,1" id="u353d9815" class="ne-image">

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733721711832-fd63c614-071a-43f1-bfc8-6d5d5600dd11.png" width="1920" title="" crop="0,0,1,1" id="uc2d2b19e" class="ne-image">

```javascript
//Sample layer NLCD
var a = ee.Image('USGS/NLCD/NLCD2001')

//Select only water and snow/ice pixels (11 or 12)
Map.addLayer(a,{})

var point1 = ee.Geometry.Point([-87.863438, 43.393651])
var point2 = ee.Geometry.Point([-87.8637768, 43.3936994])
Map.addLayer(point1,{color:"00ff00"},'Point1')
Map.addLayer(point2,{},'Point2')
Map.centerObject(point1)

print('Point1 (water)');print(a.sampleRectangle(point1))
print('Point2 (no_water)');print(a.sampleRectangle(point2))

var featureCollection =ee.FeatureCollection([a.sampleRectangle(point1),a.sampleRectangle(point2)])
print(featureCollection)
//创建Feature Collection的方法：用中括号将Features括起来即可
//FeatureCollection类是不同Feature的集合
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732627975050-951d0121-daf5-457f-9b11-7727d75f7504.png" width="1920" title="" crop="0,0,1,1" id="uc8cc3eca" class="ne-image">

#### 如何上传自己的文件
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1733722345010-d400dcc5-fe13-44d0-b2c2-1db0239db397.png" width="1920" title="" crop="0,0,1,1" id="ub1a79441" class="ne-image">

### ee.Reducer
+ 时间上
    - imageCollection.reduce() 
+ 空间上
    - image.reduceRegion()     image.reduceNeighborhood()
+ 波段上
    - image.reduce()
+ 矢量集合的属性上
    - featureCollection.reduceColumns()

#### image.reduce()方法使用示例
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732628934702-b345f85e-8e61-4b0e-88fc-32e9efbcb4df.png" width="920" title="" crop="0,0,1,1" id="ub54a5496" class="ne-image">

```javascript
// Load an image and select some bands of interest.
var image = ee.Image('LANDSAT/LC08/C02/T1/LC08_044034_20140318')
    .select(['B4', 'B3', 'B2']);

// Reduce the image to get a one-band maximum value image.
var maxValue = image.reduce(ee.Reducer.max());

// Display the result.
Map.centerObject(image, 10);
Map.addLayer(maxValue, {max: 13000}, 'Maximum value image');
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732628659622-ca904c8b-efdc-48e1-b345-4fc063e4539e.png" width="1920" title="" crop="0,0,1,1" id="ud3a8676d" class="ne-image">

#### image.reduceRegion()方法使用示例
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732629070755-0fca3351-a1da-41a3-8c17-762afc5e8e49.png" width="920" title="" crop="0,0,1,1" id="uc33f688b" class="ne-image">

```javascript
// Load input imagery: Landsat 7 5-year composite.
var image = ee.Image('LANDSAT/LE7_TOA_5YEAR/2008_2012');

// Load an input region: Sierra Nevada.
var region = ee.Feature(ee.FeatureCollection('EPA/Ecoregions/2013/L3')
  .filter(ee.Filter.eq('us_l3name', 'Sierra Nevada'))
  .first());

// Reduce the region. The region parameter is the Feature geometry.
var meanDictionary = image.reduceRegion({
  reducer: ee.Reducer.mean(),
  geometry: region.geometry(),
  scale: 30,
  maxPixels: 1e9
});

// The result is a Dictionary.  Print it.
print(meanDictionary);
// As an alternative to specifying scale, specify a CRS and a CRS transform.
// Make this array by constructing a 4326 projection at 30 meters,
// then copying the bounds of the composite, from composite.projection().
```

#### imageCollection.reduce()方法使用示例
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732629411355-e5f18dd5-6dd3-49c4-ad59-8059f06e79ef.png" width="918" title="" crop="0,0,1,1" id="ue9b523d6" class="ne-image">

```javascript
// Load an image collection, filtered so it's not too much data.
var collection = ee.ImageCollection('LANDSAT/LT05/C02/T1')
  .filterDate('2008-01-01', '2008-12-31')
  .filter(ee.Filter.eq('WRS_PATH', 44))
  .filter(ee.Filter.eq('WRS_ROW', 34));

// Compute the median in each band, each pixel.
// Band names are B1_median, B2_median, etc.
var median = collection.reduce(ee.Reducer.median());

// The output is an Image.  Add it to the map.
var vis_param = {bands: ['B4_median', 'B3_median', 'B2_median'], gamma: 1.6};
Map.setCenter(-122.3355, 37.7924, 9);
Map.addLayer(median, vis_param);
```

<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732629270558-09ea6810-ea55-4d30-9341-dada7ab728bd.png" width="1920" title="" crop="0,0,1,1" id="aA8iO" class="ne-image">

#### featureCollection.reduceColumns()使用示例
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732629488941-20529477-d928-493e-bba3-5e7046cee77f.png" width="916.6666666666666" title="" crop="0,0,1,1" id="uc289f546" class="ne-image">

### 作业
<img src="https://cdn.nlark.com/yuque/0/2024/png/2371504/1732517499036-02d52671-5a03-4a30-8b83-05d0d4657f8d.png" width="886.6666666666666" title="" crop="0,0,1,1" id="d3YRY" class="ne-image">

```javascript
function getNdviImage(image){
  var ndviImage = image.normalizedDifference(["B4","B3"])
  return ndviImage
}

function getMeanNdviImage(startDate,endDate){
  var rawImageCollection = ee.ImageCollection('LANDSAT/LT05/C02/T1')
        .filterDate(String(startDate),String(endDate));

  var ndviImageCollection = rawImageCollection.map(getNdviImage)
  var meanNdviImage = ndviImageCollection.mean();

  return meanNdviImage
}

function main(){
  var frameRectangle = ee.Geometry.Rectangle(-72.46,42.042,-72.037,42.176);
  Map.centerObject(frameRectangle)
  Map.addLayer(frameRectangle)
  
  var meanNdviImage1 = getMeanNdviImage("2011-06-01","2011-06-30");
  var meanNdviImage2 = getMeanNdviImage("2010-10-01","2010-10-31");
  var differenceNdviImage = meanNdviImage1.subtract(meanNdviImage2);
  
  Map.addLayer(meanNdviImage1,{min:-1,max:1},"meanNdviImage1",0,1);
  Map.addLayer(meanNdviImage2,{min:-1,max:1},"meanNdviImage2",0,1);

  var paletteBlueToRed =  ["0022FF","1122EE","2222DD","3322CC","4422BB","5522AA",
                "662299","772288","882277","992266","AA2255","BB2244",
                "CC2233","DD2222","EE2211","EE2211"];
  
  Map.addLayer(differenceNdviImage,{min:-1,max:1,palette:paletteBlueToRed},"differenceNdviImage",0,1);
  var clippedImage = differenceNdviImage.clip(frameRectangle)
  
  Map.addLayer(clippedImage,{min:-1,max:1,palette:paletteBlueToRed},"clippedImage",1,1);

  alert("Sharper decrease in NDVI occurs in bluer pixels.")
}

main()
```


