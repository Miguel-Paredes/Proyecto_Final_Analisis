# Proyecto_Final_Analisis
## Integrantes
Cataña Dennis, Cocha Iveth, Lascano David, Paredes Miguel, Simba Cristian

## Enlaces de los videos
### Video 1
https://www.youtube.com/watch?v=UKLx3EWgwZQ
<br>
### Video 2
https://www.youtube.com/watch?v=vCOlyQrXaU4
<br>

## Enlace casos de estudio
https://app.powerbi.com/groups/me/reports/da9174b8-daae-4f4b-8ebc-c324bb741555/ReportSection77dfc5e0673f4e73d361?experience=power-bi
## Arquitectura de datos
   ![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/819da3fd-87ab-4985-88cc-9f87dd7b73fe)<br>
   De las temáticas seleccionadas lo que se pretende es enviar algunos dataset hacia MongoDB,CocuchDB Y RavenDB, para leugo exportarlos hacia SQL server, que nos permitira tener una base sobre toda la información de nuestros dataset, con el fin de poder comprender la información generar casos de estudio.<br>
   Cada caso tendrá una imagen, esta visulñaización se realiza en PowerBi que facilita la importación de los datos que ya se encuentran en SQL Server. De esta forma Y ano es necesario crear nuevamente tablas y llenarlas; asi optimizamos el tiempo.


## Librerias a utilizar
<pre> 
import pandas as pd, couchdb, json, os, csv<br>
import pymongo<br>
from pymongo import MongoClient<br>
from pyravendb.store import document_store<br>
from couchdb import Server<br>
</pre> 
<br>**Uso:**
+ **pandas:** Manipula y análiza de Datos 
+ **json, os, csv:** json trabaja con datos en formato JSON (JavaScript Object Notation),"os" proporciona funciones para interactuar con el sistema operativo, "csv" trabaja con archivos CSV (Comma-Separated Values), que son un formato común para el almacenamiento de datos tabulares
+ **pymongo:** Biblioteca para interactuar con bases de datos MongoDB
+ **MongoClient:** Permite conectarte y trabajar con una base de datos MongoDB.
+ **pyravendb.store,document_store:** Interactuar con bases de datos RavenDB, document_store sugiere que se está configurando un almacén de documentos para trabajar con RavenDB.
+ **couchdb,Server:** Interactua con bases de datos CouchDB,la clase Server especifica la biblioteca CouchDB que se utilizará para conectarse a un servidor CouchDB.


## Transformacion del CSV a Json
<pre>   
a = pd.read_csv('chat.csv')<br>
csv_data = pd.read_csv("chat.csv", sep = ",")<br>
csv_data.to_json("chat.json", orient = "records")<br>
</pre>
<br>**Explicación:**
<br>La primera línea lee el archivo csv y los amacena temporalmente en la variable _a_. Esta variable es leída en la segunda línea, con el objetivo de separa los campos con una "_,_". Para finalemnte tomar el archivo csv y convertirlo en un archivo _json_, al cual le daremos un nombre.

## Enviar los archivos Json a Couchdb
<pre>
couch = couchdb.Server('http://admin:admin@localhost:5984')<br>
db_name = 'chat' #los nombres de las bases de datos deben ser en minusculas y con guiones bajos para los espacios<br>
if db_name in couch:<br>
db = couch[db_name]<br>
else:<br>
    db = couch.create(db_name)<br>
archivo_json = 'chat 2.json'<br>
with open(archivo_json) as archivo:<br>
    datos_json = json.load(archivo)<br>
for documento in datos_json:<br>
    db.update([documento])<br>
</pre> 
<br>**Explicación:**
<br>Establecemos conexión con CouchDB y generamos un nombre la Base donde almacenaremos los archivos json.
<br>Con el "_if_" verificamos si ta existe una base de datos en Cocuh bajo ese nombre,en el caso de existir se le ordena que use esa base para guardar la información. Caso contrario entra el "_else_" que primero crea la base para luego cargar datos desde el archivo json seleccionado y agregar esos datos a la base de datos CouchDB como documentos individuales.

## Enviar los archivos Json a Mongodb
<pre> 
client = MongoClient('localhost', 27017)<br>
db = client['Analisis']<br>
collection = db['covid_worldwide']<br>
with open('covid_worldwide.json', 'r') as file:<br>
    data = json.load(file)<br>
result = collection.insert_many(data)<br>
</pre>
<br>**Explicación:**
<br> Para conectrase a MongoDB se estable el usuario y el puerto que se utliza (el puerto ya viene gererado por defecto).
<br>Hay que tener en cuenta que en MongoDB se trabaja con colecciones dentro de cada base, estas colecciones actuan como una clase dentro de la Base de datos, y las colecciones pueden crearse tantas como sean necesarias.
<br>Para crear base de datos llamada "Analisis", debemos generar una colección llamada "covid_worldwide" en donde se alamacenrán nuestra información.
<br> La cuarta línea de código se encarga de aprir el archivo en modo lectura y no hacer modificaciones esto garantiza que cada archivo leido se vaya almecenado  en nuestra colección, el objetivo de abrir en lectura es para que los archivos se guarden individualmente y no como un solo bloque, ya que hacerlos así, se pueden generar incoventientes al momento de viasualiasr la información en Mongo.
<br>Finalmemte en "_insert_many_" insertará los múltiples documentos JSON leídos en la colección. 

## Enviar los archivos Json de Mongodb a Couchdb
<pre> 
client_mongodb = pymongo.MongoClient('localhost', 27017)<br>
db_mongodb = client_mongodb['Analisis']<br>
coleccion_mongodb = db_mongodb['sample_submission']<br>
server = couchdb.Server('http://admin:admin@127.0.0.1:5984/')<br>
db = server['sample_submission']<br>
documentos_mongodb = coleccion_mongodb.find()<br>
for doc in documentos_mongodb:<br>
    doc['mensaje'] = 'Este documento fue extraído de MongoDB' #se agrega un mensaje para indicar que el archivo es extraido de Mongodb<br>
    del doc['_id'] #se elimina el id de mongo para que no entre en conflicto con el id de couchdb<br>
    db.save(doc)<br>
</pre>
<br>**Explicación:**
1. Especificamos en cliente (usuario)  desde donde se enviará los datos de Mongo, de igual manera seleccionamos la Base de datos que contiene la colección donde se encuentran nuestros datos a envia.
2.  Establecemos la ubicación del usuario quién recibe los datos desde Mongo; seleccionamos la Base de datos que contendrá la informacióncontiene la colección donde se encuentran nuestros datos a envia.
3. _documentos_mongodb = coleccion_mongodb.find()_  Recupera todos los documentos de la colección en MongoDB utilizando el método find(). Estos documentos son iterados dentro del "_for_", para asi poder ir agregando uno por uno dentro de la base de Couch
<br>***Nota:***
<br> Los archivos de Mongo ya contenian un ID que los diferenciaba, este Id tuvo que ser eliminado al momento de importar desde Mongo hacia Couch, porque Cocuh identificada ese ID como un campo propio y pordua errores al moento de abrir el archivo.


## Enviar Json a Raven
### Configuración de RavenDB
<pre>
ravendb_url = "http://localhost:8080"<br>
database_name = "Analisis"<br>
store = document_store.DocumentStore(urls=[ravendb_url], database=database_name)<br>
store.initialize()<br>
</pre>
<br>**Explicación:**
<br>Configura un grupo de documentos para trabajar con una base de datos RavenDB al  definir la URL del servidor RavenDB,se identifica la base con la cual se va atrabajar. Luego en la tercera línea se crea un objeto del grupo de documentos utilizando la clase DocumentStore de la biblioteca PyRavenDB, para finalmete establecer la conexión con el servidor.

### Clase Country
<pre>
class Country:<br>
    def __init__(self, **kwargs):<br>
        self.__dict__.update(kwargs)<br>
</pre>
<br>**Explicación:**
<br>Representa la creación de la clase que contendrá los archivos atributos relacionados con información, para luego almacenar dicha información en forma de diccionarios de python con el fin de que se almacenen dinámicamente y asi poder agregar o modificar los campos en un objeto en tiempo de ejecución establecido

### Función para cargar datos de CSV a RavenDB
<pre>
def csv_to_ravendb(csv_file, entity_class):<br>
    with open(csv_file, 'r', encoding='utf-8') as file:<br>
        reader = csv.DictReader(file)<br>
        for row in reader:<br>
            entity = entity_class(**row)<br>
            try:<br>
                with store.open_session() as session:<br>
                    session.store(entity)<br>
                    session.save_changes()<br>
                print(f"Documento {entity.__dict__} subido exitosamente a RavenDB.")<br>
            except Exception as e:<br>
                print(f"Error al subir el documento {entity.__dict__} a RavenDB: {e}")<br>
</pre>
<br>**Explicación:**
<br>La función maneja excepciones y muestra mensajes de éxito o error en función de si la carga de datos en RavenDB se realizó correctamente o no.
En resumen, esta función toma un archivo CSV, lee sus datos, crea objetos de una clase especificada y los almacena en una base de datos RavenDB utilizando una sesión.

### Archivo CSV y clase para los datos
<pre>
csv_file = 'final_train_output.csv'<br>
entity_class = Country<br>
</pre>

### Cargar datos del CSV a RavenDB
<pre>
csv_to_ravendb(csv_file, entity_class)<br>
</pre>
<br>**Explicación:**
<br>Tomamenos la difinicion de la clase (Clase Country), luego con el archivo csv ingresamos la los datos entro de cada campo que se generó en forma de diccionario para crear instancias de la clase a partir de las filas del CSV y cargar los datos en RavenDB de manera correcta.

## Transferir datos de RavenDB a CouchDB
### Configuración de CouchDB
<pre>
couchdb_url = "http://admin:admin@localhost:5984/"<br>
database_name_couch = "olympics_medals_country_wise"<br>
couch = couchdb.Server(couchdb_url)<br>
couchdb_database = couch.get(database_name_couch, None) or couch.create(database_name_couch)<br>
</pre>
<br>**Explicación:**
<br>Este código sirve conectar y trabajar con una base de datos CouchDB, para ob tener la base de datos con el nombre especificado usando elmétodo get(). Si la base de datos no existe (None), entonces se crea una nueva base de datos con ese nombre utilizando el método create(). El resultado se almacena en la variable "_couchdb_database_"

### Función para transferir datos de RavenDB a CouchDB
<pre>
def transfer_data_raven_to_couch(entity_class, couchdb_database):<br>
    with store.open_session() as raven_session:<br>
        entities = list(raven_session.query(entity_class))<br>
        for entity in entities:<br>
            entity_data = {k: getattr(entity, k) for k in dir(entity) if not k.startswith('_')}<br>
            couchdb_database.save(entity_data)<br>

if __name__ == "__main__":<br>
    transfer_data_raven_to_couch(entity_class, couchdb_database)<br>
    print("Transferencia de datos completada.")<br>
</pre>
<br>**Explicación:**
<br>Se tranfieren datos desde una base de datos RavenDB a una base de datos CouchDB, tomando objetos de una clase específica en RavenDB y almacenándolos en CouchDB después de formatearlos como diccionarios.

## Importar a SQL 
1. Crear una base de datos
2. Cargar los archivos en la base de datos, es necesario conocer la ubicación del archivo en el escritorio.
   - En la parte de tareas seleccionamos la opción de importar datos segun el formato del archivo (json,csv,excel)<br>![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/2cfaa1b7-8941-4815-b91a-0186e2c6fa7b)
   - Selecciono la base de datos donde quiero mostrar mi información del archivo<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/c2db3063-4f47-46da-aca3-899f87e90ed7)

3. Se pueden generar graficas de distintos modelos (barras, geograficos, puntos de dispersión), eso se puede modificar según las necesidades del caso de análisis.
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/a91c2a41-b6bc-450c-98cc-1df30a9acae4)<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/44cfd8a4-3ddf-4f73-948a-518d9aa38d6a)<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/a79180e0-5e3b-46ff-82d1-cf96a82e2408)<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/8d506473-2dcb-4c06-9692-8082a31676ec)<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/3e11f469-a652-411c-a17f-4ea2d25f92b0)


## Importar a PowerBI
Seleccionar la opcion de importar base de datos y podemos seleccionar desde donde queremos importar
<br>
![image](https://github.com/Miguel-Paredes/Proyecto_Final_Analisis/assets/117743828/f6c06300-e07b-4ce7-980f-b2b97510a2db)
<br>Escogemos la fuente de nuestros datos, pudoiendo ser desde la base SQL, archivos json, ssv, etc.
## Conclusiones 
1. Lo que se pretende el inicio del trabajo es relacionar distintas bases de datos mediante el envío de información de Datasets.
2. En muchas de las ocasiones es más fácil trabajar con archivos en formato Json ya que no se requiere de transformarlos y se pueden leer fácilmente según la base de datos seleccionada, el caso de Raven los archivos se suvenen formato csv, pero internamente se trabaja como si fuera un archivo json.
3. Es importante conocer y configurar las bases de datos según lo requieran porque cada una se configura de una de manera diferente para la carga de archivos y la importación/exportación de datos.
4. PowerBI es una herramienta que nos ayuda a analizar es los datos mediante gráficas lo cual es una forma mucho más práctica de poder analizar datos más eficientemente


