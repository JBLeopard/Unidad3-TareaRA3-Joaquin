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




### Solicitud key API NVD



## 4.2 Análisis de proyecto en Node.js

### Instalación Node,js

Para verificar vulnerabilidades en las dependencias de un proyecto basado en Node.js, nstalamos `nodejs` y `npm`:



### Estado inicial y vulnerabilidad

Se crea un proyecto en Node.js e instalamos librerías con versiones conocidas por contener vulnerabilidades.

