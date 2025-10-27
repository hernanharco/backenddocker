## -- BACKEND --

## 🟡 Paso 4: Configurar el Backend (Django)
1. Crea la carpeta y entra en ella
```bash
cd ..
mkdir backend
cd backend
ejecutamos
django-admin startproject core .

```
2. Crea un requirements.txt
  ```bash
django
djangorestframework
mysqlclient
python-decouple
django-cors-headers
drf-spectacular
```
python-decouple ayuda a manejar variables de entorno. 

3. Crea el dockerfile del backend
```bash
FROM python:3.11-slim

WORKDIR /app

# Instalar dependencias del sistema (solo GCC para compilar mysqlclient)
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    default-libmysqlclient-dev \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements e instalar
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar todo el código
COPY . .

# Hacer el script ejecutable
RUN chmod +x wait-for-db.sh

# Comando final: ejecutar el script Python
CMD ["./wait-for-db.sh"]
```

# 4. Creamos wait-for-db.sh 
```bash
#!/usr/bin/env python

import os
import time
import MySQLdb
from subprocess import run, CalledProcessError

# Leer variables de entorno con valores por defecto
DB_HOST = os.getenv("DATABASE_HOST", "db")
DB_PORT = int(os.getenv("DATABASE_PORT", "3306"))
DB_USER = os.getenv("DATABASE_USER", "root")
DB_PASSWORD = os.getenv("DATABASE_PASSWORD", "")
DB_NAME = os.getenv("DATABASE_NAME", "dockersql")

print(f"⏳ Esperando conexión a MySQL...")
print(f"   Host: {DB_HOST}:{DB_PORT}")
print(f"   Usuario: {DB_USER}")
print(f"   Base de datos: {DB_NAME}")

# Intentar conexión repetidamente
while True:
    try:
        # Intentar conectar a MySQL
        MySQLdb.connect(
            host=DB_HOST,
            port=DB_PORT,
            user=DB_USER,
            passwd=DB_PASSWORD,
            db=DB_NAME,
            connect_timeout=5  # Evita esperas largas si el servicio no responde
        )
        print("✅ MySQL está disponible. Iniciando Django...")
        break
    except MySQLdb.OperationalError as e:
        print(f"🔴 MySQL no disponible aún: {e}. Reintentando en 2 segundos...")
        time.sleep(2)
    except Exception as e:
        print(f"❌ Error inesperado al conectar con MySQL: {e}")
        time.sleep(2)

# Una vez lista la DB, ejecutar el comando final
try:
    run(["python", "manage.py", "runserver", "0.0.0.0:8000"], check=True)
except CalledProcessError as e:
    print(f"❌ Fallo al iniciar Django: {e}")
    exit(1)

```


___________________________________________
