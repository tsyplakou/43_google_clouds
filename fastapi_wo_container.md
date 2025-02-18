Отлично! Сегодня **развернем FastAPI в GCP Cloud Run без Docker**.  

## **📌 Что будем делать?**
1. **Создадим FastAPI-приложение** (без контейнеров).  
2. **Настроим GCP** и включим Cloud Run.  
3. **Развернем FastAPI в Cloud Run** с помощью `gcloud run deploy --source .`.  
4. **Проверим, что FastAPI работает в облаке**.  

---

## **📌 1. Подготовка: создаем FastAPI-приложение**
### **1.1 Открываем Google Cloud Shell**
Перейди в [Google Cloud Console](https://console.cloud.google.com/) и открой **Cloud Shell** (значок терминала вверху).  

### **1.2 Устанавливаем зависимости**
В Cloud Shell выполни:  
```bash
mkdir fastapi-cloudrun && cd fastapi-cloudrun
python3 -m venv venv
source venv/bin/activate  # Для Linux/macOS
pip install fastapi uvicorn
```

### **1.3 Создаем файл `main.py` с FastAPI-приложением**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI in Google Cloud Run!"}

```

### **1.4 Создаем `requirements.txt`**
```bash
echo -e "fastapi\nuvicorn" > requirements.txt
```

---

## **📌 2. Настраиваем Google Cloud**
### **2.1 Устанавливаем проект GCP**
Сначала выбери **проект GCP**, в котором будем работать:  
```bash
gcloud config set project PROJECT_ID
```
Замените `PROJECT_ID` на свой ID проекта (можно найти в [Google Cloud Console](https://console.cloud.google.com/)).

### **2.2 Включаем Cloud Run**
Активируем сервисы Cloud Run:
```bash
gcloud services enable run.googleapis.com
```

---

## **📌 3. Деплоим FastAPI в Google Cloud Run**
Теперь **развернем приложение**, не создавая Docker-образ вручную.  

**Запускаем деплой:**
```bash
gcloud run deploy fastapi-app \
    --source . \
    --region us-central1 \
    --allow-unauthenticated
```

💡 **Что делает эта команда?**  
- `gcloud run deploy fastapi-app` – создает сервис с именем `fastapi-app`.  
- `--source .` – автоматически контейнеризует код и деплоит его.  
- `--region us-central1` – выбираем регион (`us-central1` – Айова).  
- `--allow-unauthenticated` – разрешает доступ без аутентификации (чтобы можно было открыть в браузере).  

### **После завершения ты увидишь ссылку вида:**
```bash
Deploying...  
Service [fastapi-app] deployed successfully.  
URL: https://fastapi-app-xyz123.a.run.app
```
Эта ссылка – твой **развернутый FastAPI в GCP**.

---

## **📌 4. Проверяем, что FastAPI работает**
### **4.1 Открываем в браузере**
Перейди по выданному **Cloud Run URL**, например:
```
https://fastapi-app-xyz123.a.run.app
```
Ты должен увидеть JSON-ответ:
```json
{"message": "Hello from FastAPI in Google Cloud Run!"}
```

### **4.2 Проверяем через `curl`**
```bash
curl https://fastapi-app-xyz123.a.run.app
```
Ожидаемый ответ:
```json
{"message": "Hello from FastAPI in Google Cloud Run!"}
```


### **✅ 1. Создаем файл `Procfile`**
Создай файл `Procfile` в корне проекта и добавь туда команду запуска **Uvicorn**:
```plaintext
web: uvicorn main:app --host 0.0.0.0 --port $PORT
```
**Что делает этот файл?**
- **Cloud Run автоматически читает `Procfile`** и использует указанный сервер (`uvicorn`).
- **Теперь Gunicorn не будет запущен автоматически.**

---


---

## **📌 5. Если нужно удалить сервис**
Если больше не нужен, удали сервис:
```bash
gcloud run services delete fastapi-app --region=us-central1
```

---

## **📌 Итог**
✅ Мы развернули FastAPI в **Google Cloud Run без Docker**.  
✅ GCP сам **автоматически контейнеризовал код**.  
✅ Теперь приложение **работает в облаке** по публичному URL.  

---

### **Что дальше?**
1. Можем **добавить базу данных (Cloud SQL)**.  
2. Можно **развернуть FastAPI через Terraform**.  
3. В будущем разберем, как **упаковать FastAPI в Docker** и деплоить вручную.  

Попробуй **выполнить команды** и напиши, если появятся ошибки или вопросы!
