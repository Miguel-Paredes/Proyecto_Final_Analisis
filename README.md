# Proyecto_Final_Analisis
## Integrantes
Cataña Dennis, Cocha Iveth, Lascano David, Paredes Miguel, Simba Cristian
## Librerias a utilizar
import pandas as pd, couchdb, json, os, csv<br>
import pymongo<br>
from pymongo import MongoClient<br>
from pyravendb.store import document_store<br>
from couchdb import Server<br>
<pre>    
## Transformacion del CSV a Json
a = pd.read_csv('chat.csv')<br>
csv_data = pd.read_csv("chat.csv", sep = ",")<br>
csv_data.to_json("chat.json", orient = "records")<br>

## Enviar los archivos Json a Couchdb
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

## Enviar los archivos Json a Mongodb
client = MongoClient('localhost', 27017)<br>
db = client['Analisis']<br>
collection = db['covid_worldwide']<br>
with open('covid_worldwide.json', 'r') as file:<br>
    data = json.load(file)<br>
result = collection.insert_many(data)<br>

## Enviar los archivos Json de Mongodb a Couchdb<br>
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

## Enviar Json a Raven
### Configuración de RavenDB
ravendb_url = "http://localhost:8080"<br>
database_name = "Analisis"<br>
store = document_store.DocumentStore(urls=[ravendb_url], database=database_name)<br>
store.initialize()<br>

### Clase Country
class Country:<br>
    def __init__(self, **kwargs):<br>
        self.__dict__.update(kwargs)<br>

### Función para cargar datos de CSV a RavenDB
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

### Archivo CSV y clase para los datos
csv_file = 'final_train_output.csv'<br>
entity_class = Country<br>

### Cargar datos del CSV a RavenDB
csv_to_ravendb(csv_file, entity_class)<br>

## Transferir datos de RavenDB a CouchDB

### Configuración de CouchDB
couchdb_url = "http://admin:admin@localhost:5984/"<br>
database_name_couch = "olympics_medals_country_wise"<br>
couch = couchdb.Server(couchdb_url)<br>
couchdb_database = couch.get(database_name_couch, None) or couch.create(database_name_couch)<br>

### Función para transferir datos de RavenDB a CouchDB
def transfer_data_raven_to_couch(entity_class, couchdb_database):<br>
    with store.open_session() as raven_session:<br>
        entities = list(raven_session.query(entity_class))<br>
        for entity in entities:<br>
            entity_data = {k: getattr(entity, k) for k in dir(entity) if not k.startswith('_')}<br>
            couchdb_database.save(entity_data)<br>

if __name__ == "__main__":<br>
    transfer_data_raven_to_couch(entity_class, couchdb_database)<br>
    print("Transferencia de datos completada.")<br>

