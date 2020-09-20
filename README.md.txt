Brainstorm

-	Quiero usar el dataset de airbnb
-	Quiero enriquecer esos datos con algo para ofrecer un servicio
-	Con que enriquecer?
-	Coger de datos de twitter y de alguna manera medir la positividad de los tweets y saber donde se han producido
-	Coger comentarios del dataset y seleccionar los que digan ciertas palabras como “luminoso”, “espacioso”, “divertido”
-	Que ofrecer?
-	Una lista de airbnbs más “positivos” del mundo (voy a permitir filtrar por país, ciudad)
-	Me gustaría utilizar elasticsearch y kafka
-	Me gustaría que fuera algo positivo
-	Me gusta dota
-	No quiero usar docker
-	Plantear un par de ideas a Ricardo
-	Hay un api de X y una web crawleable de Y




Diseño del DAaaS
Definición la estrategia del DAaaS
Una API que tenga una lista actualizada de los Airbnbs más “positivos” de la semana en todo el mundo, basándose en información recolectada de twitter y de comentarios de las reservas contenidos en el dataset.

Arquitectura DAaaS
La arquitectura se basaría en Google Cloud utilizando los siguientes componentes junto con su finalidad:
1.	Dataprep para borrar columnas innecesarias del CSV
2.	Kafka para enrutar y manejar el stream de datos en tiempo real de los tweets
a.	El productor sería una instancia de Compute Engine con código que lee de twitter y escribe en un topic de kafka.
b.	El consumidor estaría en el cluster de Dataproc con Flume y escribe a GS los resultados
3.	Un Dataproc 24/7h que está ejecutando un job diario para seleccionar los mejores sitios según los datos (twitter, etc)
4.	Un elastic para analizar el sentimiento de los tweets, podríamos simplificar y no buscar el sentimiento de los tweets sino filtrar por palabras clave.
5.	El producto final del proceso de selección de los airbnb es un archivo en formato CSV que se deja en GS
6.	Cloud SQL para contener los datos generados en GS.
7.	Un API 24-7h en un compute engine que lee del Cloud SQL y hace los filtros.

DAaaS Operating Model Design and Rollout
1.	Todos los 1eros de mes, un administrador descargara a mano el dataset de airbnb y lo meterá en un GS.
2.	Ese mismo usuario va a desplegar a mano un job de Dataprep para limpiar el CSV.
3.	Las máquinas de kafka, el api, el Dataproc y el productor de tweets hacia kafka están ejecutándose a diario.
4.	El productor para kafka de tweets, lee la lista de palabras consideradas “positivas” desde un fichero de GS.
5.	Voy a tener un cloud function que va a reemplazar ese fichero con las palabras que yo le pase en el cuerpo de la petición por POST.
6.	Todos los Jobs de generación de archivos de GS desde el Dataproc se van a realizar a mano desde la consola de Google Dataproc. Por lo tanto el cambio de datos lo controlo cuando quiera.
7.	Un cloud scheduler va a sincronizar la base de datos con lo que haya en GS bajo el directorio /output/results a traves de un cloud

Desarrollo de la plataforma DAaaS.
-	Los cloud function los escribiría con Python:
-	Api de google storage para el paso 5
-	El productor va a estar escrito en python
-	El api la voy a levantar con Node
-	En kafka tengo que crear una cola
-	Y tendría un job de Hadoop mágico que me cogiera los datos de hdfs de twitter, los uniria con los de airbnb, y me extraería los mejores por location de los tweets vs location de los airbnb.
