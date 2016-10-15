# Preparando datos vectoriales

## Configuraciones preliminares

A la hora de publicar datos vectoriales podemos hacer las elecciones significativas siguientes:

* Publicar en formato shapefile
* Meter los datos en una base de datos postgis

Se parte de la máquina virtual, donde se han ejecutado los siguientes comandos:

	sudo update-alternatives --config java
	sudo dpkg --configure -a
	sudo dpkg --purge linux-generic

Se configuró el JDK de oracle, con las JAI e ImageIO instalado. Se aumentó la memoria de la máquina virtual a 2Gb y se estableció la memoria de Tomcat a 1Gb
	
Partimos de datos de Open Street Map de Toulouse, que ocupan unos 80Mb en Shapefile. Estos datos son cargados en PostGIS y se crean dos capas en GeoServer, una que accede al shapefile llamada `toulouse_shp` y otra que accede a la tabla en PostGIS llamada `toulouse_postgis`.

Para que el uso de PostGIS tenga sentido, es necesaria la creación de índices tanto en la clave primaria como en la columna geométrica. Por tanto, tras realizar la carga:

	USER=portal_admin
	DB=portal
	psql -U $USER -d $DB -c "CREATE SCHEMA gis;"
	shp2pgsql -s 4326 gis.osm_buildings_v06.shp gis.toulouse | psql -U $USER -d $DB

ejecutaremos los siguientes comandos:

	psql -U $USER -d $DB -c "create index toulouse_geom_gix ON gis.toulouse using gist(geom)"
	psql -U $USER -d $DB -c "vacuum analyze gis.toulouse"

Para Shapefile también estamos utilizando un índice pero éste lo crea automáticamente GeoServer... si el usuario `Tomcat7`, que ejecuta GeoServer, tiene permisos para acceder al directorio. En caso de tener problemas de rendimiento con un Shapefile es conveniente revisar el log de GeoServer en busca de algo similar a esto:

	14 oct 05:11:52 ERROR [data.shapefile] - /var/geoserver/shapefiles/gis.osm_buildings_v06.qix (Permiso denegado)
	java.io.FileNotFoundException: /var/geoserver/shapefiles/gis.osm_buildings_v06.qix (Permiso denegado)

En un principio un shapefile dará mejor rendimiento que PostGIS ya que es una forma de acceder a los datos más directa. Para obtener un rendimiento mayor con PostGIS hay que trabajar un poco más y es muy conveniente obtener el feedback inmediato de lo que GeoServer pide a PostgreSQL. Para ello habilitaremos el log estableciendo esta opción en su fichero de configuración `/etc/postgresql/9.5/main/postgresql.conf`

	log_statement = 'all'

A continuación reiniciaremos el servidor:

	sudo service postgresql restart

y abriremos una sesión en el servidor y mostraremos constantemente el log:

	tail -f /var/log/postgresql/postgresql-9.5-main.log

Para monitorizar también posibles problemas de memoria ejecutaremos el comando `htop` en otro terminal:

![](_images/vector/htop.png)

## Pruebas

Las siguientes pruebas realizarán una petición a la extensión máxima de la capa:

![](_images/vector/full-extent.png)

	http://192.168.0.27:8080/geoserver/geomatico/wms?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&FORMAT=image%2Fpng&TRANSPARENT=true&STYLES&LAYERS=geomatico%3Atoulouse_postgis&SRS=EPSG%3A4326&WIDTH=200&HEIGHT=200&BBOX=1.2668609619140625%2C43.40080261230469%2C1.6788482666015625%2C43.81278991699219

y a una parte pequeña de ella:

![](_images/vector/small-extent.png)

	http://192.168.0.27:8080/geoserver/geomatico/wms?SERVICE=WMS&VERSION=1.1.1&REQUEST=GetMap&FORMAT=image%2Fpng&TRANSPARENT=true&STYLES&LAYERS=geomatico%3Atoulouse_pg&SRS=EPSG%3A4326&WIDTH=200&HEIGHT=200&BBOX=1.461317092180252%2C43.62045407295227%2C1.4621217548847198%2C43.62125873565674

### extensión completa

Las peticiones son extremadamente lentas, sobre todo en el caso de PostGIS. Para evitar colapsar el servicio se configura jmeter para que encadene una petición detrás de otra, dándole tiempo al sistema a terminar.

Podemos observar que el shapefile es 10 veces más eficiente que la base de datos. El índice espacial que hemos creado no se puede aprovechar ya que se quiere leer el juego de datos completo y la transferencia de datos se hace de una forma más lenta.

Se puede jugar con el parámetro `fetch_size` del datastore para aumentar el número de registros que se recuperan en una conexión pero esto sólo tiene sentido cuando el servidor de mapas y de base de datos tienen una latencia importante entre ellos. En general la dirección que se debe de tomar es la opuesta: en lugar de aumentar el buffer de lectura con la base de datos se debe reducir la cantidad de elementos que se leen, como veremos en el apartado de estilos.

**Shapefile**

![](_images/vector/shp-full-sequential.png)

* *Nombre o Servidor o IP*: url del servidor, en nuestro caso **192.168.0.12**
* *Puerto*: 8080
* *Ruta*: 

**PostGIS**

![](_images/vector/pg-full-sequential.png)

**Concurrencia en shapefile**

Sin embargo el tiempo de respuesta, incluso con Shapefile es inaceptable para un servidor que vaya a tener un uso intensivo. La siguiente imagen muestra los resultados haciendo 240 peticiones en 1m40s.

![](_images/vector/shp-full-concurrent.png)

### extensión pequeña

Vemos ahora que el rendimiento es mucho mejor ya que en ambos formatos se filtra con la ayuda de los índices y se encuentran rápidamente los objetos que hay que pintar.

Se lanzan 240 peticiones en 30 segundos y no se produce ningún colapso en el servidor, ya que las peticiones se resuelven instantáneamente. PostGIS sigue funcionando más lento pero ya a un nivel casi inapreciable.

**Shapefile**

![](_images/vector/shp-small-sequential.png)

* *Nombre o Servidor o IP*: url del servidor, en nuestro caso **192.168.0.12**
* *Puerto*: 8080
* *Ruta*: 

**PostGIS**

![](_images/vector/pg-small-sequential.png)

## Conclusiones

Shapefile es más eficiente que PostGIS. Be water my friend.




