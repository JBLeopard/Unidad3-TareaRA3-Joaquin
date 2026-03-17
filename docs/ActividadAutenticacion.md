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

![SQL](./images/apartado_tres/sql.png)

```sql
CREATE DATABASE jugadores;
USE jugadores;

CREATE TABLE participantes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    usuario VARCHAR(50) NOT NULL,
    clave VARCHAR(255) NOT NULL);

INSERT INTO participantes (usuario, clave)
VALUES ('snake','1980'),
       ('admin','leopard');
```

### Código vulnerable

![PHP](./images/apartado_tres/login_debil.png)

La aplicación utiliza el archivo login_debil.php para autenticar a los usuarios, este sistema presenta varias vulnerabilidades de seguridad relacionadas con la autenticación.


**`login_debil.php`**

```php
<?php
// ACTIVAR ERRORES
ini_set('display_errors', 1);
error_reporting(E_ALL);

// CONEXIÓN A BASE DE DATOS
$conn = new mysqli("database", "root", "tiger", "jugadores");

if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// PROCESAR PETICIÓN
if ($_SERVER["REQUEST_METHOD"] === "POST") {

    $username = $_POST["username"];
    $password = $_POST["password"];

    echo "<p>Usuario introducido: $username</p>";
    echo "<p>Clave introducida: $password</p>";

    // CONSULTA INSEGURA
    $query = "SELECT * FROM participantes WHERE usuario='$username' AND clave='$password'";
    echo "<p>Consulta SQL: $query</p>";

    $result = $conn->query($query);

    if ($result->num_rows > 0) {
        echo "<h2>Inicio de sesión exitoso</h2>";
    } else {
        echo "<h2>Usuario o contraseña incorrectos</h2>";
    }
}

$conn->close();
?>

<h1>Login vulnerable</h1>

<form method="post">
<input type="text" name="username" placeholder="Usuario"><br><br>
<input type="password" name="password" placeholder="Clave"><br><br>
<button type="submit">Login</button>
</form>
```

**Explicación de las vulnerabilidades**  

Este sistema tiene varios problemas:  

- **Contraseñas en texto plano**, en la base de datos se guardan así: `snake | 1980`.
- **SQL Injection**, la consulta usa variables directamente: `$query = "SELECT * FROM usuarios WHERE usuario='$username' AND clave='$password'";`, un atacante puede introducir `' OR '1'='1`.
- **Sin protección contra fuerza bruta**, el sistema permite infinitos intentos de login.

---

## 2.2 Explotación, ataque de fuerza bruta con Hydra

Cuando ejecuto Hydra desde la terminal de mi máquina atacante, mientras se está ejecutando:

- Hydra envía miles de peticiones POST.
- La web las procesa automáticamente.
- No necesita mi intervención en el formulario.













