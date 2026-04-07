# URL-based access control can be circumvented

### Objetivo <a href="#objetivo" id="objetivo"></a>

El objetivo de este laboratorio es acceder al panel de administración sin pasar por el control del front-end y eliminar al usuario `carlos`.

### Contexto técnico <a href="#contexto-tcnico" id="contexto-tcnico"></a>

La aplicación expone un panel de administración en `/admin`, pero el front-end bloquea el acceso directo a esa ruta. Sin embargo, el back-end interpreta el encabezado `X-Original-URL`, lo que permite sobrescribir la ruta real procesada por el servidor.

### Hallazgo <a href="#hallazgo" id="hallazgo"></a>

Si enviamos una petición a una ruta permitida, pero añadimos `X-Original-URL: /admin`, el back-end nos entrega el contenido del panel administrativo.

### Paso a paso <a href="#paso-a-paso" id="paso-a-paso"></a>

### 1. Interceptar la petición <a href="#id-1-interceptar-la-peticin" id="id-1-interceptar-la-peticin"></a>

Capturamos una petición en Burp Suite y la reenviamos a **Repeater**.

### 2. Acceder al panel admin <a href="#id-2-acceder-al-panel-admin" id="id-2-acceder-al-panel-admin"></a>

Modificamos la request para que apunte a una URL permitida y añadimos el encabezado:

```
textGET /login HTTP/2
Host: <host>
X-Original-URL: /admin
```

Con esto, el servidor devuelve el panel de administración aunque la URL visible sea `/login`.

### 3. Identificar la acción de borrado <a href="#id-3-identificar-la-accin-de-borrado" id="id-3-identificar-la-accin-de-borrado"></a>

Dentro del HTML del panel, localizamos el enlace para eliminar usuarios. En este caso, la acción para `carlos` apunta a:

```
text/admin/delete?username=carlos
```

### 4. Eliminar a `carlos` <a href="#undefined" id="undefined"></a>

Volvemos a modificar la request y cambiamos el encabezado para invocar la ruta de borrado:

```
textGET /login?username=carlos HTTP/2
Host: <host>
X-Original-URL: /admin/delete
```

Al enviar la petición, el usuario `carlos` queda eliminado y el laboratorio se completa.

### Resumen <a href="#resumen" id="resumen"></a>

La vulnerabilidad consiste en un control de acceso basado en URL que solo aplica en el front-end. Al abusar de `X-Original-URL`, podemos reescribir la ruta procesada por el back-end y acceder a funciones administrativas.
