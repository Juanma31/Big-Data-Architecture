Brainstorm

-	Quiero usar el dataset de airbnb
-	Quiero enriquecer esos datos con algo para ofrecer un servicio
-	Con que enriquecer?
-	Usando una api del tiempo, mostrar los datos meteorológicos que hay por localización (nombre de la ciudad).
-	
-	Que ofrecer?
-	Una página web estática con distintas listas de alojamientos de Airbnb categorizados por diferentes climas. 
-	Permite filtrar por temperatura, humedad, nieve, viento, etc. 
-	Permite filtrar por países y ciudad.
-	Se actualizan una vez por día.

-	Me gustaría que fuera para elegir destino dependiendo de si se quiere ir a un sitio a esquiar, tomar el sol, hacer windsurf, etc.
-	Quiero usar la api de Yahoo Weather.
-	Quiero usar herramientas google cloud.
-	Me gustaría utilizar elasticsearch y kafka.
-	No quiero usar docker.




Diseño del DAaaS
Definición la estrategia del DAaaS
Una Web que muestre una lista con los mejores alojamientos de Airbnb dependiendo de unos parámetros dados en relación a los datos meteorológicos obtenidos de la API de Yahoo Weather.

Arquitectura DAaaS
La arquitectura se basaría en Google Cloud utilizando los siguientes componentes junto con su finalidad:
1.	Dataprep para borrar columnas innecesarias del CSV
2.	Kafka para enrutar y manejar el stream de datos en tiempo real de la API Yahoo Weather.
a.	El productor sería una instancia de Compute Engine con código que lee de Yahoo Weather y escribe en un topic de kafka.
b.	El consumidor estaría en el cluster de Dataproc con Flume y escribe a GS los resultados
3.	Un Dataproc 24/7h que está ejecutando un job diario para seleccionar los mejores alojamientos según los datos (Yahoo Weather, etc).
4.	Filtrar por categorías los datos meteorológicos (temperatura, viento, humedad, precipitación, horas de sol y presión).
5.	El producto final del proceso de selección de los airbnb es un archivo en formato CSV que se deja en GS.
6.	Cloud SQL para contener los datos generados en GS.
7.	Un API 24-7h en un compute engine que lee del Cloud SQL y hace los filtros.
8.	Una página web que llame a la api y permita al usuario filtrar entre los datos meteorológicos.

DAaaS Operating Model Design and Rollout
1.	Todos los primeros de mes, un administrador descarga a mano el dataset de airbnb y lo meterá en un GS.
2.	Ese mismo usuario va a desplegar a mano un job de Dataprep para limpiar el CSV.
3.	Las máquinas de kafka, el api, el Dataproc y el productor de Yahoo Weather hacia kafka están ejecutándose a diario.
4.	El productor para kafka de Yahoo Weather recoge los datos meteorológicos de todo el mundo.
5.	Cloud scheduler dispara un job que recalcula los datos meteorológicos y los mete en cloud sql cada 6 horas renovando lo que hay en Cloud Storage con el Cloud Function.
6.	Todos los Jobs de generación de archivos de GS desde el Dataproc se van a realizar a mano desde la consola de Google Dataproc. Por lo tanto el cambio de datos lo controlo cuando quiera.
7.	Un cloud scheduler va a sincronizar la base de datos con lo que haya en GS bajo el directorio /output/results a través de un cloud.

Desarrollo de la plataforma DAaaS.
-	Los cloud function los escribiría con Python:
-	Api de google storage.
-	El productor va a estar escrito en python
-	El API lo voy a levantar con Node.
-	En kafka crea una cola.
-	Y tendría un job de Hadoop que cogiera los datos de hdfs de Yahoo Weather, los une con los de airbnb, y me extrae los mejores alojamientos de Airbnb por categorías.


Link a Diagrama:
https://drive.google.com/file/d/1nMBAFpwmxltMTB-Lp_yHMF0slrskDzdq/view?usp=sharing


