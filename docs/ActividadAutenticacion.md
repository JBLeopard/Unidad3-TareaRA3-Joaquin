# 2. Documentación de autenticación, vulnerabilidades de gestión de sesiones, protección de datos sensibles o Control de acceso

En esta sección se analiza una vulnerabilidad de autenticación débil **(Broken Authentication)** presente en una aplicación web desarrollada en PHP.  

Se muestra:  

- El **código vulnerable del sistema de autenticación**.
- La **explotación mediante ataques de fuerza bruta**.
- La **implementación de mitigaciones de seguridad**.
- Una **batería de pruebas** para verificar las soluciones aplicadas.

---

## 2.1 Base de datos y código vulnerable

A continuación muestro el contenido del archivo que voy a usar `comentario.php`, es una web tipo formulario para insertar texto, la cual presenta una vulnerabilidad de **Cross-Site Scripting (XSS)** debido a que los datos introducidos por el usuario se muestran en la página sin ningún tipo de validación ni sanitización.
