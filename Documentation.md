# Path of the project 

``on aspaelp00000084.pfizer.com server : /data2/corpus/Github/fix/EM-Cognitivesearch/data_preprocessing/pipeline-processing ``

# important files and folders 
1. ``worker`` contain celery code for excuting the modules (partail upload , partial extract,..etc) in the background
2. ``template`` contain UI designs (HTML Files)
3. ``main.py`` the main file to start 
4. ``requirement.txt`` requirement file to install packages 
5. ``config.json`` confgiuration file contain index name and file pathes 
6. ``command_script.sh`` bash script to start all component in the background

# component of the project 
1. fastapi
2. celery 
3. flower

#  Setup before build 
1. create virtual enivroment
2. ``pip install -r requirement.txt``

# start project 
## first way start earch component alone
1. ``uvicorn --host 0.0.0.0 --port 8501 main:app --reload`` to start fastapi 
2. ``celery worker -A worker.celery_app -l info -f celery.logs`` to start celery with log file (celery.logs)
3. ``celery -A  worker -A worker.celery_app flower --host=aspaelp00000084 --port=8555 -f celery.logs -l info`` to start flower

## second way to start from bash script
1. ``sh command_script.sh`` 

# project modules and its parameter
1. partial upload  -> to partial upload docs to elasticsearch
parameter : (pipe configuration , index name , filepath)
2. partial extaction -> to extract data from elasticsearch 
parameter : (file path , index name , number of record to extract(if blank will be full extraction))
3. partial extraction advance -> for extract data more than 10k recorde 
parameter (file path , index name , time to keep data in elastic search )
4. Full ingestion -> to fully ingest data to elastic search
parameter: (options , filepath, number of record to be ingested , index name)
5. incremental ingestion - > ingest data according to time interval 
parameter (options , filepath , interval day , index name)
