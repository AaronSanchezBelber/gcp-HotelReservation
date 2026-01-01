### Caso de Negocio
## Revenue management

Si un cliente hace una reserva pero puedes predecir si la va a cancelar. Pones en la misma reserva a de habitación a dos clientes. Así te aseguras de no perder la venta

## Targeted marketing

Puedes aplicar ofertas del hotel a clientes especificos


## Fraud detection
Puedes evitar que personas que te hacen reservas que realmente no las quieren y pasa muy amenudo no las hagan baneandolos.


### Carga de datos en GPD

Como meter el csv en un cubo de la nube (data engineering), haciendo ETL con Airflow o Kafka. 
GPD - Usaremos buckets y Virtual machines para el storage

### Entorno
python 3.11
requirements.txt
setup.py - para replicar el proyecto en otros sitios
logger - para traquear o hacer checkpoints de lo que vas haciendo, para saber donde se rompe
exeption - capturar exepciones raras que rompen tu codigo para tirarlo paraalante 
source (artifacts, pipeline)
apendices (notebook, config, utils)
app (templates, static) 

1) generar el setup.py tal y como esta con los requierements
2) pip install -e . 
3) hacer el logger.py

### Data Ingestion
service account, puedes restringir el acceso a cualquier ordeandor
GCP, cli instalar
hacer rol de storage admin y storage viewer
sacar el Json file de tu service account
en consola, poner set GOOGLE_APPLICATION_CREDENTIALS=ruta sin comillas de donde este el .json
ir a la carpeta config y crear el init y el config.yaml. crear la ingesta de datos desde el bucket
crear el archivo en la carpeta confiug paths_config.py, poner todas las rutas bien

tirar en consola : python -m src.data_ingestion para comprobar que entra la data

### Pasar a la experimentacion en notebooks
Sacar todos los cambios de data en funciones para generar data_preprocessing.py. Poner la configuracion adecuada