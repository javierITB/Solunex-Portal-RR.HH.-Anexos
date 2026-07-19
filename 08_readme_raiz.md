# 08 · README Raíz del Proyecto — Solunex Portal RRHH

> *Copia del archivo `README.md` ubicado en la raíz del monorepo*

---

# Solunex portal RRHH

Bienvenido al espacio de trabajo unificado de **Solunex portal RRHH**, una plataforma integrada de alto rendimiento para la gestión de recursos humanos, automatización de plantillas legales de contratos (Tiptap Compiler) y administración de Domicilios Virtuales comerciales y tributarios.

---

## 🏗️ Arquitectura del Sistema

El sistema implementa una arquitectura desacoplada y de alto rendimiento:

1. **Capa de Presentación (Frontend)**: Construida con **React v18, Vite, Tailwind CSS y Redux**. Incorpora cifrado de extremo a extremo utilizando Web Crypto API y un diseño adaptativo con micro-animaciones (Framer Motion).
2. **Capa de Lógica (Backend API)**: Servidor **Node.js con Express v5**. Presenta una arquitectura multi-inquilino (multi-tenant) aislando conexiones de **MongoDB Atlas** por cada empresa.
3. **Capa de Procesamiento Asíncrono (Redis + BullMQ)**: Para garantizar tiempos de respuesta rápidos, las tareas pesadas se delegan a *workers* en segundo plano apoyados en **Redis**:
   - `answerWorker`: Generación y compilación de documentos de Word (DOCX) con asistencia de IA (Groq).
   - `domicilioVirtualWorker`: Automatización y sincronización de servicios de domicilio.
   - `mailWorker`: Cola estructurada para el envío transaccional de correos SMTP.
   - `scheduledActionWorker`: Motor de programación para automatizar acciones futuras (correos, mensajes, etc.).

---

## ✨ Últimas Funcionalidades Implementadas

En las iteraciones más recientes, el sistema ha recibido actualizaciones significativas:

- **Programación de Actividades y Mensajería**: Nueva zona para programar correos electrónicos, cargar documentos de forma asíncrona y enviar mensajes directos dentro del sistema.
- **Renovación del Creador de Anuncios**: Interfaz de creación de anuncios y comunicaciones totalmente estilizada y reordenada, incorporando selectores de destinatarios avanzados y lógicas condicionales detalladas.
- **Optimización de Rendimiento y Paginación**: Sistema de paginación global implementado en zona de clientes, solicitudes y notificaciones. Esto ha **reducido en un 60% el tiempo de carga** de la página inicial.
- **Mejoras de UI/UX**: Corrección integral del modo oscuro en secciones de formularios y de búsqueda.

---

## ⚙️ Guía Rápida de Arranque en Desarrollo

### 1. Requisitos Previos

Asegúrate de tener instalado y corriendo **Redis**:

```bash
# macOS
brew services start redis

# Linux (Debian/Ubuntu)
sudo systemctl start redis-server
```

### 2. Levantar el Backend (Express API)

```bash
cd ./back_vercel
npm install
npm start
```

El servidor correrá en `http://localhost:3000` conectado a MongoDB Atlas.

### 3. Levantar el Frontend (React SPA)

```bash
cd ./react_app
npm install
npm run dev
```

La aplicación estará activa en `http://localhost:3001`.

---

## 🗂️ Documentación Completa

Ver la carpeta [`docs/`](docs/00_README.md) para la documentación técnica completa organizada para el Comité de Titulación.
