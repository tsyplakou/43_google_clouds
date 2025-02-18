## **🚀 Добавляем базу данных (Cloud SQL) в FastAPI на Google Cloud Run**  

Теперь **добавим Cloud SQL (PostgreSQL) в наше FastAPI-приложение** на Google Cloud Run.  

---

## **📌 План действий**  
1. **Создадим Cloud SQL (PostgreSQL) в GCP**.  
2. **Настроим доступ к базе (IAM, сервисный аккаунт, соединение через Cloud SQL Proxy)**.  
3. **Добавим SQLAlchemy в FastAPI**.  
4. **Обновим код FastAPI для работы с БД**.  
5. **Перезапустим деплой в Cloud Run**.  

---

## **📌 1. Создаем Cloud SQL (PostgreSQL) в GCP**
### **1.1 Включаем API для Cloud SQL**
Запускаем в Cloud Shell:
```bash
gcloud services enable sqladmin.googleapis.com
```

### **1.2 Создаем базу данных (PostgreSQL)**
```bash
gcloud sql instances create fastapi-db \
    --database-version=POSTGRES_14 \
    --cpu=1 --memory=4GiB \
    --region=us-central1 \
    --root-password=securepassword
```
**Что делает эта команда?**
- Создаёт PostgreSQL 14  
- Выделяет **1 vCPU, 4GB RAM**  
- Название БД: `fastapi-db`  
- Регион: `us-central1`  
- Устанавливает root-пароль: `securepassword`  

### **1.3 Создаём базу и пользователя**
После создания **подключимся и создадим базу**:
```bash
gcloud sql databases create fastapi_db --instance=fastapi-db
```
Создаем пользователя:
```bash
gcloud sql users create fastapi_user --instance=fastapi-db --password=mydbpassword
```

---

## **📌 2. Настраиваем доступ Cloud Run → Cloud SQL**
### **2.1 Создаём сервисный аккаунт**
Cloud Run **не может напрямую подключаться к Cloud SQL**, нужно создать **сервисный аккаунт**.

Создадим сервисный аккаунт:
```bash
gcloud iam service-accounts create fastapi-sql \
    --display-name="FastAPI Cloud SQL Access"
```

Выдадим права **Cloud SQL Client**:
```bash
gcloud projects add-iam-policy-binding $(gcloud config get-value project) \
    --member="serviceAccount:fastapi-sql@$(gcloud config get-value project).iam.gserviceaccount.com" \
    --role="roles/cloudsql.client"
```

Теперь привяжем его к Cloud Run:
```bash
gcloud run services update fastapi-app \
    --region=us-central1 \
    --service-account=fastapi-sql@$(gcloud config get-value project).iam.gserviceaccount.com
```

---

## **📌 3. Настраиваем подключение в FastAPI**
### **3.1 Устанавливаем зависимости**
Выполним:
```bash
pip install sqlalchemy psycopg2-binary
```

### **3.2 Создаём `database.py`**
Создадим `database.py` для работы с БД:
```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import os

DB_USER = "fastapi_user"
DB_PASSWORD = "mydbpassword"
DB_NAME = "fastapi_db"
DB_INSTANCE = "nice-symbol-451311-d7:us-central1:fastapi-db"  # Замени на свой!

SQLALCHEMY_DATABASE_URL = f"postgresql+psycopg2://{DB_USER}:{DB_PASSWORD}@/{DB_NAME}?host=/cloudsql/{DB_INSTANCE}"

engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```
**Что тут важно?**
- Подключение идёт через **Unix-сокет `/cloudsql/INSTANCE_NAME`**.
- `DB_INSTANCE` = `"PROJECT_ID:REGION:INSTANCE_NAME"`.

---

## **📌 4. Добавляем таблицу в FastAPI**
### **4.1 Создаём модель данных**
Создадим `models.py`:
```python
from sqlalchemy import Column, Integer, String
from database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    email = Column(String, unique=True, index=True)
```

### **4.2 Обновляем FastAPI**
Добавим новый эндпоинт в `main.py`:
```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from database import get_db, Base, engine
from models import User
from pydantic import BaseModel, EmailStr

app = FastAPI()

# Создаём таблицы
Base.metadata.create_all(bind=engine)

@app.get("/")
def read_root():
    return {"message": "Hello from FastAPI with Cloud SQL!"}


class UserCreate(BaseModel):
    name: str
    email: EmailStr


@app.post("/users/")
def create_user(user:UserCreate, db: Session = Depends(get_db)):
    user = User(name=name, email=email)
    db.add(user)
    db.commit()
    db.refresh(user)
    return user

@app.get("/users/")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

---

## **📌 5. Перезапускаем деплой**
Теперь **развернём обновлённое FastAPI с базой**:
```bash
gcloud run deploy fastapi-app \
    --source . \
    --region=us-central1 \
    --allow-unauthenticated \
    --add-cloudsql-instances=nice-symbol-451311-d7:us-central1:fastapi-db
```

---

## **📌 6. Тестируем API**
### **6.1 Создаём пользователя**
```bash
curl -X POST "https://fastapi-app-xyz123.a.run.app/users/?name=John&email=john@example.com"
```

### **6.2 Получаем список пользователей**
```bash
curl "https://fastapi-app-xyz123.a.run.app/users/"
```

---

## **📌 Итог**
✅ **Создали Cloud SQL (PostgreSQL) и настроили сервисный аккаунт**.  
✅ **Настроили FastAPI для работы с БД через SQLAlchemy**.  
✅ **Подключили Cloud Run к Cloud SQL и развернули приложение**.  
✅ **Теперь FastAPI работает с базой!**  

Попробуй выполнить шаги и напиши, если появятся вопросы! 🚀
