# 05 · Especificación de API REST — Solunex Portal RRHH

> **URL Base:** `https://api.solunex.cl/{company}/`  
> **Documentación Interactiva (Swagger):** `/api-docs`  
> **Formato:** JSON con cifrado AES-256-GCM en tránsito  
> **Autenticación:** Opaque Token (Bearer) o API Master Key (administración)

---

## 1. Convenciones Globales

### Estructura de URL Multi-Tenant

```
https://api.solunex.cl/{company}/{modulo}/{recurso}
```

El parámetro `{company}` es el identificador del tenant. Ejemplos:
- `api` o `solunex` → Tenant del sistema
- `empresa1`, `constructoraX` → Tenants de clientes registrados

**Rutas globales (sin tenant):**
- `/monitoring/*` → Telemetría del servidor
- `/sas/plans` → Gestión de planes de facturación

### Cabeceras de Autenticación

```http
# Sesión de colaborador
Authorization: Bearer <opaque_token_hex>

# Operaciones administrativas (bypass)
x-api-master-key: <api_master_key>
```

### Cifrado en Tránsito (E2E)

Todas las peticiones POST/PUT/PATCH con body JSON deben enviarse cifradas:

```json
{
  "iv": "base64_de_12_bytes_aleatorios",
  "payload": "base64_del_ciphertext_mas_authTag",
  "encrypted": true
}
```

Las respuestas del servidor se devuelven en **JSON plano** (no cifrado).

---

## 2. Módulo de Autenticación

**Prefijo:** `/{company}/auth`

---

### `POST /auth/login`

Autentica credenciales. Aplica Rate Limiting reforzado.

**Request Body (antes de cifrar):**
```json
{
  "email": "colaborador@empresa.com",
  "password": "mi_clave_segura"
}
```

**Respuesta — Login exitoso (sin 2FA):**
```json
{
  "success": true,
  "token": "a1f9d8c3e7b2f4a9...",
  "expiresAt": "2026-07-19T20:00:00.000Z",
  "usr": {
    "name": "Juan",
    "lastName": "Pérez",
    "email": "colaborador@empresa.com",
    "cargo": "Analista de RRHH",
    "rol": "RRHH",
    "userId": "695fb53c627ed819c757e310"
  }
}
```

**Respuesta — 2FA requerido:**
```json
{
  "success": true,
  "twoFA": true,
  "userId": "695fb53c627ed819c757e310",
  "email": "colaborador@empresa.com",
  "message": "Se requiere código 2FA. Enviado a tu correo."
}
```

**Errores posibles:**

| Código | Mensaje |
|:------:|:--------|
| 400 | Datos incompletos |
| 401 | Credenciales inválidas |
| 401 | Usuario pendiente de activación |
| 401 | Usuario inactivo |
| 429 | Rate Limit excedido |

---

### `POST /auth/verify-login-2fa`

Verifica el código OTP de 6 dígitos y emite el token de sesión.

**Request Body:**
```json
{
  "email": "colaborador@empresa.com",
  "verificationCode": "485923"
}
```

**Respuesta:** Igual al login directo exitoso (con `token`, `expiresAt`, `usr`).

---

### `POST /auth/recuperacion`

Genera y envía un código de recuperación de contraseña al correo.

**Request Body:**
```json
{ "email": "colaborador@empresa.com" }
```

**Respuesta:**
```json
{ "message": "Código de recuperación enviado con éxito." }
```

---

### `GET /auth/full/:mail`

Recupera el perfil completo descifrado del usuario. Protegido por Bearer Token.

**Respuesta:**
```json
{
  "_id": "695fb53c627ed819c757e310",
  "nombre": "Juan",
  "apellido": "Pérez",
  "mail": "colaborador@empresa.com",
  "empresa": "Constructora empresa1",
  "cargo": "Analista",
  "rol": "RRHH",
  "notificaciones": [],
  "twoFactorEnabled": false,
  "estado": "activo"
}
```

---

## 3. Módulo de Formularios y Respuestas

**Prefijo:** `/{company}/respuestas`

---

### `POST /respuestas/`

Guarda las respuestas de un formulario, las cifra y las encola en BullMQ para generación del documento `.docx`.

**Request Body (antes de cifrar):**
```json
{
  "formId": "695fb53c627ed819c757e307",
  "formTitle": "Contrato Plazo Fijo",
  "user": {
    "token": "a1f9d8c3...",
    "nombre": "Juan Pérez",
    "empresa": "empresa1",
    "uid": "695fb53c..."
  },
  "responses": {
    "NombreCompleto": "Juan Pérez",
    "RutTrabajador": "12.345.678-9",
    "SueldoBase": "800000"
  },
  "adjuntos": [],
  "mail": "juan@empresa.com"
}
```

**Respuesta (200 OK — inmediata):**
```json
{
  "_id": "695fb53c627ed819c757e350",
  "formId": "695fb53c627ed819c757e307",
  "message": "Respuesta guardada exitosamente.",
  "user": { ... },
  "responses": { ... }
}
```

> El documento `.docx` se genera de forma **asíncrona** por un Worker en segundo plano. La respuesta es inmediata para no bloquear al usuario.

---

### `POST /respuestas/:id/adjuntos`

Sube un archivo adjunto individual (Multipart/form-data). Procesado por Multer.

**Form Data:**
| Campo | Tipo | Descripción |
|:------|:-----|:------------|
| `archivo` | File | Archivo binario (máx 10 MB) |
| `index` | Number | Índice del archivo (0-based) |
| `total` | Number | Total de archivos a subir |
| `pregunta` | String | Nombre del campo del formulario |

**Respuesta:**
```json
{
  "success": true,
  "message": "Adjunto 1/3 recibido",
  "fileName": "cedula_identidad.pdf"
}
```

---

## 4. Módulo de Domicilio Virtual

**Prefijo:** `/{company}/domicilio-virtual`

---

### `GET /domicilio-virtual/mini`

Lista el panel general con paginación y filtros avanzados.

**Query params opcionales:**
- `page`, `limit`, `status`, `search`, `dateRange`

**Respuesta:**
```json
{
  "success": true,
  "data": [
    {
      "_id": "...",
      "nombreEmpresa": "Inversiones del Sur Ltda",
      "rutEmpresa": "77.892.481-K",
      "status": "documento_generado",
      "fechaInicioContrato": "26/05/2026",
      "fechaTerminoContrato": "25/05/2027"
    }
  ],
  "pagination": { "total": 45, "page": 1, "limit": 30, "totalPages": 2 },
  "stats": { "total": 45, "documento_generado": 20, "pendiente": 5, ... }
}
```

---

### `POST /domicilio-virtual/:id/extend`

Extiende la vigencia del contrato (anual o manual).

**Request Body:**
```json
{ "type": "anual" }
```

---

### `POST /domicilio-virtual/import-csv`

Importación masiva de clientes desde CSV (Multipart/form-data).

**Comportamiento:** BulkWrite con Upsert. Busca clientes existentes por Blind Index del RUT.

**Respuesta:**
```json
{
  "success": true,
  "message": "Éxito. Se crearon 12 clientes nuevos y se actualizaron 3 existentes."
}
```

---

## 5. Módulo de Asesor Legal IA (Chatbot)

**Prefijo:** `/{company}/chat`

---

### `POST /chat/`

Envía una consulta y retorna la respuesta del modelo Groq (LLaMA-3.1-8b-instant).

**Request Body (antes de cifrar):**
```json
{ "message": "¿Cuál es el plazo para pagar un finiquito en Chile?" }
```

**Respuesta:**
```json
{
  "success": true,
  "response": "De acuerdo al artículo 177 del Código del Trabajo..."
}
```

---

### `GET /chat/history`

Historial de mensajes activos, descifrado en memoria antes de retornar.

---

### `POST /chat/clear`

Borrado lógico del historial (marca `active: false`, sin eliminar físicamente).

---

## 6. Módulo de Monitoreo de Rendimiento

**Prefijo:** `/monitoring` (ruta global — sin tenant)

---

### `GET /monitoring/ping`

```json
"pong"
```

### `GET /monitoring/metrics/current`

Retorna CPU, RAM y latencia de DB en tiempo real.

```json
{
  "hardware": {
    "memory": { "totalMb": 8192, "usedMb": 2048, "usedPercent": "25.00" },
    "cpu": { "cores": 4, "nodeCpuPercent": "2.50" },
    "uptimeSeconds": 86400
  },
  "database": { "status": "connected", "latencyMs": 42 }
}
```

### `GET /monitoring/metrics/historical`

Serie temporal de métricas (cada 5 minutos, 7 días de retención).

**Query param:** `limit` (default: 288 = 24 horas)

### `GET|POST /monitoring/config`

Leer y actualizar umbrales de alerta del sistema:

```json
{
  "cpuThreshold": 85,
  "ramThresholdMb": 14000,
  "dbLatencyThresholdMs": 1000,
  "notifyPlatform": true,
  "notifyEmail": true,
  "alertEmail": "admin@empresa.cl"
}
```

---

## 7. Manejo de Errores

La API retorna errores estándar en el siguiente formato:

```json
{
  "error": "Descripción del error",
  "success": false
}
```

| Código HTTP | Causa |
|:-----------:|:------|
| 400 | Datos mal formados o fallo de descifrado de integridad |
| 401 | Token inválido, expirado o no proporcionado |
| 403 | Límite de plan excedido o empresa no autorizada |
| 404 | Recurso no encontrado / Tenant no registrado |
| 429 | Rate Limit excedido (demasiadas peticiones) |
| 500 | Error interno del servidor |

---

*[← Manual del Desarrollador](04_manual_desarrollador.md) · [Siguiente: Guía de Usuario →](06_guia_usuario.md)*
