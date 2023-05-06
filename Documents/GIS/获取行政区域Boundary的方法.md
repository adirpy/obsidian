1.  BaiduMap API BaiduMap的API直接就可以获取Boundary，具体例子如下：

```javascript
function getBoundary(){       
		var bdary = new BMap.Boundary();
		var name = document.getElementById("districtName").value;
		//获取行政区域
		bdary.get(name, function(rs){   
			//清除地图覆盖物    
			map.clearOverlays(); 
			//行政区域的点有多少个              
			var count = rs.boundaries.length; 
			for(var i = 0; i < count; i++){
				//建立多边形覆盖物
				var ply = new BMap.Polygon(rs.boundaries[i], {strokeWeight: 2, strokeColor: "#ff0000"}); 
				//添加覆盖物
				map.addOverlay(ply);  
				//调整视野
				map.setViewport(ply.getPath());             
			}                
		});   
	}
```

    但是API获取的范围，不知道是否有国际的数据（国内数据是没问题的）

2.  直接从OSMBoundariesMap中获取（可直接导出为shp格式）<https://wambachers-osm.website/boundaries/可以直接选择对应的级别，然后下载为GeoJSON格式。>

3.  从polygons.openstreetmap.fr获取对应URL <http://polygons.openstreetmap.fr/index.py这个需要查找OpenStreetMap对应的RELA_ID来进行导出。>

