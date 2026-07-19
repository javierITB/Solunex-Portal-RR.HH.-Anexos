# 04 · Manual del Desarrollador — Solunex Portal RRHH

> **Para:** Ingenieros de desarrollo, colaboradores técnicos y auditores de código  
> **Requisitos previos:** Node.js LTS, npm, Redis, acceso a MongoDB Atlas

---

## 1. Estructura del Repositorio

El proyecto es un **monoreposit** con dos aplicaciones independientes:

```
Accionaweb/                         ← Raíz del monorepo
│
├── docs/                           ← Documentación técnica completa (esta carpeta)
│
├── back_vercel/                    ← Backend: Node.js + Express 5
│   ├── config/                     ← Configuraciones auxiliares
│   ├── endpoints/                  ← Controladores HTTP por módulo
│   │   ├── auth.js                 ← Autenticación, RBAC, 2FA
│   │   ├── answers.js              ← Formularios y generación de documentos
│   │   ├── monitoring.js           ← Telemetría y métricas del servidor
│   │   ├── chatbot.js              ← Asesor Legal IA (Groq)
│   │   ├── domicilioVirtual.js     ← Módulo de Domicilio Virtual
│   │   ├── plantillas.js           ← Gestión de plantillas Word
│   │   └── ...                     ← Resto de módulos
│   ├── queues/                     ← Definición de colas BullMQ
│   │   ├── answerQueue.js
│   │   ├── mailQueue.js
│   │   └── domicilioQueue.js
│   ├── workers/                    ← Procesadores asíncronos
│   │   ├── answerWorker.js         ← Genera .docx + Groq + notificaciones
│   │   ├── mailWorker.js           ← SMTP transaccional
│   │   ├── domicilioVirtualWorker.js
│   │   ├── scheduledActionWorker.js
│   │   ├── anuncioSchedulerWorker.js
│   │   └── monitoringCron.js       ← Cron de métricas (cada 5 min)
│   ├── utils/                      ← Helpers transversales
│   │   ├── seguridad.helper.js     ← AES-256-GCM, Argon2id, Blind Index
│   │   ├── generador.helper.js     ← Motor de plantillas Tiptap Compiler
│   │   ├── mail.helper.js          ← Nodemailer (sync + async)
│   │   ├── validarToken.js         ← Validación de Opaque Tokens
│   │   ├── rateLimiter.js          ← express-rate-limit (global + auth)
│   │   ├── planLimits.js           ← Verificación de límites de plan SaaS
│   │   └── ...
│   ├── tests/                      ← Suite de pruebas
│   │   ├── unit/                   ← Pruebas unitarias (Jest)
│   │   ├── integration/            ← Pruebas de integración (Supertest)
│   │   └── benchmarks/             ← Scripts de carga (Autocannon)
│   ├── index.js                    ← Punto de entrada: middleware, rutas, DB
│   ├── swagger.js                  ← Especificación OpenAPI
│   ├── jest.config.js
│   └── package.json
│
└── react_app/                      ← Frontend: React 18 + Vite
    ├── public/                     ← Assets estáticos
    └── src/
        ├── clientPages/            ← Portal del Cliente (formularios, pagos, IA)
        ├── pages/                  ← Portal de RRHH / Admin (gestión, dashboard)
        ├── components/             ← Componentes UI reutilizables
        ├── context/                ← Providers de contexto (RBAC, permisos)
        ├── utils/
        │   ├── api.js              ← Cliente HTTP con cifrado E2E automático
        │   └── ...
        ├── Routes.jsx              ← Enrutamiento protegido por rol
        └── index.jsx               ← Entry point DOM
```

---

## 2. Variables de Entorno Requeridas

### Backend — `back_vercel/.env`

| Variable | Tipo | Descripción |
|:---------|:-----|:------------|
| `PORT` | Number | Puerto del servidor Express (default: `3000`) |
| `MONGO_URI` | String | Cadena de conexión MongoDB Atlas (`mongodb+srv://...`) |
| `MASTER_KEY` | Hex (64 chars) | Llave AES-256 de 32 bytes para cifrado en reposo |
| `TRANSIT_SECRET` | String | Secreto compartido para descifrado de payloads E2E |
| `API_MASTER_KEY` | String | Token administrativo para bypass controlado |
| `GROQ_API_KEY` | String | API Key de Groq Cloud (`gsk_...`) |
| `SMTP_HOST` | String | Servidor SMTP saliente (ej: `mail.solunex.cl`) |
| `SMTP_PORT` | Number | Puerto SMTP (465 para SSL, 587 para TLS) |
| `SMTP_SECURE` | Boolean | `true` para SSL/TLS |
| `SMTP_USER` | String | Cuenta de correo emisora |
| `SMTP_PASS` | String | Contraseña de autenticación SMTP |
| `NODE_ENV` | String | `production` / `desarrollo` / `test` |
| `TZ` | String | `America/Santiago` (se pasa en el script npm start) |

### Frontend — `react_app/.env`

| Variable | Tipo | Descripción |
|:---------|:-----|:------------|
| `VITE_API_BASE_URL` | String | URL base del backend (ej: `http://localhost:3000`) |
| `VITE_API_MASTER_KEY` | String | Token maestro (solo para pruebas locales) |
| `VITE_TRANSIT_SECRET` | String | Mismo valor que `TRANSIT_SECRET` del backend |

> **Seguridad:** Todos los `.env` están en `.gitignore`. **Nunca** compartir estos valores en el repositorio.

---

## 3. Instalación y Puesta en Marcha

### Paso 1 — Levantar Redis

BullMQ requiere Redis activo para funcionar:

```bash
# macOS (Homebrew)
brew install redis
brew services start redis

# Linux (Debian/Ubuntu)
sudo apt install redis-server
sudo systemctl start redis-server

# Docker (cualquier sistema)
docker run -d -p 6379:6379 redis:alpine
```

### Paso 2 — Backend

```bash
cd back_vercel
npm install
npm start
```

La consola confirmará:
```
✅ Conectado a MongoDB Atlas
🚀 Workers de BullMQ inicializados y escuchando
⏰ Scheduler de anuncios programados inicializado
[API Server] Running on http://0.0.0.0:3000
```

La documentación Swagger estará disponible en: `http://localhost:3000/api-docs`

### Paso 3 — Frontend

```bash
cd react_app
npm install
npm run dev
```

La SPA estará disponible en: `http://localhost:3001` (o el puerto que indique Vite)

---

## 4. Ejecución de Pruebas

### Pruebas Unitarias e Integración (Jest + Supertest)

```bash
cd back_vercel
npm test
```

La suite de pruebas usa `mongodb-memory-server` para aislar completamente la base de datos de producción durante los tests.

### Benchmarks de Rendimiento (Autocannon)

```bash
cd back_vercel
node tests/benchmarks/run_benchmarks.js
```

---

## 5. Motor de Plantillas — Tiptap Compiler

El sistema de generación de documentos interpreta las plantillas HTML creadas con el editor Tiptap y las transforma en archivos `.docx` con datos reales.

### Variables Simples

Cualquier token entre dobles llaves `{{}}` será reemplazado con la respuesta del formulario:

```
{{NOMBRE_TRABAJADOR}}      → Juan Pérez
{{RUT_EMPRESA}}            → 76.543.210-K
{{SUELDO_BASE}}            → 800000
```

**Comportamiento de casing automático:**
- `{{NOMBRE}}` → MAYÚSCULAS SOSTENIDAS
- `{{nombre}}` → minúsculas
- `{{Nombre}}` → Caso Mixto Original

**Filtro de fechas:** Si la variable contiene `FECHA`, `INICIO` o `CONTRATO`, el valor ISO `2026-05-26` se formatea a `26 de mayo de 2026`.

### Token Especial `{{NUMERAL}}`

Genera ordinales en español con autoincremento secuencial:
```
{{NUMERAL}} → PRIMERO
{{NUMERAL}} → SEGUNDO
{{NUMERAL}} → DÉCIMO PRIMERO
```

### Variables Secuenciales

Si una variable se repite, el parser busca variantes numeradas:
```
{{DIRECCION}}    → Primer uso
{{DIRECCION_2}}  → Segundo uso
{{DIRECCION_3}}  → Tercer uso (fallback al original si no existe)
```

### Condicionales `[[IF: ... ]]`

Ocultan o muestran bloques completos según las respuestas:

```html
<p>[[IF: TIPO_CONTRATO = PLAZO_FIJO]]</p>
<p>Este contrato tiene vigencia hasta el {{FECHA_TERMINO}}.</p>
<p>[[ENDIF]]</p>
```

**Operadores soportados:**

| Operador | Función | Ejemplo |
|:--------:|:--------|:--------|
| `=` / `==` | Igualdad (case-insensitive) | `[[IF: PLAN = ANUAL]]` |
| `!=` | Desigualdad | `[[IF: CARGO != MAESTRO]]` |
| `><` | Contiene (para checkboxes/arrays) | `[[IF: BENEFICIOS >< SEGURO_MEDICO]]` |
| `<` | Menor que (numérico o fecha) | `[[IF: SUELDO < 500000]]` |
| `>` | Mayor que | `[[IF: MONTO > 100000]]` |

**Encadenamiento lógico:**
```
[[IF: SUELDO > 500000 && CARGO = ANALISTA]]   ← Ambas condiciones
[[IF: PLAN = ANUAL || MONTO > 100000]]        ← Cualquiera de las dos
```

---

## 6. Convenciones de Código

- **Patrón de autenticación:** Toda ruta protegida llama a `validarToken(req.db, token)` antes de cualquier operación.
- **Multi-Tenant:** Nunca hardcodear `"formsdb"`. Usar siempre `req.db` que ya viene inyectado por el middleware.
- **Cifrado:** Siempre usar `cifrarObjeto()` antes de insertar, y `descifrarObjeto()` antes de retornar.
- **Búsquedas:** Nunca buscar por campos cifrados directamente. Siempre usar `createBlindIndex()` sobre el campo de búsqueda.
- **Respuesta asíncrona:** Operaciones pesadas (`.docx`, correos) siempre via cola BullMQ. Nunca bloquear el hilo principal.

---

*[← Seguridad](03_seguridad_criptografia.md) · [Siguiente: API Endpoints →](05_api_endpoints.md)*
