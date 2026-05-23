## Часть I. Docker

1. Добавьте в код Dockerfile, который позволит запустить web-приложение с исходным кодом в каталоге app/ через docker.

```bash
cat > Dockerfile <<EOF
FROM python:3.9-slim

WORKDIR /app

COPY app/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

2. Выполните запуск контейнера с этим приложением.

```bash
docker build -t lab-docker .

```bash
[+] Building 0.9s (10/10) FINISHED                               docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 207B                                       0.0s
 => [internal] load metadata for docker.io/library/python:3.9-slim         0.8s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [1/5] FROM docker.io/library/python:3.9-slim@sha256:2d97f6910b16bd338  0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 204B                                          0.0s
 => CACHED [2/5] WORKDIR /app                                              0.0s
 => CACHED [3/5] COPY app/requirements.txt .                               0.0s
 => CACHED [4/5] RUN pip install --no-cache-dir -r requirements.txt        0.0s
 => CACHED [5/5] COPY app/ .                                               0.0s
 => exporting to image                                                     0.0s
 => => exporting layers                                                    0.0s
 => => writing image sha256:c8e57d0c1236823bcda72a841040e75af0696ac619482  0.0s
 => => naming to docker.io/library/lab-docker                              0.0s
 ```
 
```bash
docker run -d --name lab_docker -p 5000:5000 lab-docker
```

3. Скопируйте из консоли в каталог /home/ контейнера файл README.md.

```bash 
docker cp README.md lab_docker:/home/README.md
```

```
\Successfully copied 3.3kB (transferred 5.12kB) to lab_docker:/home/README.md~
```

4. Подключитесь к терминалу контейнера с приложением в интерактивном режиме. Проверьте, что скопированный файл находится в нужном каталоге.

```bash
docker exec -it lab_docker bash
ls /home
```

```
README.md~
```

5. Выйдите из интерактивного режима.

```bash
exit
```

6. Остановите контейнер с приложением.

```bash
docker stop lab_docker
docker rm lab_docker
```

## Часть II. Docker compose

1. Создайте файл docker-compose.yml таким образом, чтобы совместно с описанным в части 1 контейнером работала бы база данных mysql. Файл инициализации БД в каталоге db/init.sql. Также пропишите порт подключения к приложению. Например 5000.

```bash
cat > docker-compose.yml <<EOF
version: '3.8'

services:
  web:
    build: .
    container_name: python_backend

    ports:
      - "5000:5000"

    depends_on:
      db:
        condition: service_healthy

    environment:
      DB_HOST: \${DB_HOST}
      DB_USER: \${DB_USER}
      DB_PASSWORD: \${DB_PASSWORD}
      DB_NAME: \${DB_NAME}

    env_file:
      - .env

  db:
    image: mysql:8.0
    container_name: database

    restart: always

    command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

    environment:
      MYSQL_ROOT_PASSWORD: \${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: \${DB_NAME}
      MYSQL_USER: \${DB_USER}
      MYSQL_PASSWORD: \${DB_PASSWORD}

    env_file:
      - .env

    ports:
      - "3306:3306"

    volumes:
      - db_data:/var/lib/mysql
      - ./db:/docker-entrypoint-initdb.d

    healthcheck:
      test:
        [
          "CMD",
          "mysqladmin",
          "ping",
          "-h",
          "localhost",
          "-u",
          "root",
          "-p\${DB_ROOT_PASSWORD}"
        ]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db_data:
EOF
```

```bash
cat >> .gitignore <<EOF
.env
EOF
```

2. Запустите связку web-приложение - БД.

```bash
docker compose up --build
```

3. Проверьте подключение к приложению через браузер. Сделайте снимок экрана.

![Скриншот приложения](screenshot.png)
