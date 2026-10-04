 var pt = ee.Geometry.Polygon(
 [[[29.944314195241628,31.223657145958377],
 [30.031518174733815,31.161395038647903],
 [29.686135484304128,30.808197662577975],
  [29.521340562429128,30.96141036198428],
 [29.708108140554128,31.168445634999095]]]);
 var imgVV = ee.ImageCollection('COPERNICUS/S1_GRD')
        .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))
           .filter(ee.Filter.eq('instrumentMode', 'IW'))
         .filterBounds(pt)
        .select('VV')
        .map(function(image) {
          var edge = image.lt(-30.0);
          var maskedImage = image.mask().and(edge.not());
          return image.updateMask(maskedImage);
        });

var desc = imgVV.filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING'));
var asc = imgVV.filter(ee.Filter.eq('orbitProperties_pass', 'ASCENDING'));

var spring = ee.Filter.date('2019-8-5', '2019-8-10');
var lateSpring = ee.Filter.date('2019-8-11', '2019-8-15');
var summer = ee.Filter.date('2019-8-16', '2019-8-20');

var descChange = ee.Image.cat(
        desc.filter(spring).mean(),
        desc.filter(lateSpring).mean(),
        desc.filter(summer).mean());

var ascChange = ee.Image.cat(
        asc.filter(spring).mean(),
        asc.filter(lateSpring).mean(),
        asc.filter(summer).mean());
        
Map.setCenter(29.6861,30.8081, 12);
Map.addLayer(ascChange, {min: -25, max: 5}, 'Multi-T Mean ASC', true);
Map.addLayer(descChange, {min: -25, max: 5}, 'Multi-T Mean DESC', true);

Export.image.toDrive({
 image: descChange,
 description: 'descChange2019-8-1-2019-8-25',
 scale: 10,
 region: pt,
 maxPixels: 3E10

});
