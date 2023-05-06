
WFS版本之间的主要区别是：

*   WFS 1.1.0和2.0.0将GML3返回为默认GML，而在WFS 1.0.0中，默认GML2。GML3采用了稍微不同的方式来指定几何图形。geoserver支持GML3和GML2格式的请求。
*   在WFS 1.1.0和2.0.0中，SRS（空间参考系统或投影）是用 urn\:x-ogc\:def\:crs\:EPSG\:XXXX ，而在WFS 1.0.0中，规范是 <http://www.opengis.net/gml/srs/epsg.xml#XXXX> 。这一变化对 axis order 返回的数据。
*   WFS 1.1.0和2.0.0支持动态重新投影数据，支持在本机SRS以外的SRS中返回数据。
*   WFS 2.0.0引入了新版本的过滤器编码规范，增加了对时间过滤器的支持。
*   WFS 2.0.0支持通过GetFeature请求进行联接。
*   WFS 2.0.0通过 startIndex 和 count 参数。现在，GeoServer在WFS 1.0.0和1.1.0中支持此功能。
*   WFS 2.0.0支持存储查询，这些查询是存储在服务器上的常规WFS查询，因此可以通过向WFS请求传递适当的标识符来调用它们。
*   WFS 2.0.0支持SOAP（简单对象访问协议）作为OGC接口的替代方案。
    注解，参数名称也有两个更改，这会导致混淆。
    WFS 2.0.0使用 count 参数来限制返回的功能的数量，而不是 maxFeatures 以前版本中使用的参数。它也改变了 typeName 到 typeNames 尽管geoserver也会接受。

