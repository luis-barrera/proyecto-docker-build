# Table of Contents

1.  [Docker con Node.js](#org8d44e2f)
    1.  [Crear API REST](#org7efbe8d)
    2.  [Dockerfile para Node.js](#org1b04045)
    3.  [docker build](#orgf5aff13)
    4.  [Correr nuestro contenedor](#orgeaad652)
    5.  [Detached mode](#orgdd83c1c)


<a id="org8d44e2f"></a>

# Docker con Node.js

A manera de practicar y recordar lo visto vamos a ver todas las formas en que podemos usar Docker para mejorar el proceso de desarrollo de una aplicación en Javascript sobre Node.js.


<a id="org7efbe8d"></a>

## Crear API REST

Vamos a crear un directorio para nuestro proyecto

    mkdir node-docker; cd node-docker

Iniciamos el proyecto de node

    npm init -y

Agregamos las dependencias necesarias

    npm install express --save

Creamos un archivo llamado `index.js`:

    var express = require("express");
    var app = express();
    
    var museos_json = require('./data/museos-cdmx.json');
    
    app.listen(3000, () => {
     console.log("Server running on port 3000");
    });
    
    app.get("/museos", (req, res, next) => {
     res.json(museos_json);
    });

Corremos nuestra aplicación

    node index.js

Probamos en el navegador: `localhost:3000/museos`


<a id="org1b04045"></a>

## Dockerfile para Node.js

Un *Dockerfile* es un archivo que contiene el conjunto de instrucciones que se deben ejecutar dentro del contenedor para replicar el ambiente en el que estamos ejecutando nuestra aplicación.

Este archivo es el va usar el comando `docker build` para crear una imagen de contenedor.

El nombre del archivo que contiene las instrucciones debe ser `Dockerfile`.

    touch Dockerfile

La primera linea que vamos a poner

    # syntax=docker/dockerfile:1

Para asegurar retrocompatibilidad con otras versiones de docker.

Luego, vamos a especificar la imagen base a usar, en este caso la imagen de node con la versión 12

    # syntax=docker/dockerfile:1
    
    FROM node:12.18.1

Dentro de este contendedor vamos a crear un directorio llamado `app` en el que vamos a poner todo el código de nuestra app

    # syntax=docker/dockerfile:1
    
    FROM node:12.18.1
    WORKDIR /app

Copiamos los archivos para instalar las dependencias de la aplicación

    # syntax=docker/dockerfile:1
    
    FROM node:12.18.1
    WORKDIR /app
    COPY ["package.json", "package-lock.json*", "./"]

Usamos el comando `npm install` para instalar las dependencias

    # syntax=docker/dockerfile:1
    
    FROM node:12
    WORKDIR /app
    COPY ["package.json", "package-lock.json*", "./"]
    RUN npm install

Copiamos el resto de archivos

    # syntax=docker/dockerfile:1
    
    FROM node:12
    WORKDIR /app
    COPY ["package.json", "package-lock.json*", "./"]
    RUN npm install
    COPY . .

Especificamos el puerto que se va a dejar abierto para que nos podamos conectar.

    # syntax=docker/dockerfile:1
    
    FROM node:12
    WORKDIR /app
    COPY ["package.json", "package-lock.json*", "./"]
    RUN npm install
    COPY . .
    EXPOSE 3000

Por último tenemos que decirle a docker el comando que se va ejecutar cuando se cree y corra (`docker run`) el contenedor.

    # syntax=docker/dockerfile:1
    
    FROM node:12
    WORKDIR /app
    COPY ["package.json", "package-lock.json*", "./"]
    RUN npm install
    COPY . .
    CMD [ "node", "server.js" ]

Aquí hay un error que vamos a cambiar más adelante.

Al dar la instrucción `COPY . .` le decimos a docker que copee todo el contenido actual. Puede haber algunos archivos que no queremos que sean considerados en estas instrucciones, para eso podemos usar un `.dockerignore`.

    node_modules

En este caso, podemos ignorar `node_modules` porque es un archivo autogenerado al correr el comando `npm install`.


<a id="orgf5aff13"></a>

## docker build

Es el comando que se encarga de correr las instrucciones declaradas en el *Dockerfile*.

    docker build --tag node-docker .

`--tag` se usa para darle un nombre a nuestra imagen.

Podemos ver las imagenes que tenemos localmente con

    docker images

Cambiemos el nombre de la imágen a una que describa mejor lo que contiene la imagen

    docker tag node-docker:latest node-docker:v1.0.0

Podemos quitar esta imagen con el comando

    docker rmi node-docker:latest


<a id="orgeaad652"></a>

## Correr nuestro contenedor

    docker run node-docker

Podemos probarlo como lo hicimos al principio.
Veremos que nos da un problema al querer conectarse.
Esto es porque no podemos conectarnos al puerto abierto en el contenedor.
Para arreglar esto usamos la opción `-p` del comando `docker run`.

    docker run -p 8080:3000 node-docker


<a id="orgdd83c1c"></a>

## Detached mode

Podemos ver que cuando corremos el contenedor, la terminal se queda ocupada corriendo el container.
Si queremos que este se siga ejecutando en segundo plano, usamos la opción `-d`

    docker run -d -p 8080:3000 node-docker

Así ya podemos ver los contenedores que tenemos creados

    docker ps -a

Podemos detenerlo con

    docker stop <container>

