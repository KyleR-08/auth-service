# auth-service

Microservicio de **autenticación** construido con **FastAPI**, **SQLModel** y **AES (pycryptodome)**.

Forma parte del **Proyecto Integrador del Tercer Semestre** de la **Universidad Internacional del Ecuador (UIDE)**, materia **Fundamentos de la Seguridad**, dictada por el profesor **Darío Cabezas**.

Este servicio corre dentro de la **VLAN 10** del entorno del proyecto y se encarga del **registro e inicio de sesión** de los usuarios del sistema.

---

## Tecnologías

- **FastAPI** — framework web
- **SQLModel** — ORM sobre SQLAlchemy
- **SQLite** — base de datos local (`database.db`)
- **pycryptodome** — cifrado simétrico AES de las contraseñas
- **python-dotenv** — carga de variables de entorno

---

## Conceptos de seguridad aplicados

- **Cifrado simétrico AES (modo CBC) con padding PKCS7:** las contraseñas se cifran antes de guardarse en la base de datos, nunca se almacenan en texto plano.
- **IV aleatorio por contraseña:** se genera un Vector de Inicialización (IV) único de 16 bytes para cada cifrado, de modo que dos contraseñas iguales produzcan ciphertexts distintos.
- **Clave AES de 32 bytes (AES-256):** se carga desde la variable de entorno `AES_KEY` definida en `.env`. Si la clave es más corta, se rellena con `=` hasta llegar a 32 bytes.
- **Formato de almacenamiento:** `iv_base64:ciphertext_base64`, lo que permite recuperar el IV al descifrar.

> A diferencia del hashing (que es unidireccional), el cifrado es **reversible**: en el login se descifra la contraseña almacenada y se compara con la enviada por el usuario.

---

## Estructura del proyecto

```
auth-service/
├── main.py
├── models.py
├── auth.py
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

---

## Instalación

### 1. Situarse en el proyecto

```bash
cd auth-service
```

### 2. Crear y activar un entorno virtual

**Windows (PowerShell):**

```powershell
python -m venv venv
venv\Scripts\activate
```

**Linux / macOS:**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Configurar variables de entorno

El archivo `.env` viene con un valor de ejemplo:

```
AES_KEY=MiClaveSecretaAES32Bytes12345678
```

> En producción, la `AES_KEY` debe ser un valor secreto de exactamente 32 bytes y **nunca** debe subirse al repositorio.

---

## Ejecución

Levantar el servidor en modo desarrollo:

```bash
fastapi dev main.py
```

Por defecto FastAPI abrirá el servicio en:

- API: <http://127.0.0.1:8000>
- Documentación interactiva (Swagger UI): <http://127.0.0.1:8000/docs>

La base de datos `database.db` se crea automáticamente al iniciar la aplicación.

---

## Endpoints

### `POST /register`

Registra un nuevo usuario. La contraseña se cifra con AES-256-CBC antes de guardarse.

**Request:**

```json
{
  "username": "alice",
  "password": "supersecreta"
}
```

**Respuesta exitosa:**

```json
{
  "message": "Usuario registrado correctamente.",
  "user_id": 1,
  "username": "alice"
}
```

**Respuesta si el usuario ya existe (400):**

```json
{
  "detail": "El username ya está registrado."
}
```

### `POST /login`

Verifica las credenciales del usuario descifrando la contraseña almacenada.

**Request:**

```json
{
  "username": "alice",
  "password": "supersecreta"
}
```

**Respuesta exitosa:**

```json
{
  "message": "Login exitoso.",
  "username": "alice"
}
```

**Respuesta si las credenciales son inválidas (401):**

```json
{
  "detail": "Credenciales inválidas."
}
```

---

## Ejemplos con `curl`

```bash
curl -X POST http://127.0.0.1:8000/register \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"alice\",\"password\":\"supersecreta\"}"

curl -X POST http://127.0.0.1:8000/login \
  -H "Content-Type: application/json" \
  -d "{\"username\":\"alice\",\"password\":\"supersecreta\"}"
```

---

## Créditos

- **Universidad:** Universidad Internacional del Ecuador (UIDE)
- **Materia:** Fundamentos de la Seguridad
- **Profesor:** Darío Cabezas
- **Semestre:** Tercero
- **Despliegue:** VLAN 10
