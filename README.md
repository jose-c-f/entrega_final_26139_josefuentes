# API REST (Spring Boot)

Este proyecto consiste en un backend implementado como API REST con Spring Boot y un frontend.

La API REST de Spring Boot está deployada en [Render](https://render.com), el frontend en [Vercel](https://vercel.com), y la base de datos está hosteada en [TiDB Cloud](https://tidbcloud.com).

---

## 🔗 Demo y enlaces

| Recurso | Enlace |
|---------|--------|
| 🖥️ **Frontend** (React + Vite, en Vercel) | https://jose-c-f-articulos-frontend.vercel.app/ |
| ⚙️ **API REST** (Spring Boot, en Render) | https://entrega-final-26139-josefuentes.onrender.com/api/articulos |
| 🎬 **Video demo** | [Ver abajo ⬇️](#video-demo) |

> ⏳ El backend usa el plan gratuito de Render: tras ~15 min de inactividad se duerme y la
> primera petición puede tardar ~30–60 s en responder.

### Frontend en funcionamiento

[![Frontend de la aplicación](docs/img/frontend.png)](https://jose-c-f-articulos-frontend.vercel.app/)

<!--
  La imagen de arriba es clickeable. Apunta al frontend de Vercel.
  Si preferís que abra el VIDEO, reemplazá la URL entre paréntesis por el link del video.
  Pegá la captura en: docs/img/frontend.png
-->

### Video demo

https://github.com/user-attachments/assets/ff5ec077-9d49-4026-be17-3a8b1ff95990

---

## 1. Introducción

**Render** hace el build y ejecuta la aplicación Spring Boot desde un contenedor Docker, ésta se conecta a la base de datos en **TiDB Cloud**.  El frontend en **Vercel** se comunica con la aplicación Spring Boot.

### 1.1 Repositorio
**Render** lee el código de la aplicación Spring Boot de **GitHub**.
Por otra parte, **Vercel** también lee el código del frontend de otro repositorio de **GitHub**

### 1.2 Credenciales
Render requiere que las credenciales estén en variables
de entorno, esto se configura desde la consola de Render.

### 1.3 pom.xml
El build se hace con Maven, con el archivo pom.xml, el resultado del build es un jar con la aplicación Spring Boot.

### 1.4 Dockerfile
El archivo Dockerfile contiene las instrucciones para el deploy y arranque de la aplicación en el servidor (que en este caso es Render).

### 1.5 Base de datos
La base de datos en está en la nube y es TiDB, en Render se configuran las variables de entorno para la conexión.


---



## 2. Deploy

### 1 — Base de datos

1. Entrá en https://tidbcloud.com y creá un cluster **Serverless** (tiene capa
   gratuita).
2. En el cluster, hacé click en el botón **Connect** (arriba a la derecha) y elegí
   **Connect With > .env**. TiDB te muestra los valores listos para copiar:
   ```env
   DB_HOST=gateway01.<region>.prod.aws.tidbcloud.com
   DB_PORT=4000
   DB_USERNAME='<prefijo>.root'
   DB_PASSWORD='<tu-password>'
   DB_DATABASE='articulos_db'
   ```
   Anotalos: los vas a cargar tal cual en Render (Paso 4). Las comillas que muestra TiDB
   son sintaxis del archivo `.env`; en el panel de Render se pegan **sin** comillas.
3. Creá la base de datos `articulos_db`. Desde la consola SQL de TiDB (SQL Editor):
   ```sql
   CREATE DATABASE articulos_db;
   ```
   (La tabla `articulo` la creará Hibernate sola al arrancar la app.)

### 2 — GitHub

Render despliega desde GitHub/GitLab. Subí el proyecto `articulos-api` a un
repositorio público (debe incluir `Dockerfile`, `pom.xml` y `src/`).

### 3 — Render

1. En Render, andá a **New > Web Service** y conectá el repositorio.
2. En **Runtime** elegí **Docker** (Render usará el `Dockerfile`).
3. En **Health Check Path** indicá `/api/articulos`.

### 4 — Configurar las variables de entorno en Render

En la sección **Environment** del servicio, definí:

| Variable      | Valor |
|---------------|-------|
| `DB_HOST`     | el host de TiDB (ej. `gateway01.us-east-1.prod.aws.tidbcloud.com`) |
| `DB_PORT`     | `4000` |
| `DB_DATABASE` | `articulos_db` |
| `DB_USERNAME` | el usuario de TiDB (ej. `xxxxxxxx.root`) |
| `DB_PASSWORD` | la contraseña de TiDB |

Son los mismos valores del `.env` que copiaste en el Paso 1 (cambiando `DB_DATABASE` a
`articulos_db`). La app arma la URL JDBC con ellos automáticamente.

> **Importante:** TiDB Cloud exige conexión cifrada. El modo SSL viene por defecto en
> `VERIFY_IDENTITY` (variable `DB_SSL_MODE`), así que **no hace falta cargarlo** en
> Render. El driver MySQL valida el certificado contra los CA del sistema, que ya vienen
> incluidos en la imagen `eclipse-temurin`.

> **Health check:** asegurate de haber fijado el **Health Check Path** a
> `/api/articulos` en los ajustes del servicio (Paso 3.3). Render consulta esa ruta
> para saber si la app está viva.

### 5 — Deployar backend

1. Render compilará la imagen (etapa Maven → jar → imagen JRE) y arrancará el
   contenedor. El primer build tarda unos minutos.
2. Cuando el estado sea **Live**, probá la API en la URL pública que da Render:
   ```bash
   # Listar artículos (al inicio devuelve [] )
   curl https://<tu-servicio>.onrender.com/api/articulos

   # Crear un artículo
   curl -X POST https://<tu-servicio>.onrender.com/api/articulos \
        -H "Content-Type: application/json" \
        -d '{"nombre":"Teclado","precio":29.99,"imagen":"teclado.png"}'
   ```


### 6 — Deployar frontend

El frontend (React + Vite) se publica en **Vercel**, que sirve el build estático y se
comunica con la API de Render.

1. Subí el proyecto del frontend a un repositorio de **GitHub**.
2. En Vercel, andá a **Add New > Project** y elegí ese repositorio.
   - Si el repo contiene varios proyectos, en **Root Directory** seleccioná la carpeta del
     frontend.
   - **Framework Preset:** Vercel detecta **Vite** automáticamente
     (build: `npm run build`, output: `dist`).
3. En **Environment Variables** agregá la URL de la API publicada en Render:

   | Variable       | Valor |
   |----------------|-------|
   | `VITE_API_URL` | `https://entrega-final-26139-josefuentes.onrender.com` |

4. Click en **Deploy**. Al terminar, Vercel te da la URL pública
   (`https://<tu-proyecto>.vercel.app`) — esa es la que va en la sección
   [🔗 Demo y enlaces](#-demo-y-enlaces).

> El frontend ya tiene habilitado el CORS contra la API (`@CrossOrigin` en el controller),
> así que no hay que configurar nada extra del lado del backend.


---

## 3. Debug en VS Code

Guía para debuggear esta API usando el debugger de VS Code.

> **Importante:** este proyecto lee la conexión de **variables de entorno**
> (`DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `DB_SSL_MODE`).

### 3.1 Extensiones necesarias

En VS Code (`Ctrl+Shift+X` para abrir el panel de extensiones) instalá:

- **Extension Pack for Java** (Microsoft) — incluye el debugger de Java. **Imprescindible.**
- **Spring Boot Extension Pack** — soporte específico para Spring Boot (opcional pero recomendado).

### 3.2 Requisitos previos (la base de datos)

El debugger arranca la app y necesita una base de datos accesible. Hay dos opciones, una
por cada configuración de debug:

**a) MySQL local**

- **MySQL corriendo** en `localhost:3306`.
- La base **`articulos_db` creada**:
  ```sql
  CREATE DATABASE IF NOT EXISTS articulos_db;
  ```
- **SSL desactivado**: un MySQL local no tiene TLS, por eso la configuración local usa
  `DB_SSL_MODE=DISABLED` (el valor por defecto de la app es `VERIFY_IDENTITY`).

**b) TiDB Cloud**

- Los datos de conexión del cluster (**Connect > Connect with .env**).
- No hay que modificar SSL (default `VERIFY_IDENTITY`).

### 3.3 Prueba rápida (sin configurar nada)

1. Abrí `src/main/java/com/ejemplo/articulos/ArticuloApiApplication.java`.
2. Arriba del método `main` aparece un enlace **`Run | Debug`** (un *CodeLens*).
3. Hacé click en **Debug**.

### 3.4 Debug con `launch.json`

> **Importante:** no hagas commit de este archivo si tiene tus claves.

`.vscode/launch.json` define **dos** configuraciones:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "Debug (MySQL local)",
      "request": "launch",
      "mainClass": "com.ejemplo.articulos.ArticuloApiApplication",
      "projectName": "articulos-api",
      "console": "integratedTerminal",
      "env": {
        "DB_HOST": "localhost",
        "DB_PORT": "3306",
        "DB_DATABASE": "articulos_db",
        "DB_USERNAME": "root",
        "DB_PASSWORD": "****",
        "DB_SSL_MODE": "DISABLED"
      }
    },
    {
      "type": "java",
      "name": "Debug (TiDB Cloud)",
      "request": "launch",
      "mainClass": "com.ejemplo.articulos.ArticuloApiApplication",
      "projectName": "articulos-api",
      "console": "integratedTerminal",
      "env": {
        "DB_HOST": "gateway01.<region>.prod.aws.tidbcloud.com",
        "DB_PORT": "4000",
        "DB_DATABASE": "articulos_db",
        "DB_USERNAME": "<prefijo>.root",
        "DB_PASSWORD": "****"
      }
    }
  ]
}
```

Para usarlo:

1. Abrí el panel **Run and Debug** (`Ctrl+Shift+D`).
2. Elegí **"Debug (MySQL local)"** o **"Debug (TiDB Cloud)"**.
3. Click al ícono ▶ verde (o `F5`).

---

## 4. Probar la API en vivo (Postman y CURL)

Los ejemplos apuntan a la API publicada en Render. Si la corrés en local, reemplazá la
URL base por `http://localhost:8080`.

> 🔗 URL base de producción: `https://entrega-final-26139-josefuentes.onrender.com`

### 4.1 Endpoints

| Método   | Ruta                  | Descripción            | Body (JSON) |
|----------|-----------------------|------------------------|-------------|
| `GET`    | `/api/articulos`      | Lista todos            | —           |
| `GET`    | `/api/articulos/{id}` | Trae uno por id        | —           |
| `POST`   | `/api/articulos`      | Crea un artículo       | sí          |
| `PUT`    | `/api/articulos/{id}` | Actualiza un artículo  | sí          |
| `DELETE` | `/api/articulos/{id}` | Elimina un artículo    | —           |

El cuerpo JSON de un artículo tiene esta forma:

```json
{ "nombre": "Teclado", "precio": 29.99, "imagen": "teclado.png" }
```

### 4.2 Pruebas con CURL

```bash
# Listar todos (al inicio devuelve [])
curl https://entrega-final-26139-josefuentes.onrender.com/api/articulos

# Obtener uno por id
curl https://entrega-final-26139-josefuentes.onrender.com/api/articulos/1

# Crear un artículo
curl -X POST https://entrega-final-26139-josefuentes.onrender.com/api/articulos \
     -H "Content-Type: application/json" \
     -d '{"nombre":"Teclado","precio":29.99,"imagen":"teclado.png"}'

# Actualizar el artículo con id 1
curl -X PUT https://entrega-final-26139-josefuentes.onrender.com/api/articulos/1 \
     -H "Content-Type: application/json" \
     -d '{"nombre":"Teclado mecánico","precio":49.99,"imagen":"teclado.png"}'

# Eliminar el artículo con id 1
curl -X DELETE https://entrega-final-26139-josefuentes.onrender.com/api/articulos/1
```

> **En PowerShell (Windows)** usá `curl.exe` en lugar de `curl` (este último es un alias
> de `Invoke-WebRequest` y maneja distinto las comillas).

#### Respuesta en el navegador

Un `GET` se puede abrir directo en el navegador. Devuelve el JSON con la lista de artículos:

![Listado de artículos en formato JSON en el navegador](docs/img/api-browser.png)

### 4.3 Pruebas con Postman

1. Creá una request nueva (**New > HTTP Request**).
2. Elegí el **método** (GET, POST, etc.) y pegá la **URL** del endpoint.
3. Para `POST` y `PUT`: pestaña **Body** → **raw** → tipo **JSON**, y pegá el cuerpo:
   ```json
   { "nombre": "Teclado", "precio": 29.99, "imagen": "teclado.png" }
   ```
4. Presioná **Send** y revisá el código de estado y la respuesta.

![Request POST en Postman con el body JSON y la respuesta](docs/img/postman-post.png)

> Opcional: podés agrupar las 5 requests en una **Collection** y exportarla
> (`Export > Collection v2.1`) para incluirla en la entrega.
