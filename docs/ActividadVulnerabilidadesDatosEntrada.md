# 2. Documentación de vulnerabilidades de inyección de datos de entrada

En esta documentación se analiza la vulnerabilidad **Cross-Site Scripting (XSS)** presente en una aplicación web desarrollada en PHP.  

Se muestra:  

- El **código vulnerable**.
- La **explotación de la vulnerabilidad**.
- La **implementación de mitigaciones**.
- Una **batería de pruebas** para verificar la solución.

---

## 2.1 Código vulnerable

A continuación muestro el contenido del archivo que voy a usar `comentario.php`, es una web tipo formulario para insertar texto, la cual presenta una vulnerabilidad de **Cross-Site Scripting (XSS)** debido a que los datos introducidos por el usuario se muestran en la página sin ningún tipo de validación ni sanitización.

![PHP](./images/apartado_dos/commentphp.png)

**`comentario.php`**
```php
<?php
// ACTIVAR ERRORES
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);

$comment = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
// SE GUARDA EL COMENTARIO SIN SEGURIDAD
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
        <!-- SIN htmlspecialchars: SE MUESTRA EL CONTENIDO SIN ESCAPAR -->
        <textarea name="comment" id="comment" rows="4" cols="50"><?= $comment ?></textarea><br>
        <button type="submit">Enviar</button>
    </form>

    <?php if ($comment !== ''): ?>
        <h2>Comentario recibido (SIN sanitizar)</h2>
        <!-- SE IMPRIME DE FORMA DIRECTA, VULNERABLE A XSS -->
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
<script>setTimeout(() => {window.location.href = "https://fakeupdate.net/win10ue/";}, 1000);</script>
<h2 style="color:red;">Explotación 2</h2>
```
**Explicación**

- Este ataque utiliza JavaScript para redirigir automáticamente al usuario a otra página web después de un segundo mediante: `window.location`.
- Debido a la vulnerabilidad XSS, el script se ejecuta cuando la página muestra el comentario.

**Impacto**  

Un atacante podría redirigir a los usuarios a:

- Páginas de phishing.
- Páginas con malware.
- Páginas con actualizaciones falsas del sistema.

En mi caso utilizo [fakeupdate.net](https://fakeupdate.net/win10ue/), una de sus páginas falsas que simula una actualización de Windows 10.  

---


## 2.2 Mitigación

Muestra del código modíficado con las mitigaciones aplicadas

**`commentario_ok.php`**
```php
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

### 2.2.1 Mitigación 1 - Uso de filter_input()

Código aplicado:

```php
$comment = filter_input(INPUT_POST, 'comment', FILTER_UNSAFE_RAW);
```
Explicación:

La función `filter_input()` permite obtener datos procedentes de entradas externas como formularios `POST`, en este caso se utiliza para recuperar el contenido del campo `comment`.

El filtro utilizado es `FILTER_UNSAFE_RAW`, este filtro obtiene el dato sin modificarlo para permitir aplicar posteriormente validaciones personalizadas.

---

### 2.2.2 Mitigación 2 - Validación de entrada

Código aplicado:

```php
$comment = trim($comment);

$length = mb_strlen($comment, 'UTF-8');

if ($length === 0) {
    $error = "El comentario no puede estar vacío.";
} elseif ($length > 500) {
    $error = "El comentario no puede superar los 500 caracteres.";
}
elseif (!preg_match('/^[\p{L}\p{N}\s.,!?¡¿()@#%&\-]*$/u', $comment)) {
    $error = "El comentario contiene caracteres no permitidos.";
}
```
Explicación:

Se aplican tres validaciones:

- Eliminación de espacios, `trim()` elimina espacios al inicio y al final.
- Validación de longitud, se limita el comentario a un máximo de 500 caracteres.
- Validación de caracteres, se utiliza una expresión regular para permitir únicamente letras, números, espacios y puntuación básica.

Esto evita que el usuario introduzca etiquetas HTML o código JavaScript.

---

### 2.2.3 MITIGACIÓN 3 - Sanitización con htmlspecialchars()

Código aplicado:

```php
$comment = htmlspecialchars(
    $comment,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```
Explicación:

La función `htmlspecialchars()` convierte caracteres especiales en entidades `HTML` seguras.

Ejemplo: **Entrada** `<script>` **Salida** `&lt;script&gt;`.

Esto impide que el navegador ejecute el contenido como código.

---

## 2.3 Batería de pruebas

Tras aplicar las mitigaciones se realizaron varias pruebas para comprobar el comportamiento de la aplicación.

**Prueba 1 - Intento de XSS**

Payload: `<script>alert('Vulnerabilidad XSS')</script>`

Resultado: el sistema detecta caracteres no permitidos y muestra un mensaje de error, el script no se ejecuta.  


**Prueba 2 - Redirección maliciosa**

Payload: `<script>window.location='https://fakeupdate.net/win10ue/'</script>`

Resultado: el comentario es rechazado debido a la validación de caracteres, no se produce ninguna redirección.
