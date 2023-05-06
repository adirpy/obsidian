## GDAL安装

前置条件，需要如下包：

* proj
* geos

安装包直接下载，这次是下载的源代码方式。直接

```bash
./configure --prefix=xxx
make
make install
```

即可。
之后安装GDAL，同样采取源代码安装，这里注意需要设置proj的位置，方式如下：

```bash
./configure --prefix=/home/adir/Bin/gdal --with-java=$JAVA_HOME --with-proj=/home/adir/Bin/proj
make
make install
```

安装完成后，设置对应的环境变量

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:"/home/adir/Bin/proj/lib":"/home/adir/Bin/geos/lib":"/home/adir/Bin/gdal/lib"
```



## 编译jar包

前置条件，需要

* apache ant
* swig

ant直接下载压缩包，配置环境变量即可：

```bash
export ANT_HOME="/home/adir/Bin/apache-ant-1.9.15"
```

swig这次没有源代码安装，直接从仓库装的，后续看是否源代码安装一次。
进入GDAL的源代码的目录下的swig/java目录，运行

```bash
make
```

即可，会在目录下生成如下3个文件:

* gdal.jar
* libgdalalljni.la
* libgdalalljni.so

jar放入java的lib目录下，另外两个我这次放在GDAL的安装目录的lib下即可。

## 代码测试

新建一个java项目，代码如下，进行测试：

```java
import java.util.HashMap;
import java.util.Map;

import org.gdal.gdal.Band;
import org.gdal.gdal.gdal;
import org.gdal.ogr.DataSource;
import org.gdal.ogr.Feature;
import org.gdal.ogr.FeatureDefn;
import org.gdal.ogr.FieldDefn;
import org.gdal.ogr.Layer;
import org.gdal.ogr.ogr;
import org.gdal.osr.SpatialReference;

public class App {
 public static void main(String[] args) throws Exception {
    // 指定文件的名字和路径
    //String strVectorFile = "/home/adir/Git/gdal/gdal/data/PDA_RPCITYMUNIC.shp";
    String strVectorFile = "data/PDA_RPCITYMUNIC.shp";
    // 注册所有的驱动
    ogr.RegisterAll();


    // 配置GDAL_DATA路径（gdal根目录下的bin\gdal-data）
    //gdal.SetConfigOption("GDAL_DATA", "F:\\GDAL学习文件夹\\release-1900-x64-gdal-2-3-2-mapserver-7-2-1\\bin\\gdal-data");

    // 为了支持中文路径，请添加下面这句代码
    gdal.SetConfigOption("GDAL_FILENAME_IS_UTF8", "YES");

    // 为了使属性表字段支持中文，请添加下面这句
    gdal.SetConfigOption("SHAPE_ENCODING", "CP936");

    // 读取数据，这里以ESRI的shp文件为例
    String strDriverName = "ESRI Shapefile";

    // 创建一个文件，根据strDriverName扩展名自动判断驱动类型
    org.gdal.ogr.Driver oDriver = ogr.GetDriverByName(strDriverName);
    if (oDriver == null) {
        System.out.println(strDriverName + " 驱动不可用！\n");
        return;
    }
    DataSource dataSource = oDriver.Open(strVectorFile);
    Layer layer = dataSource.GetLayer("PDA_RPCITYMUNIC");

    String layerName = layer.GetName();
    System.out.println("图层名称：" + layerName);

    SpatialReference spatialReference = layer.GetSpatialRef();
    // System.out.println(spatialReference);
    System.out.println("空间参考坐标系：" + spatialReference.GetAttrValue("AUTHORITY", 0)
            + spatialReference.GetAttrValue("AUTHORITY", 1));

    double[] layerExtent = layer.GetExtent();
    System.out.println("图层范围：minx:" + layerExtent[0] + ",maxx:" + layerExtent[1] + ",miny:" + layerExtent[2]
            + ",maxy:" + layerExtent[3]);

    FeatureDefn featureDefn = layer.GetLayerDefn();
    int fieldCount = featureDefn.GetFieldCount();
    Map<String, Object> fieldMap = new HashMap();
    for (int i = 0; i < fieldCount; i++) {
        FieldDefn fieldDefn = featureDefn.GetFieldDefn(i);
        // 得到属性字段类型
        int fieldType = fieldDefn.GetFieldType();
        String fieldTypeName = fieldDefn.GetFieldTypeName(fieldType);
        // 得到属性字段名称
        String fieldName = fieldDefn.GetName();

        fieldMap.put(fieldTypeName, fieldName);
    }

    long featureCount = layer.GetFeatureCount();
    System.out.println("图层要素个数：" + featureCount);
    for (int i = 0; i < featureCount; i++) {
        Feature feature = layer.GetFeature(i);
        Object[] arr = fieldMap.values().toArray();
        for (int k = 0; k < arr.length; k++) {
            String fvalue = feature.GetFieldAsString(arr[k].toString());
            System.out.print(" 属性名称:" + arr[k].toString() + ",属性值:" + fvalue);
        }
        System.out.println();
    }
}

```


