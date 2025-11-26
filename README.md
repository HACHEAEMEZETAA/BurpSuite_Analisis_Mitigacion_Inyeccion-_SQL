# 🏆 Descubrimiento de BurpSuite: Análisis y Mitigación de la Inyección SQL (SQLi)

Esta actividad documenta el proceso completo de **identificación, explotación y mitigación** de una vulnerabilidad de **Inyección SQL (SQLi)** en un entorno de laboratorio controlado con Docker. La herramienta principal utilizada para la fase de explotación y análisis de peticiones fue **BurpSuite Community Edition**.

## 🚀 Arquitectura y Entorno de Pruebas

El laboratorio se desplegó utilizando **Docker Compose** para simular una arquitectura web vulnerable y aislada.

| Componente | Imagen Base | Puerto Local | Función |
| :--- | :--- | :--- | :--- |
| **`web`** | `php:8.1-apache` (Construida) | `8001` | Aplicación web vulnerable con formulario de *login*. |
| **`db`** | `mysql:5.7` | N/A | Base de datos que contiene la tabla `users` y las credenciales de prueba. |
| **Herramienta** | BurpSuite Community | `8080` (Proxy) | Intercepción, manipulación de tráfico y explotación. |

### 🛠️ Corrección de Compatibilidad (Dockerfile)

Se creó un `Dockerfile` para resolver el error de función obsoleta (`mysql_connect`) forzando la instalación de la extensión **`mysqli`** (`RUN docker-php-ext-install mysqli`) en el contenedor web.

---

## 💥 Explotación: Bypass de Autenticación (SQLi)

La fase de explotación se llevó a cabo utilizando **BurpSuite Repeater** para manipular la petición de *login*.

### Payload de Ataque Utilizado

El ataque tuvo como objetivo alterar la cláusula `WHERE` de la consulta de autenticación para forzar un resultado siempre verdadero.

| Método | Payload | Resultado en la Consulta |
| :--- | :--- | :--- |
| **Bypass con Comentario (Final)** | `' OR 1=1 #` | El ataque exitoso forzó la lógica a `... WHERE (TRUE) [Comentario #]`, anulando la verificación de la contraseña. |

### Evidencia de la Explotación

1.  **Ejecución en Repeater:** Al enviar el *payload* **`' OR 1=1 #`** (URL-encoded: `%27+OR+1%3D1+%23`), el servidor respondió con un código **`200 OK`**.
2.  **Confirmación:** La respuesta incluyó el mensaje **`Bienvenido, ' OR 1=1 #`**, confirmando el **acceso no autorizado** al sistema.

> 

---

## 🛡️ Mitigación de la Vulnerabilidad

La solución se implementó modificando el código PHP vulnerable para eliminar la concatenación de cadenas, abordando así la causa raíz de la SQLi.

### Solución Implementada: Sentencias Preparadas

Se reemplazó el código inseguro por la implementación de **Sentencias Preparadas** (*Prepared Statements*) utilizando la extensión **MySQLi**. 

```php
// Código Seguro
$stmt = $conn->prepare("SELECT * FROM users WHERE username = ? AND password = ?"); 
$stmt->bind_param("ss", $user_input, $pass_input); // Los datos son tratados como strings.
$stmt->execute();
