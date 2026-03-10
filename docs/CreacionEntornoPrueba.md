# 1. Documentación sobre la creación del entorno de Pruebas

En esta documentación se realiza la **creación del entorno de Pruebas**, que consiste en crear un entorno de pruebas, un servidor LAMP, en el que posteriormente vamos a realizar posteriormente diferentes actividades en las que introduciremos archivos con vulnerabiliades, para ver cómo podemos corregirlas posteriormente.  

-  Creo un escenario multicontenedor con LAMP. la encontramos en [https://github.com/sprintcube/docker-compose-lamp.git](https://github.com/sprintcube/docker-compose-lamp.git).


![LAMP 1](./images/apartado_uno/lamp1.png)
**Vulnerabilidad analizada**

- **Producto afectado:** GoAnywhere MFT  
- **Fabricante:** Fortra  
- **Tipo:** Omisión de autenticación  
- **Gravedad:** Crítica

La vulnerabilidad permite a un atacante no autenticado acceder a funcionalidades internas de la aplicación.

---

## 1.1 Instalación

Para la instalación de LAMP seguimos las instrucciones del repositorio.

![LAMP 2](./images/apartado_uno/lamp2.png)

-  Muestra de mi fichero .emv
`sample.env`
```
# Please Note:
# In PHP Versions <= 7.4 MySQL8 is not supported due to lacking pdo support

# To determine the name of your containers
COMPOSE_PROJECT_NAME=lamp

# Possible values: php54, php56, php71, php72, php73, php74, php8, php81, php82, php83
PHPVERSION=php83
DOCUMENT_ROOT=./www
APACHE_DOCUMENT_ROOT=/var/www/html
VHOSTS_DIR=./config/vhosts
APACHE_LOG_DIR=./logs/apache2
PHP_INI=./config/php/php.ini
SSL_DIR=./config/ssl

# PHPMyAdmin
UPLOAD_LIMIT=512M
MEMORY_LIMIT=512M

# Xdebug
XDEBUG_LOG_DIR=./logs/xdebug
XDEBUG_PORT=9003
#XDEBUG_PORT=9000

# Possible values: mysql57, mysql8, mariadb103, mariadb104, mariadb105, mariadb106
#
# For Apple Silicon User: 
# Please select Mariadb as Database. Oracle doesn't build their SQL Containers for the arm Architecure

DATABASE=mysql8
MYSQL_INITDB_DIR=./config/initdb
MYSQL_DATA_DIR=./data/mysql
MYSQL_LOG_DIR=./logs/mysql

# If you already have the port 80 in use, you can change it (for example if you have Apache)
HOST_MACHINE_UNSECURE_HOST_PORT=80

# If you already have the port 443 in use, you can change it (for example if you have Apache)
HOST_MACHINE_SECURE_HOST_PORT=443

# If you already have the port 3306 in use, you can change it (for example if you have MySQL)
HOST_MACHINE_MYSQL_PORT=3306

# If you already have the port 8080 in use, you can change it (for example if you have PMA)
HOST_MACHINE_PMA_PORT=8080
HOST_MACHINE_PMA_SECURE_PORT=8443

# If you already has the port 6379 in use, you can change it (for example if you have Redis)
HOST_MACHINE_REDIS_PORT=6379

# MySQL root user password
MYSQL_ROOT_PASSWORD=tiger

# Database settings: Username, password and database name
#
# If you need to give the docker user access to more databases than the "docker" db 
# you can grant the privileges with phpmyadmin to the user.
MYSQL_USER=docker
MYSQL_PASSWORD=docker
MYSQL_DATABASE=docker
```
![LAMP 3](./images/apartado_uno/lamp3.png)
-  Muestra de mi fichero docker ![LAMP 2](./images/apartado_uno/docker.png)

![INCIBE 2](./imagenes/apartado_uno/incibe2.png)

---

## 1.2 Web oficial del fabricante (Fortra)

Desde el artículo de INCIBE se accede a la web oficial del fabricante, donde se publica el aviso de seguridad correspondiente.

En esta página se detallan:

- La descripción técnica de la vulnerabilidad  
- Las versiones afectadas  
- Las medidas de mitigación y parches disponibles  

![FORTRA 1](./imagenes/apartado_uno/fortra1.png)
![FORTRA 2](./imagenes/apartado_uno/fortra2.png)

---

## 1.3 Identificación del CVE y análisis

En la información proporcionada por el fabricante se identifica el identificador CVE asignado a la vulnerabilidad.  
Este identificador permite realizar el seguimiento oficial del fallo de seguridad en las diferentes bases de datos.  
Accedemos a la página oficial de **cve.org**, donde se muestra la información básica del CVE:

- Descripción de la vulnerabilidad  
- Referencias oficiales  
- Fecha de publicación  

![CVE 1](./imagenes/apartado_uno/cve1.png)
![CVE 2](./imagenes/apartado_uno/cve2.png)

---

## 1.4 Consulta en la NVD (NIST)

A continuación se consulta la **National Vulnerability Database (NVD)** mantenida por el NIST, donde se amplía la información técnica.

![NVD 1](./imagenes/apartado_uno/nvd1.png)

En la NVD se muestra la puntuación **CVSS**, que indica el nivel de riesgo de la vulnerabilidad.

- **Puntuación CVSS:** 9.8 Crítica  
- **Vector CVSS:** Permite analizar los factores utilizados para calcular la puntuación  

![NVD 2](./imagenes/apartado_uno/nvd2.png)

También podemos ver las versiones de softwate a las que afecta:  

![NVD 3](./imagenes/apartado_uno/nvd3.png)
![NVD 4](./imagenes/apartado_uno/nvd4.png)

---

## 1.5 Análisis de la CWE

Desde la NVD se identifican las **debilidades (CWE)** explotadas por la vulnerabilidad.  
Accedemos a la página oficial de **cwe.mitre.org** para analizar en detalle la debilidad principal.  
En esta sección se describe:  

- En qué consiste la debilidad  
- Lenguajes y tecnologías afectadas  
- Posibles mitigaciones  

![CWE 1](./imagenes/apartado_uno/cwe1.png)

Se observa la relación de la debilidad con otras CWE, mostrando jerarquías de debilidades padre e hijas.

![CWE 2](./imagenes/apartado_uno/cwe2.png)

También una lista de CVE relacionadas.

![CWE 3](./imagenes/apartado_uno/cwe3.png)

Patrones de ataque (CAPEC)  
Desde la información de la CWE se accede a los **patrones de ataque (CAPEC)** que pueden explotar esta debilidad.  

![CWE 4](./imagenes/apartado_uno/cwe4.png)

---

## 1.6 Análisis del patrón de ataque CAPEC

En la web oficial de **capec.mitre.org** se analiza el patrón de ataque identificado.  
Se detallan:  

- Descripción del ataque  
- Flujo de ejecución  
- Requisitos previos  
- Consecuencias  
- Posibles mitigaciones  

![CAPEC](./imagenes/apartado_uno/capec.png)

---

## 1.7 Descarga del registro CVE en formato JSON

Finalmente, desde **cve.org** se accede al **registro CVE en formato JSON**, utilizado para el tratamiento automatizado de la información de vulnerabilidades.  
Este registro contiene información estructurada sobre:  

- CVE  
- CWE  
- CPE  
- CAPEC  
- Referencias oficiales

[CVE-2024-0204.json](https://github.com/JBLeopard/Unidad2-TareaRA2-Joaquin/blob/main/docs/CVE-2024-0204.json)

---
