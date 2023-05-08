# Simple docker tutorial for Angular and Spring boot Application

### Step 1: Create Angular Dockerfile

Write Dockerfile for Angular application

```plaintext title="Dockerfile"
FROM node:12.18.2-alpine AS builder

// Create some label for your docker image

LABEL owner="jm119"

LABEL maintainer="jm119"

LABEL description="Sample Application"


// Build your Angular application
WORKDIR /usr/src/app



COPY package.json package-lock.json ./



RUN npm install



COPY . .



RUN npm run build


// Host your angular application by nginx server on port 8080

FROM nginx:1.17-alpine


COPY nginx.config /etc/nginx/conf.d/default.conf

COPY --from=builder /usr/src/app/dist/leave-request /usr/share/nginx/html


EXPOSE 8080
```

After you build this image, when you run this image, it will host your angular application on your local machine at port 8080 (http://localhost:8080)

### Step 2: Create nginx config file

Create your nginx config file, otherwise it will use default config of nginx

```plaintext title="nginx.config" hl_lines="1"
absolute_redirect off;

server {

    listen   8080;

    root /usr/share/nginx/html;

    location / {

        try_files $uri /index.html;

    }

    location /api {

        proxy_pass http://example-api:8080/;

    }

}

```

This highlight term prevent nginx from redirect to your request to another place. Example: http://localhost:8080/directory -> http://localhost/directory/

This nginx configuration host your Angular app on port 8080

- Every request start with "/": It will redirect to our Angular main page

- Every request start with "/api": it will redirect to our Spring boot application in image name "example-api" with port 8080

### Step 3: Build Angular Dockerfile

Create your nginx config file, otherwise it will use default config of nginx

```bash
docker build -t example-app .
```

### Step 4: Create Spring Boot Dockerfile

Write Dockerfile for Spring Boot application

```plaintext title="Dockerfile"
FROM openjdk:8-jdk-alpine

// Create label for your image

LABEL owner="jm119"

LABEL maintainer="jm119"

LABEL description="Example api"

// Host this application on port 8080

EXPOSE 8080

// Copy your built jar to openjdk image and run it
COPY --chown=java:root target/example-0.0.1-SNAPSHOT.jar /apps

ENTRYPOINT ["java","-jar","/apps/example-0.0.1-SNAPSHOT.jar"]
```

### Step 5: Build Spring Boot Dockerfile

Build your Spring boot application image

```bash
docker build -t example-api .
```

### Step 6: Create docker-compose.yaml

Create docker-compose.yaml file in example-stack folder

```yaml
version: "3.8"

services:
  example-app:
    depends_on:
      - example-api
    image: example-app:latest
    ports:
      - ${EXAMPLE_APP_PORT}:8080
    restart: always
    volumes:
      - app-configuration:/usr/share/nginx/html/example/assets/config

  exmaple-api:
    depends_on:
      example-database:
        condition: service_healthy
    image: example-api:latest
    volumes:
      - api-configuration:/apps/config
    environment:
      - DATABASE_PASSWORD=${DATABASE_PASSWORD}
      - DATABASE_USERNAME=${DATABASE_USERNAME}
      - DATABASE_NAME=${DATABASE_NAME}
      - DATABASE_URL=jdbc:postgresql://${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_NAME}
    ports:
      - ${EXAMPLE_API_PORT}:8080
    restart: always

  example-database:
    image: postgres:12.2
    healthcheck:
      test:
        [
          "CMD",
          "pg_isready",
          "-q",
          "-d",
          "${DATABASE_NAME}",
          "-U",
          "${DATABASE_USERNAME}",
        ]
      timeout: 45s
      interval: 10s
      retries: 10
    environment:
      - POSTGRES_PASSWORD=${DATABASE_PASSWORD}
      - POSTGRES_USER=${DATABASE_USERNAME}
      - POSTGRES_DB=${DATABASE_NAME}
    ports:
      - ${DATABASE_HOST_PORT}:5432
    restart: always

volumes:
  api-configuration:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/conf/api
  app-configuration:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: ${PWD}/conf/app
```

### Step 7: Create .env file

We store all ports and user information in .env file

```plaintext title=".env"
# This .env is used to overwrite all
# enviroment variables used in docker-compose.yaml

# COMPOSE_PROJECT_NAME is used to instruct Docker Compose
# to use as the project (prefixed the container's names)
# rather using the current directory's name.
COMPOSE_PROJECT_NAME=example-prd

# The username & password of admin in PostgreSQL database.
DATABASE_HOST=example-database
DATABASE_PORT=5432
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=postgres
DATABASE_NAME=example

# These ports are configured in order to let the host connect
# to the docker container.
EXAMPLE_APP_PORT=8082
EXAMPLE_API_PORT=8081
DATABASE_HOST_PORT=8083
```

### Step 8: Run docker-compose.yaml

```bash
docker-compose up -d
```

### Step 9: Check out your application

Finally we reach this step, 9 such a lucky number. Let's go to your browser and check http://localhost:8082 for your Angular application and http://localhost:8081 for your Spring server.
