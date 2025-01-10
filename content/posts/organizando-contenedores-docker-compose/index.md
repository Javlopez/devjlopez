---
title: "Organizando contenedores - docker compose"
date: 2025-01-09T15:55:25-06:00
draft: false
tags: ["compose", "docker", "automate"]
image: 'pexels-anilsharma65-28503131.jpg'
description: ""
---
Cuando trabajamos con contenedores muchas veces se vuelve complejo manejar manualmente uno por uno, ya que 
en un entorno de trabajo usualmente se requieren, además de la app, varios servicios como base de datos, un servicio de queues, redis, casandra, etc.

Existen diversas formas de manejar esta situation, pero vamos a concentrar en las más usuales, primero veremos como lograr esto manualmente y después nos iremos a docker compose

Usar docker o algún software para manejo de contenedores puede verse tan simple como lo siguiente:

```bash
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
$
```

Esto funciona bastante bien, es simple y efectivo, pero cuando trabajas en cualquier proyecto es obvio que no usaras un solo contenedor, imaginemos la siguiente situación, 
necesitas poner en contenedores los siguientes servicios:  

- Tu app (Asumamos que necesitas PHP, pero puede ser cualquier otro lenguaje)
- Nginx: Tu servidor
- PostgreSQL: Una base de datos relacional
- Redis: Redis para cache

**Entonces basado en dichos requerimientos necesitaríamos construir los siguientes contenedores:**
```bash
# run postgress
$ docker run -d \
  --name postgres \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=pass \
  -e POSTGRES_DB=app_db \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15

# run redis
$ docker run -d \
  --name redis \
  --network my_network \
  -v redis_data:/data \
  redis:7

# run your app
$ docker run -d \
  --name app \
  --network my_network \
  -v $(pwd)/app/src:/var/www/html \
  -v $(pwd)/app/nginx.conf:/etc/nginx/nginx.conf \
  -v $(pwd)/app/php.ini:/usr/local/etc/php/php.ini \
  -p 8080:80 \
  app
```
Con tanto código, hay muchas cosas que pueden fallar al usar los comandos anteriores, y ni hablar de depurar, cambiar algo o añadir más servicios. 

Entonces, ¿cómo lo solucionamos? Hay varias maneras, y empezaremos con la más sencilla.

### Docker compose 

> Docker Compose es una herramienta para definir y ejecutar aplicaciones multicontenedor. 
Compose simplifica el control de toda la pila de aplicaciones, lo que facilita la administración de servicios, redes y volúmenes en un único archivo de configuración YAML comprensible. Luego, con un solo comando, crea e inicia todos los servicios desde su archivo de configuración.
 https://docs.docker.com/compose/

Básicamente puedes organizar todos tus contenedores mediante un solo archivo YAML lo cual te permitirá simplificar tu rutina diaria.

- Iniciar, detener y reconstruir servicios
- Ver el estado de los servicios en ejecución
- Transmitir la salida del registro de los servicios en ejecución
- Ejecutar un comando único en un servicio

Y como hacemos esto? bueno veamos como se veria esto mismo pero desde docker compose

```yaml

services:
  app:
    image: app
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: app
    networks:
      - my_network
    ports:
      - "8080:80"
    volumes:
      - ./app/src:/var/www/html
      - ./app/nginx.conf:/etc/nginx/nginx.conf
      - ./app/php.ini:/usr/local/etc/php/php.ini

  postgres:
    image: postgres:15
    container_name: postgres
    networks:
      - my_network
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app_db
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: redis
    networks:
      - my_network
    volumes:
      - redis_data:/data

  localstack:
    image: localstack/localstack
    container_name: localstack
    networks:
      - my_network
    environment:
      SERVICES: s3,ec2,lambda
      LOCALSTACK_API_KEY: your_api_key_here
    ports:
      - "4566:4566"

networks:
  my_network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

A pesar de que el código parece correcto y se identifican patrones a simple vista, un análisis detallado de cada parte del archivo es fundamental:

#### Service: app
```yaml
app:
  image: app
  build:
    context: ./app
    dockerfile: Dockerfile
  container_name: app
  networks:
    - my_network
  ports:
    - "8080:80"
  volumes:
    - ./app/src:/var/www/html
    - ./app/nginx.conf:/etc/nginx/nginx.conf
    - ./app/php.ini:/usr/local/etc/php/php.ini
```
¿Qué hace en cada parte? Bien veamos a detalle

- image: app: Utiliza una imagen personalizada llamada app (construida a partir del Dockerfile).
- build: Construye la imagen personalizada desde el directorio ./app y el archivo Dockerfile.
- container_name: app: Asigna el nombre app al contenedor.
- networks: Conecta el contenedor a la red my_network.
- ports: Mapea el puerto 80 dentro del contenedor al puerto 8080 en el host.
- volumes: Monta volúmenes para:
  - Código fuente (src) en /var/www/html.
  - Configuración personalizada de Nginx.
  - Configuración personalizada de PHP.

#### Service: postgres
```yaml
postgres:
 image: postgres:15
 container_name: postgres
 networks:
  - my_network
 environment:
  POSTGRES_USER: user
  POSTGRES_PASSWORD: pass
  POSTGRES_DB: app_db
 volumes:
  - postgres_data:/var/lib/postgresql/data
```

- image: postgres:15: Utiliza la imagen oficial de PostgreSQL versión 15.
- container_name: postgres: Asigna el nombre postgres al contenedor.
- networks: Conecta el contenedor a la red my_network.
- environment: Define variables de entorno:
  - El usuario (user).
  - La contraseña (pass).
  - La base de datos inicial (app_db).
- volumes: Monta un volumen persistente (postgres_data) para almacenar datos de PostgreSQL.

#### Service: redis

```yaml
redis:
  image: redis:7
  container_name: redis
  networks:
    - my_network
  volumes:
    - redis_data:/data
```

- image: redis:7: Utiliza la imagen oficial de Redis versión 7.
- container_name: redis: Asigna el nombre redis al contenedor.
- networks: Conecta el contenedor a la red my_network.
- volumes: Monta un volumen persistente (redis_data) para almacenar los datos de Redis.

Todo en un solo archivo ahora bien como ejecuto esto, eso es lo mejor con un sencillo comando podemos lanzar todos nuestros contenedores

```bash
$ docker compose -f docker-compose.yaml up
[+] Running 0/0
[+] Running 1/1
[+] Running 0/1 Building                                                                                                                                                                 0.1s 
[+] Building 13.2s (14/15)                                                                                                                                                     docker:default 
 => [app internal] load build definition from Dockerfile                                                                                                                                 0.0s
 => => transferring dockerfile: 1.26kB                                                                                                                                                   0.0s
 => [app internal] load metadata for docker.io/library/php:8.3-apache                                                                                                                    0.4s 
 => [app internal] load .dockerignore                                                                                                                                                    0.0s
 => => transferring context: 2B                                                                                                                                                          0.0s
 => [app  1/11] FROM docker.io/library/php:8.3-apache@sha256:fce243539486d99cfefba35724ec485fd6078f1d4928feba5728d3ca587f8820                                                            0.0s
 => [app internal] load build context                                                                                                                                                    0.0s
 => => transferring context: 1.32kB 
 ....                             
```

En esencia, docker compose hace lo siguiente:

 - Define aplicaciones multi-contenedor: Permite especificar los servicios (contenedores), las redes, los volúmenes y otras configuraciones necesarias para una aplicación compleja en un archivo YAML (normalmente llamado docker-compose.yml).
 - Orquesta la ejecución: Con un solo comando (`docker compose up`), se crean y se inician todos los contenedores definidos en el archivo de configuración, junto con sus dependencias.
 - Simplifica la gestión del ciclo de vida: Facilita la detención (`docker compose down`), el reinicio (`docker compose restart`), la escalabilidad (`docker compose up --scale <servicio>=<cantidad>`, por ejemplo, `docker compose up --scale web=3`), la visualización de logs (`docker compose logs`), la ejecución de comandos dentro de un contenedor en ejecución (`docker compose exec <servicio> <comando>`, por ejemplo `docker compose exec web bash`), entre otras operaciones, de toda la aplicación multi-contenedor.
 - Reconstrucción de imágenes: Puedes reconstruir todas las imágenes definidas en tu archivo docker-compose.yml con el comando `docker compose build`. Esto es útil cuando has modificado los Dockerfiles o necesitas actualizar las dependencias de tus imágenes.
 - Ejecución en modo "detached": Para ejecutar los contenedores en segundo plano y liberar la terminal, se utiliza el comando `docker compose up -d`. Esto permite que los contenedores sigan funcionando incluso después de cerrar la terminal. Para detener los contenedores que se ejecutan en modo detached, se usa docker compose down.

En resumen: `docker compose` simplifica enormemente el desarrollo y la gestión de aplicaciones multi-contenedor, ahorrando tiempo y esfuerzo en la configuración y orquestación manual de los contenedores. Es una herramienta fundamental para cualquier desarrollador que trabaje con Docker en entornos de desarrollo, pruebas o incluso producción