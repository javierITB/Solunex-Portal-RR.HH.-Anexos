# 09 · README Backend — Solunex Portal RRHH

> *Copia del archivo `back_vercel/README.md`*

---

# Solunex Portal RRHH — Backend API (Express)

La capa lógica central de Solunex es un servidor **Node.js** apoyado en **Express v5**, enfocado en operaciones de alta seguridad y procesamiento concurrente mediante concurrencia de eventos asíncronos y soporte nativo de promesas.

---

## 🏗️ Arquitectura Multi-Inquilino (Multi-tenant)

Todo el tráfico de la API cruza por un `tenantRouter`. Este middleware:

1. Extrae el subdominio de origen (ej: `empresaA.solunex.cl`).
2. Valida la existencia y estado de la empresa en la base de datos principal (`formsdb`).
3. Retorna y cachea una conexión dinámica y exclusiva de base de datos de **MongoDB Atlas**, aislando la información de cada compañía sin requerir clústeres separados.
4. Ejecuta un descifrado AES-256-GCM (mediante `Node Crypto`) de los datos en tránsito asegurados por el frontend.

---

## 🔐 Arquitectura de Seguridad

El backend implementa un sistema de defensa en profundidad con múltiples capas:

- **Cifrado End-to-End:** Middleware global descifra todos los payloads cifrados con AES-256-GCM antes de pasarlos a los controladores.
- **Hashing Argon2id:** Contraseñas hasheadas con parámetros de grado industrial (64 MB memoria + 3 iteraciones).
- **Blind Indexing:** Búsquedas en datos cifrados mediante hashes SHA-256 deterministas.
- **Opaque Tokens:** Tokens de sesión aleatorios almacenados en MongoDB (revocación inmediata posible).
- **Rate Limiting:** Límites reforzados sobre el endpoint de autenticación para prevenir ataques de fuerza bruta.
- **CORS Estricto:** Lista blanca de orígenes permitidos con soporte para subdominios dinámicos.

---

## ⚙️ Workers y Colas Asíncronas (BullMQ + Redis)

| Worker | Cola | Función |
|:-------|:-----|:--------|
| `answerWorker` | `tareas-respuestas` | Genera .docx, revisión IA (Groq), envía correos, notifica |
| `mailWorker` | `tareas-correo` | SMTP transaccional con reintentos automáticos |
| `domicilioVirtualWorker` | `tareas-domicilio` | Contratos de domicilio virtual |
| `scheduledActionWorker` | `tareas-programadas` | Acciones diferidas en el tiempo |
| `monitoringCron` | Node-Cron | Métricas de sistema cada 5 minutos |

---

## ✨ Últimos Avances en el Servidor

1. **Gestión de Acciones Programadas (`scheduledActions`):** Endpoints para recepcionar, almacenar y programar acciones asíncronas en el tiempo, integrándose con la cola `scheduledActionQueue`.
2. **Consultas Paginadas Optimizadas:** Los endpoints críticos (`/answers`, notificaciones) utilizan cursores con `limit`/`skip`, reduciendo la transferencia de payload en un 60%.
3. **Resoluciones de Conectividad:** Ajustes en políticas CORS permitiendo enrutado fluido entre subdominios de inquilinos y dominios principales.
4. **Monitor de Rendimiento:** Nuevo módulo `/monitoring` con métricas en tiempo real de CPU, RAM y latencia de base de datos, con umbrales de alerta configurables.

---

## 🚀 Inicio en Desarrollo

```bash
# Desde la carpeta back_vercel/
npm install
npm start
```

**Prerequisito:** Redis debe estar corriendo localmente en el puerto 6379.

**Confirmación de arranque exitoso:**
```
✅ Conectado a MongoDB Atlas
🚀 Workers de BullMQ inicializados y escuchando
⏰ Scheduler de anuncios programados inicializado
[API Server] Running on http://0.0.0.0:3000
```

**Documentación Swagger disponible en:** `http://localhost:3000/api-docs`

---

## 🧪 Pruebas

```bash
npm test   # Jest + Supertest + mongodb-memory-server
```

---

*Para documentación completa ver [docs/](00_README.md)*
