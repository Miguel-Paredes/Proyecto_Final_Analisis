# Proyecto_Final_Analisis
## Integrantes
Cataña Dennis, Cocha Iveth, Lascano David, Paredes Miguel, Simba Cristian
## Librerias a utilizar
import pandas as pd, couchdb, json, os, csv
import pymongo
from pymongo import MongoClient
from pyravendb.store import document_store
from couchdb import Server

## Transformacion del CSV a Json
a = pd.read_csv('chat.csv')
csv_data = pd.read_csv("chat.csv", sep = ",")
csv_data.to_json("chat.json", orient = "records")

## Enviar los archivos Json a Couchdb
couch = couchdb.Server('http://admin:admin@localhost:5984')
db_name = 'chat' #los nombres de las bases de datos deben ser en minusculas y con guiones bajos para los espacios
if db_name in couch:
    db = couch[db_name]
else:
    db = couch.create(db_name)
archivo_json = 'chat 2.json'
with open(archivo_json) as archivo:
    datos_json = json.load(archivo)
for documento in datos_json:
    db.update([documento])

## Enviar los archivos Json a Mongodb
client = MongoClient('localhost', 27017)
db = client['Analisis']
collection = db['covid_worldwide']
with open('covid_worldwide.json', 'r') as file:
    data = json.load(file)
result = collection.insert_many(data)

