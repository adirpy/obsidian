
```javascript
//经纬度转墨卡托
function lonLat2Mercator(wx,wy)
{
 var mx = wx *20037508.34/180;
 var my = Math.log(Math.tan((90+wy)*Math.PI/360))/(Math.PI/180);
 my = my*20037508.34/180;
 return mx+","+my;
}

//墨卡托转经纬度
function Mercator2lonLat(mx,my)
{
 var wx = mx/20037508.34*180;
 var wy = my/20037508.34*180;
 wy= 180/Math.PI*(2*Math.atan(Math.exp(wy*Math.PI/180))-Math.PI/2);
 return wx+","+wy;
}

//lonLat2Mercator(120,40);
Mercator2lonLat(13358338.893333334,4865942.278825832);
```

