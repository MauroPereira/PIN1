# CI/CD Pipeline para la Aplicación Pin1

Este documento describe el pipeline de CI/CD configurado para la aplicación **Pin1**, utilizando Jenkins y Docker. El pipeline realiza las siguientes etapas:

## Tabla de Contenidos

- [CI/CD Pipeline para la Aplicación Pin1](#cicd-pipeline-para-la-aplicación-pin1)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Descripción General](#descripción-general)
  - [Variables de Entorno](#variables-de-entorno)
  - [Etapas del Pipeline](#etapas-del-pipeline)
    - [Construcción de la Imagen](#construcción-de-la-imagen)
      - [Ejecutar Pruebas](#ejecutar-pruebas)
      - [Etiquetar la Imagen](#etiquetar-la-imagen)
      - [Enviar la Imagen al Registro](#enviar-la-imagen-al-registro)
      - [Despliegue](#despliegue)
        - [Despliegue en Dev](#despliegue-en-dev)
        - [Despliegue en Prod](#despliegue-en-prod)

## Descripción General

El pipeline automatiza la construcción, prueba, etiquetado, envío y despliegue de la imagen de Docker de la aplicación Pin1. Se ejecuta en diferentes entornos según la rama de Git (`dev` para desarrollo y `stable` para producción).

## Variables de Entorno

Las siguientes variables de entorno son utilizadas en el pipeline:

- `DOCKER_REGISTRY_SERVER`: Dirección del servidor del registro de Docker (ej. `192.168.100.20:5000`).
- `APP_NAME`: Nombre de la aplicación (ej. `pin1`).
- `DEV_SERVER`: Dirección del servidor de desarrollo (ej. `192.168.100.21`).
- `DEV_USER`: Usuario para el servidor de desarrollo (ej. `root`).
- `PROD_SERVER`: Dirección del servidor de producción (ej. `192.168.100.22`).
- `PROD_USER`: Usuario para el servidor de producción (ej. `root`).

## Etapas del Pipeline

### Construcción de la Imagen

En esta etapa, se construye la imagen de Docker utilizando el Dockerfile presente en el directorio de trabajo.

```
sh "sudo docker build -t ${env.APP_NAME} ."
```

#### Ejecutar Pruebas

Se ejecutan las pruebas definidas en el proyecto utilizando npm dentro del contenedor.

```
sh "sudo docker run ${env.APP_NAME} npm test"
```

#### Etiquetar la Imagen

La imagen construida se etiqueta con el número de construcción y también con la etiqueta latest.

``` 
sh """
  sudo docker tag ${env.APP_NAME} ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} && \
  sudo docker tag ${env.APP_NAME} ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
"""
```
####  Enviar la Imagen al Registro

Las imágenes etiquetadas se envían al registro de Docker.

```
sh """
  sudo docker push ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
  sudo docker push ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
"""
```

#### Despliegue

##### Despliegue en Dev
Si la rama actual es dev, se ejecuta el despliegue en el servidor de desarrollo.

```
ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${DEV_USER}@${DEV_SERVER} bash -c "
  docker pull ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} &&
  docker stop ${env.APP_NAME} || true &&
  docker rm ${env.APP_NAME} || true &&
  docker run -d --name ${env.APP_NAME} -p 3000:3000 ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}
"
```
##### Despliegue en Prod
Si la rama actual es stable, se ejecuta el despliegue en el servidor de producción.

```
ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${PROD_USER}@${PROD_SERVER} bash -c "
  docker pull ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest &&
  docker stop ${env.APP_NAME} || true &&
  docker rm ${env.APP_NAME} || true &&
  docker run -d --name ${env.APP_NAME} -p 3000:3000 ${env.DOCKER_REGISTRY_SERVER}/${env.APP_NAME}:latest
"
```