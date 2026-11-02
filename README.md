# ☕ CaféPro Colombia

Aplicación web para la gestión integral de una unidad productiva cafetera: registro de agricultores, registro de pases de cosecha (kilos secos, kilos cereza, precio de venta y egresos), cálculo de ganancia, calendario agrícola, fase lunar, clima y noticias del sector.

- **Frontend:** HTML5, CSS3, Bootstrap 5.3.8 y JavaScript (POO con la clase `GestorCafePro`).
- **Backend:** Node.js, Express y MySQL, con API REST y Programación Orientada a Objetos.

---

## 1. Estructura del proyecto

```
cafepro/
├── index.html
├── app.js                    # Frontend (consume la API)
├── styles.css
├── decor-rama.svg
├── fondo-colinas.svg
├── README.md
└── cafepro-backend/
    ├── config/
    │   └── db.js                     # Clase Database (pool de conexiones MySQL)
    ├── controllers/
    │   ├── BaseController.js         # Clase padre (manejo de errores)
    │   ├── UsuarioController.js      # CRUD de usuarios + login
    │   └── RegistroController.js     # CRUD de registros de cosecha
    ├── routes/
    │   ├── usuarios.routes.js
    │   └── registros.routes.js
    ├── sql/
    │   └── schema.sql                # Script de la base de datos
    ├── .env.example                  # Plantilla de variables de entorno
    ├── app.js                        # Clase Servidor (punto de entrada)
    └── package.json
```

## 2. Requisitos

- [Node.js](https://nodejs.org) (versión LTS)
- [MySQL Server](https://dev.mysql.com/downloads/) encendido (se recomienda MySQL Workbench)
- Un navegador moderno
- Insomnia o Postman (para probar la API)

## 3. Instalación y ejecución

### 3.1 Crear la base de datos

Ejecuta el script `cafepro-backend/sql/schema.sql` en MySQL Workbench, o desde la terminal:

```bash
mysql -u root -p < cafepro-backend/sql/schema.sql
```

Esto crea la base `cafepro_db` con las tablas `usuarios` y `registros_cosecha`.

### 3.2 Configurar variables de entorno

Dentro de `cafepro-backend/`, copia `.env.example` a `.env` y coloca tu contraseña de MySQL:

```env
PORT=3000
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=tu_contraseña_de_mysql
DB_NAME=cafepro_db
```

### 3.3 Instalar dependencias y arrancar el backend

```bash
cd cafepro-backend
npm install
npm run dev
```

Resultado esperado:

```
✅ Conectado a MySQL
🚀 Servidor corriendo en http://localhost:3000
```

### 3.4 Abrir el frontend

Abre `index.html` en el navegador (o con la extensión *Live Server* de VS Code). El backend debe estar encendido para poder ingresar y guardar registros.

## 4. Modelo de datos

| Tabla | Campos |
|---|---|
| `usuarios` | `id`, `nombre_agricultor`, `nombre_cafetal` (único), `password_hash`, `created_at` |
| `registros_cosecha` | `id`, `usuario_id` (FK → `usuarios.id`, `ON DELETE CASCADE`), `fecha`, `kilos_seco`, `kilos_cereza`, `precio_venta` (por kilo), `egresos`, `ganancia`, `created_at` |

La ganancia se calcula en el servidor: `ganancia = kilos_seco × precio_venta − egresos`.

## 5. Endpoints de la API

URL base: `http://localhost:3000/api`

### Usuarios (agricultores)

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/usuarios` | Listar usuarios |
| GET | `/usuarios/:id` | Obtener un usuario |
| POST | `/usuarios` | Crear usuario |
| PUT | `/usuarios/:id` | Actualizar usuario |
| DELETE | `/usuarios/:id` | Eliminar usuario (y sus registros) |
| POST | `/usuarios/login` | Autenticación |

**Crear usuario** — `POST /api/usuarios`
```json
{
  "nombre_agricultor": "Juan Pérez",
  "nombre_cafetal": "Finca La Esperanza",
  "password": "123456"
}
```

**Login** — `POST /api/usuarios/login`
```json
{
  "nombre_cafetal": "Finca La Esperanza",
  "password": "123456"
}
```

### Registros de cosecha

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/registros` | Listar todos los registros |
| GET | `/registros?usuario_id=1` | Listar los registros de un agricultor |
| GET | `/registros/:id` | Obtener un registro |
| POST | `/registros` | Crear registro |
| PUT | `/registros/:id` | Actualizar registro |
| DELETE | `/registros/:id` | Eliminar registro |

**Crear registro** — `POST /api/registros`
```json
{
  "usuario_id": 1,
  "fecha": "2026-11-01",
  "kilos_seco": 125,
  "kilos_cereza": 625,
  "precio_venta": 14800,
  "egresos": 300000
}
```
La `fecha` es opcional (si se omite, se usa la fecha de hoy).

### Códigos de respuesta

| Código | Significado |
|---|---|
| 200 | OK |
| 201 | Recurso creado |
| 400 | Datos faltantes o inválidos |
| 401 | Credenciales incorrectas |
| 404 | Recurso no encontrado |
| 409 | Dato duplicado (cafetal ya registrado) |
| 500 | Error interno del servidor |

## 6. Programación Orientada a Objetos

| Concepto | Dónde se aplica |
|---|---|
| **Clases** | `Database`, `BaseController`, `UsuarioController`, `RegistroController`, `Servidor` y `GestorCafePro` (frontend) |
| **Herencia** | `UsuarioController` y `RegistroController` extienden `BaseController` |
| **Encapsulamiento** | La lógica de validación y de acceso a datos vive dentro de cada clase |
| **Abstracción** | Las rutas solo llaman a métodos como `crear()` o `login()` sin conocer el detalle interno |
| **Asincronía** | Métodos `async/await` para todas las operaciones con la base de datos |

## 7. Seguridad aplicada

- Las contraseñas se almacenan cifradas con **bcrypt** (nunca en texto plano).
- Las consultas usan **parámetros preparados** (`?`) para evitar inyección SQL.
- El archivo `.env` está excluido del repositorio con `.gitignore`.
- Las respuestas de la API nunca incluyen el `password_hash`.

## 8. Flujo de la aplicación

1. El agricultor escribe su nombre, el nombre del cafetal y su contraseña. Si el cafetal ya existe ingresa; si no, se crea la cuenta (`POST /usuarios/login` y, si falla, `POST /usuarios`).
2. En **Finanzas** registra un pase de cosecha (`POST /registros`). El historial se carga con `GET /registros?usuario_id=...` y cada registro puede eliminarse (`DELETE /registros/:id`).
3. Los datos viven en MySQL, no en el navegador.

## 9. Solución de problemas

| Error | Causa probable |
|---|---|
| `ER_ACCESS_DENIED_ERROR` | Usuario o contraseña de MySQL incorrectos en `.env` |
| `ER_BAD_DB_ERROR` | No se ejecutó `schema.sql` |
| `ECONNREFUSED` | MySQL no está encendido |
| `EADDRINUSE` | El puerto 3000 ya está en uso |
| "No se pudo conectar con el servidor" (en la web) | El backend no está corriendo (`npm run dev`) |

## 10. Autor

Proyecto desarrollado por **Darwin** — Ingeniería Electrónica, UNAD.
