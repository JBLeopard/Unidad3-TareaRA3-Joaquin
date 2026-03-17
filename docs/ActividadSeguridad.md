# 4. Documentación de errores en la Seguridad y componentes vulnerables

En esta actividad se analiza la presencia de **dependencias vulnerables** en un proyecto `Node.js` utilizando la herramienta **OWASP Dependency-Check**.  

Se muestra:  

- El **estado inicial del proyecto con vulnerabilidades**.
- La **detección de dependencias inseguras**.
- La **aplicación de mitigaciones**.
- Evidencias mediante capturas de pantalla.

---

## 4.1 Instalación OWASP Dependency-Check y solicitud key API NVD

A continuación muestro la instalación de la herramienta de código abierto de OWASP que permite identificar dependencias vulnerables en un proyecto al compararlas con bases de datos de vulnerabilidades conocidas, como la National Vulnerability Database (NVD) de la cual solicitaré su API key.

### Instalación OWASP Dependency-Check

![OWASP](./images/apartado_cuatro/owasp.png)

---

### Solicitud key API NVD

![API](./images/apartado_cuatro/apikey.png)

---

## 4.2 Análisis de proyecto en Node.js

### Instalación Node,js

Para verificar vulnerabilidades en las dependencias de un proyecto basado en Node.js, nstalamos `nodejs` y `npm`:

![NODEJS1](./images/apartado_cuatro/nodejs1.png)

---

### Estado inicial y vulnerabilidad

Se crea un proyecto en Node.js e instalamos librerías con versiones conocidas por contener vulnerabilidades.

![NODEJS](./images/apartado_cuatro/nodejs2.png)

---

### Análisis de la vulnerabilidad

El informe muestra múltiples dependencias con vulnerabilidades, por ejemplo:

lodash 4.17.15

Vulnerabilidad: Prototype Pollution

Severidad: Alta (CVSS ~7.4)

debug 2.6.9

Vulnerabilidades relacionadas con exposición de información.

Estas vulnerabilidades pueden permitir:

Ejecución de código no autorizado.

Manipulación de objetos en la aplicación.

Fugas de información sensible.


---


## 4.3 Mitigación

A continuación muestro las soluciones aplicadas.

### Mitigación 1 - Actualización de dependencias





### Mitigación 2 - Auditoría y verificación de seguridad
