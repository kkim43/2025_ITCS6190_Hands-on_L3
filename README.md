# 2025_ITCS6190_Hands-on_L3

## Result (Screentshot.pdf)
[View Screenshot (PDF)](./screenshot.pdf)

## My Reflections
This was my first time trying deployment with Docker. 
Compared to manually installing Python and libraries with pip, I found the process much simpler and more convenient. 
Through this hands-on, I was able to understand how Docker makes setting up and running applications easier.

## Execution steps

### 1. Docker Verify Installation
``` bash
docker -v
```
### 2. PostgreSQL Setup with Docker

#### a. Pull the PostgreSQL image
``` bash
docker pull postgres
```

#### b. Create and start a PostgreSQL instance
``` bash
docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres
```

#### c. Open a terminal to the container
``` bash
docker exec -it postgres1 bash
``` 

#### d. Interact with PostgreSQL using psql
``` bash
psql -d postgres -U postgres
```

### 3. Build a Python Web App with Docker Compose in code environments 

#### a. Define application dependencies in requirements.txt
``` txt
flask
redis
```

#### b. Define the application in app.py
``` python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')

def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {}\ntimes.\n'.format(count)
```

#### c. Create a Dockerfile to containerize the application. Use the following template:
```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

#### d. Define services in a compose.yaml file
``` yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    depends_on:
      - redis
  redis:
    image: "redis:alpine"
```

#### e. Build and run the application
``` bash
docker compose up
```

#### f. Open the application in a browser at: http://localhost:8000
