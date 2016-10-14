# JMeter
## ¿Qué es JMeter?
Apache JMeter es una aplicación usada para analizar y medir el rendimiento de gran cantidad de servicios, centrado principalmente en servicios web.

Su instalación y ejecución es muy sencilla (debemos tener Java instalado). 

## Instalación de JMeter
### Windows
Para realizar la instalación de JMeter en Windows:

1. Ir a la [página de descargas de JMeter](http://jmeter.apache.org/download_jmeter.cgi) y descargar la versión [*.zip](http://apache.uvigo.es//jmeter/binaries/apache-jmeter-3.0.zip)
2. Descomprima la aplicación en un directorio seleccionado para ello, por ejemplo c:\apache-jmeter.
3. Ejecute el archivo %JMETER_HOME%/bin/jmeter.bat para iniciar la aplicación.

### Linux
1. Ir a la [página de descargas de JMeter](http://jmeter.apache.org/download_jmeter.cgi) y descargar la versión [*.zip](http://apache.uvigo.es//jmeter/binaries/apache-jmeter-3.0.zip) o [.tgz](http://apache.uvigo.es//jmeter/binaries/apache-jmeter-3.0.tgz)
2. Descomprima la aplicación en un directorio seleccionado para ello.
3. Ejecute el archivo %JMETER_HOME%/bin/jmeter.sh para iniciar la aplicación.

## Uso de JMeter

En la ventana principal de JMeter, observamos en la parte izquierda el “Plan de Pruebas” y el “Banco de Trabajo”. Si nos situamos sobre cualquiera de ellos, con el botón derecho del ratón nos aparecerá el menú contextual que nos permite añadir componentes sobre cada uno.

Sobre el “Plan de Pruebas” lo habitual es añadir un “Grupo de Hilos” (Añadir / Hilos (Usuarios) / Grupo de Hilos). Un grupo de hilos representa el número de usuarios que ejecuta nuestro plan de pruebas.

Sobre el “Grupo de Hilos” añadiremos las peticiones que irá realizando cada uno de los hilos. Lo más habitual es que sean peticiones HTTP, por lo que añadiremos una “Petición HTTP” (Añadir / Muestreador / Petición HTTP).

Vemos que en la “Petición HTTP” podemos poner los parámetros de una petición: servidor, protocolo, método (GET, POST, etc.), ruta y demás. Por ahora ponemos:

1. Servidor: IP_GEOSERVER
2. Método: GET

En el “Grupo de Hilos” también añadiremos un “Informe Agregado” (Añadir / Receptor / Informe Agregado). Esto nos creará un pequeño informe con los resultados de la prueba. Hay otros muchos receptores que incluyen gráficas, detalles de la petición y respuesta, etc. También añadiremos un receptor de Gráfico y otro de Gráfico de Resultados.


