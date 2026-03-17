# 2. Documentación de autenticación, vulnerabilidades de gestión de sesiones, protección de datos sensibles o Control de acceso

En esta sección se analiza una vulnerabilidad de autenticación débil **(Broken Authentication)** presente en una aplicación web desarrollada en PHP.  

Se muestra:  

- El **código vulnerable del sistema de autenticación**.
- La **explotación mediante ataques de fuerza bruta**.
- La **implementación de mitigaciones de seguridad**.
- Una **batería de pruebas** para verificar las soluciones aplicadas.

---

## 2.1 Base de datos y código vulnerable

### Creación base de datos

Primero creo la base de datos `jugadores` con la tabla `participantes` e inserto datos en las filas de `usuario` y `clave`.

```sql
CREATE DATABASE jugadores;
USE jugadores;

CREATE TABLE participantes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    usuario VARCHAR(50) NOT NULL,
    clave VARCHAR(255) NOT NULL);

INSERT INTO participantes (usuario, clave)
VALUES ('snake','1980'),
       ('kratos','password');
´´´

