# Preparando datos Raster
Son varios los problemas que nos encontramos a la hora de optimizar los datos raster. Por un lado debemos encontrar un equilibrio que nos permita obtener una buena experiencia de navegación entre diferentes resoluciones y zooms.

Deberemos evitar en la medida de lo posible abrir un gran número de archivos en cada petición, o sea, disponer de nuestros datos en el número menor de archivos.

Para poder obtener la mejor configuración posible deberemos manejar diferentes formatos, compresiones, tamaños de tesela, etc.

## GeoTIFF
GeoTIFF es un standard que permite georeferenciar la información de un archivo TIFF.

GeoTIFF es un formato muy flexible, y es válido para muchos tipos de casos de almacenamiento de datos. Nos permite cantidad de configuraciones diferentes sobre el formato para adaptarlo a las necesidades puntuales de nuestros datos.

Para trabajar con GeoTIFF será necesario el conocimiento de [GDAL](http://www.gdal.org/), Geospatial Data Abstraction Library, una librería que nos permite el manejo de datos tanto raster (GDAL) como vector (OGR). La mayoría de los software GIS actuales se apoyan en esta librería para realizar operaciones sobre los datos.

Se podrá instalar GDAL para diferentes sistemas operativos. Instrucciones en su [web](https://trac.osgeo.org/gdal/wiki/DownloadingGdalBinaries).

### Georeferenciando al SRS más usado
Nuestros datos estarán en el sistema de referencia de datos en el que hayan sido creados. Por lo general, a la hora de publicarlos, deberemos valorar el sistema de coordenadas en el que vayan a ser más consumidos habitualmente, ya será en el propio de nuestra región, o en uno general como [EPSG:4326](http://spatialreference.org/ref/epsg/4326/) o uno específico de web [EPSG:3857](http://spatialreference.org/ref/sr-org/6864/). GeoServer nos permite la reproyección de los datos al vuelo, pero esto require un consumo de CPU y memoria que penalizará la visualización de los datos en función de la disponibilidad de estos recursos en nuestros servidores.

Lo primero será comprobar el sistema de referencia de nuestros datos, para ello:

```bash
$ gdalinfo /var/local/geoserver/data/TC_NG_Baghdad_IQ_Geo.tif

Driver: GTiff/GeoTIFF
Files: TC_NG_Baghdad_IQ_Geo.tif
       TC_NG_Baghdad_IQ_Geo.tfw
Size is 4322, 4323
Coordinate System is:
GEOGCS["WGS 84",
    DATUM["WGS_1984",
        SPHEROID["WGS 84",6378137,298.257223563,
            AUTHORITY["EPSG","7030"]],
        AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0],
    UNIT["degree",0.0174532925199433],
    AUTHORITY["EPSG","4326"]]
Origin = (44.096343971001403,33.616173643867562)
Pixel Size = (0.000138800000000,-0.000138800000000)
Metadata:
  AREA_OR_POINT=Area
Image Structure Metadata:
  INTERLEAVE=PIXEL
Corner Coordinates:
Upper Left  (  44.0963440,  33.6161736) ( 44d 5'46.84"E, 33d36'58.23"N)
Lower Left  (  44.0963440,  33.0161412) ( 44d 5'46.84"E, 33d 0'58.11"N)
Upper Right (  44.6962376,  33.6161736) ( 44d41'46.46"E, 33d36'58.23"N)
Lower Right (  44.6962376,  33.0161412) ( 44d41'46.46"E, 33d 0'58.11"N)
Center      (  44.3962908,  33.3161574) ( 44d23'46.65"E, 33d18'58.17"N)
Band 1 Block=4322x1 Type=Byte, ColorInterp=Red
Band 2 Block=4322x1 Type=Byte, ColorInterp=Green
Band 3 Block=4322x1 Type=Byte, ColorInterp=Blue

```
Como podemos observar, el sistema de referencia de nuestro datos es **WGS:84** o **EPSG:4326**. Ahora [publicaremos](http://docs.geoserver.org/latest/en/user/data/webadmin/layers.html) nuestros datos en GeoServer en ese sistema de referencia.

Desde JMeter realizaremos una petición al servidor para comprobar la descarga de la imagen. Para ello insertaremos los siguientes datos en los parámetros de JMeter:

* *Nombre o Servidor o IP*: url del servidor, en nuestro caso **192.168.0.12**
* *Puerto*: 8080
* *Ruta*: */geoserver/unredd/wms?service=WMS&version=1.1.0&request=GetMap&layers=unredd:TC_NG_Baghdad_IQ_Geo&styles=&bbox=44.0963439710014,33.01614124386756,44.696237571001404,33.61617364386756&width=767&height=768&srs=EPSG:4326&format=image/png8*

Ahora realizaremos las peticiones en un **sistema de coordenadas diferente**:

* *Nombre o Servidor o IP*: url del servidor, en nuestro caso **192.168.0.12**
* *Puerto*: 8080
* *Ruta*: */geoserver/unredd/wms?service=WMS&version=1.1.0&request=GetMap&layers=unredd:TC_NG_Baghdad_IQ_Geo&styles=&bbox=4908782.556696915,3897446.639583851,4975562.406779059,3977379.2311116955&width=641&height=768&srs=EPSG:3857&format=image/png8*

Seguidamente lo que realizaremos será una reproyección de nuestros datos mediante el uso de GDAL, en este caso [gdalwarp](http://www.gdal.org/gdalwarp.html):

```bash
$ gdalwarp -s_srs EPSG:4326 -t_srs EPSG:3857 TC_NG_Baghdad_IQ_Geo.tif TC_NG_Baghdad_IQ_Geo_3857.tif
```

Ahora tendremos nuestro archivo en el sistema de coordenadas EPSG:3857 que será el que pensamos que va a ser más utilizado en nuestro portal. Podemos comprobar esto mediante *gdalinfo*:

```bash
Driver: GTiff/GeoTIFF
Files: TC_NG_Baghdad_IQ_Geo_3857.tif
Size is 3919, 4691
Coordinate System is:
PROJCS["WGS 84 / Pseudo-Mercator",
    GEOGCS["WGS 84",
        DATUM["WGS_1984",
            SPHEROID["WGS 84",6378137,298.257223563,
                AUTHORITY["EPSG","7030"]],
            AUTHORITY["EPSG","6326"]],
        PRIMEM["Greenwich",0],
        UNIT["degree",0.0174532925199433],
        AUTHORITY["EPSG","4326"]],
    PROJECTION["Mercator_1SP"],
    PARAMETER["central_meridian",0],
    PARAMETER["scale_factor",1],
    PARAMETER["false_easting",0],
    PARAMETER["false_northing",0],
    UNIT["metre",1,
        AUTHORITY["EPSG","9001"]],
    EXTENSION["PROJ4","+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +wktext  +no_defs"],
    AUTHORITY["EPSG","3857"]]
Origin = (4908782.556696915999055,3977379.231111695524305)
Pixel Size = (17.038846858524995,-17.038846858524995)
Metadata:
  AREA_OR_POINT=Area
Image Structure Metadata:
  INTERLEAVE=PIXEL
Corner Coordinates:
Upper Left  ( 4908782.557, 3977379.231) ( 44d 5'46.84"E, 33d36'58.23"N)
Lower Left  ( 4908782.557, 3897450.000) ( 44d 5'46.84"E, 33d 0'58.20"N)
Upper Right ( 4975557.798, 3977379.231) ( 44d41'46.31"E, 33d36'58.23"N)
Lower Right ( 4975557.798, 3897450.000) ( 44d41'46.31"E, 33d 0'58.20"N)
Center      ( 4942170.177, 3937414.616) ( 44d23'46.57"E, 33d19' 0.07"N)
Band 1 Block=3919x1 Type=Byte, ColorInterp=Red
Band 2 Block=3919x1 Type=Byte, ColorInterp=Green
Band 3 Block=3919x1 Type=Byte, ColorInterp=Blue
```

Publicaremos una nueva capa con los nuevos datos y volvemos a realizar la prueba con JMeter con los siguientes datos:

* *Nombre o Servidor o IP*: url del servidor, en nuestro caso **192.168.0.12**
* *Puerto*: 8080
* *Ruta*: */geoserver/unredd/wms?service=WMS&version=1.1.0&request=GetMap&layers=unredd:TC_NG_Baghdad_IQ_Geo_3857&styles=&bbox=4908782.556696916,3897450.000498355,4975557.797535475,3977379.2311116955&width=641&height=768&srs=EPSG:3857&format=image/png8*

### Añadir tileado a la imagen



