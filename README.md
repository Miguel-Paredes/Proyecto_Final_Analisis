# Proyecto_Final_Analisis
## Integrantes
Cataña Dennis, Cocha Iveth, Lascano David, Paredes Miguel, Simba Cristian
## Librerias a utilizar
import pandas as pd, couchdb, json, os, csv<br>
import pymongo<br>
from pymongo import MongoClient<br>
from pyravendb.store import document_store<br>
from couchdb import Server<br>

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
ravendb_url = "http://localhost:8080"<br>
database_name = "Analisis"<br>
store = document_store.DocumentStore(urls=[ravendb_url], database=database_name)
store.initialize()
class Country:
    def __init__(self, series_id, country_code, country_name, year, value, name, gold_medals, silver_medals, bronze_medals):
        self.series_id = series_id
        self.country_code = country_code
        self.country_name = country_name
        self.year = year
        self.value = value
        self.name = name
        self.gold_medals = gold_medals
        self.silver_medals = silver_medals
        self.bronze_medals = bronze_medals
def csv_to_ravendb(csv_file):
    with open(csv_file, 'r', encoding='utf-8') as file:
        reader = csv.DictReader(file)
        for row in reader:
            country = Country(series_id=row['series_id'], country_code=row['country_code'],
                              country_name=row['country_name'], year=row['year'], value=row['value'],
                              name=row['name'], gold_medals=row['gold_medals'], silver_medals=row['silver_medals'],
                              bronze_medals=row['bronze_medals'])
            try:
                with store.open_session() as session:
                    session.store(country)
                    session.save_changes()
                print(f"Documento {country.__dict__} subido exitosamente a RavenDB.")
            except Exception as e:
                print(f"Error al subir el documento {country.__dict__} a RavenDB: {e}")
csv_file = 'final_train_output.csv'
csv_to_ravendb(csv_file)

## Enviar los archivos Json de Raven a Couchdb
import csv
from pyravendb.store import document_store
import couchdb
ravendb_url = "http://localhost:8080"
database_name_raven = "olympics_medals_country_wise"
raven_store = document_store.DocumentStore(urls=[ravendb_url], database=database_name_raven)
raven_store.initialize()
couchdb_url = "http://admin:admin@localhost:5984/"
database_name_couch = "olympics_medals_country_wise"
couch = couchdb.Server(couchdb_url)
if database_name_couch in couch:
    couchdb_database = couch[database_name_couch]
else:
    couchdb_database = couch.create(database_name_couch)
class Country:
    def __init__(self, countries, ioc_code, summer_participations, summer_gold, summer_silver, summer_bronze, summer_total, winter_participations, winter_gold, winter_silver, winter_bronze, winter_total, total_participation, total_gold, total_silver, total_bronze, total_total):
        self.countries = countries
        self.ioc_code = ioc_code
        self.summer_participations = summer_participations
        self.summer_gold = summer_gold
        self.summer_silver = summer_silver
        self.summer_bronze = summer_bronze
        self.summer_total = summer_total
        self.winter_participations = winter_participations
        self.winter_gold = winter_gold
        self.winter_silver = winter_silver
        self.winter_bronze = winter_bronze
        self.winter_total = winter_total
        self.total_participation = total_participation
        self.total_gold = total_gold
        self.total_silver = total_silver
        self.total_bronze = total_bronze
        self.total_total = total_total
def transfer_data_raven_to_couch():
    with raven_store.open_session() as raven_session:
        countries = list(raven_session.query(Country))
        for country in countries:
            country_data = {
                'countries': country.countries,
                'ioc_code': country.ioc_code,
                'summer_participations': country.summer_participations,
                'summer_gold': country.summer_gold,
                'summer_silver': country.summer_silver,
                'summer_bronze': country.summer_bronze,
                'summer_total': country.summer_total,
                'winter_participations': country.winter_participations,
                'winter_gold': country.winter_gold,
                'winter_silver': country.winter_silver,
                'winter_bronze': country.winter_bronze,
                'winter_total': country.winter_total,
                'total_participation': country.total_participation,
                'total_gold': country.total_gold,
                'total_silver': country.total_silver,
                'total_bronze': country.total_bronze,
                'total_total': country.total_total}
            couchdb_database.save(country_data)
if __name__ == "__main__":
    transfer_data_raven_to_couch()
    print("Transferencia de datos completada.")
