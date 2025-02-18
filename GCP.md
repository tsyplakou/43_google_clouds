## Начало, базовые команды, созданием VM

# Выведет текущий проект в GCP.
gcloud config list project

# Покажет текущего авторизованного пользователя.
gcloud auth list

# Покажет список всех VM-инстансов в GCP.
gcloud compute instances list
# SQL instances list
gcloud sql instances list

# Создает Compute Engine-инстанс в GCP.
gcloud compute instances create my-instance \
  --machine-type=e2-micro \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --zone=us-central1-a



## Работа с хранилищем GCS
# список доступных регионов
gcloud compute regions list
# Создадим GCS-бакет (имя должно быть уникальным):
gcloud storage buckets create gs://my-gcs-bucket-$(date +%s) --location=us-west1
# вывести список bucket-ов
gcloud storage buckets list
# загрузить файл в bucket (Команда копирует файл или директорию из локальной файловой системы в GCS или обратно.)
gcloud storage cp myfile.txt gs://my-gcs-bucket/
# директория
gcloud storage cp -r myfolder gs://my-gcs-bucket/
# из бакета в бакет
gcloud storage cp gs://my-gcs-bucket/myfile.txt gs://another-bucket/
# содержимое бакета
gcloud storage ls  gs://my-gcs-bucket-1232354/




# устанавливаем активный аккаунт
gcloud config set account yahor.tsyplakou@gmail.com
# список сборок
gcloud builds list


