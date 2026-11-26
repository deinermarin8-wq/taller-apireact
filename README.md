# Gestor de tareas

Aplicación web sencilla para consultar y agregar tareas. El proyecto está dividido en un frontend hecho con React y Vite, una API hecha con FastAPI y una base de datos MySQL.

## Funciones actuales

- Lista las tareas guardadas en MySQL.
- Permite agregar una tarea con un título de hasta 80 caracteres.
- Muestra estados de carga y mensajes de error o confirmación.

La aplicación actualmente no incluye edición, eliminación ni marcado de tareas como completadas.

## Tecnologías

| Parte | Tecnología |
| --- | --- |
| Interfaz | React 19 |
| Servidor de desarrollo del frontend | Vite |
| API | FastAPI y Uvicorn |
| Base de datos | MySQL, accedida mediante PyMySQL |
| Validación de datos | Pydantic |

## Estructura del proyecto

```text
gestor-tareas/
├── README.md
├── backend/
│   ├── .env                 # Credenciales locales; no subir al repositorio
│   ├── requirements.txt
│   ├── schema.sql           # Archivo disponible para el esquema de MySQL
│   └── app/
│       ├── db.py            # Carga de configuración y conexión a MySQL
│       ├── main.py          # Endpoints de la API
│       └── schemas.py       # Modelos y validación de datos
└── frontend/
    ├── package.json
    ├── vite.config.js       # Configura el proxy /api hacia el backend
    └── src/
        ├── app.jsx          # Interfaz y acciones de tareas
        ├── main.jsx         # Punto de entrada de React
        ├── index.css        # Estilos globales
        └── App.jsx          # Archivo de plantilla de Vite (no usado por main.jsx)
```

## Requisitos

- Python 3.10 o posterior.
- Node.js y npm.
- MySQL Server en ejecución.
- Una base de datos llamada `gestor_tareas` y una tabla `tareas` con las columnas `id` (entero autoincremental) y `titulo` (texto de hasta 80 caracteres).

Por ejemplo, el esquema mínimo puede crearse en MySQL con:

```sql
CREATE DATABASE IF NOT EXISTS gestor_tareas
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE gestor_tareas;

CREATE TABLE IF NOT EXISTS tareas (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  titulo VARCHAR(80) NOT NULL
);
```

El usuario de base de datos configurado en `.env` necesita permisos sobre `gestor_tareas`.

## Configuración del backend

Desde la carpeta `backend`, crea o activa un entorno virtual e instala las dependencias:

```powershell
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Crea `backend/.env` con los datos de tu instalación de MySQL:

```dotenv
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=gestor_tareas
DB_USER=gestor_app
DB_PASSWORD=tu_contraseña
```

No publiques ni compartas este archivo. `db.py` lo carga desde la carpeta `backend`.

Inicia la API desde `backend`:

```powershell
python -m uvicorn app.main:app --reload --port 8000
```

La documentación interactiva de FastAPI queda disponible en <http://127.0.0.1:8000/docs>.

## Configuración del frontend

En otra terminal, desde la carpeta `frontend`, instala paquetes e inicia Vite:

```powershell
cd frontend
npm install
npm run dev
```

Abre la dirección que indique Vite en la terminal (normalmente <http://localhost:5173>). Vite reenvía las solicitudes que empiezan por `/api` al backend en `http://127.0.0.1:8000`.

## API

| Método | Ruta | Descripción | Respuesta esperada |
| --- | --- | --- | --- |
| `GET` | `/api/tareas` | Devuelve todas las tareas ordenadas por ID. | Lista de `{ "id": 1, "titulo": "..." }` |
| `POST` | `/api/tareas` | Crea una tarea. El cuerpo JSON es `{ "titulo": "..." }`. | `201 Created` y la tarea creada |

El título debe tener entre 1 y 80 caracteres. La API también elimina espacios al principio y al final antes de guardar.

## Flujo de una solicitud

1. React solicita `GET /api/tareas` al cargar la pantalla.
2. Vite reenvía `/api` al servidor FastAPI.
3. FastAPI valida los datos con los modelos de `schemas.py` y llama a `db.py` para conectarse a MySQL.
4. El endpoint ejecuta la consulta, confirma los cambios cuando crea una tarea y devuelve JSON.
5. React actualiza la lista que muestra en pantalla.

## Solución de problemas

- **Error 1045 / Access denied:** revisa `DB_USER`, `DB_PASSWORD` y el `Host` de la cuenta MySQL. El usuario debe tener acceso a `gestor_tareas`.
- **Error de tabla desconocida:** crea la base y la tabla `tareas` con el esquema indicado arriba.
- **Error de conexión o timeout:** confirma que MySQL esté en ejecución y que `DB_HOST` y `DB_PORT` sean correctos.
- **Error 500 al agregar:** revisa la terminal donde corre Uvicorn; el traceback indica si falló la consulta, la tabla o la transacción.
- **Frontend no llega a la API:** confirma que Uvicorn esté escuchando en el puerto 8000 y que Vite siga usando el proxy de `vite.config.js`.
