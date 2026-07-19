# Solunex Portal RRHH — Documentación Técnica Completa

> **Proyecto de Título de Ingeniería en Informática**  
> Acciona Centro de Negocios SpA · 2026  
> Versión del sistema: `2.1.0`

---

## Descripción del Proyecto

**Solunex Portal RRHH** es un sistema web de automatización documental y gestión de procesos para departamentos de Recursos Humanos, con integración de Inteligencia Artificial. Permite la generación automática de contratos, anexos de teletrabajo y documentos legales a partir de formularios reactivos, con asistencia de revisión ortográfica y consultas jurídicas mediante IA (Groq/LLaMA-3).

El sistema opera bajo una **arquitectura distribuida Cliente-Servidor**, implementando separación de bases de datos por empresa (Multi-Tenant), procesamiento asíncrono en segundo plano (BullMQ + Redis), seguridad criptográfica de grado industrial (AES-256-GCM + Argon2id) y un modelo de Control de Acceso Basado en Roles (RBAC).

---

## Índice de Documentación

La carpeta `docs/` contiene la documentación organizada en secciones numeradas. Se recomienda leer en el siguiente orden:

### 📐 Arquitectura y Diseño
| # | Documento | Contenido |
|---|-----------|-----------|
| 01 | [Arquitectura General](01_arquitectura_general.md) | Vista global del sistema, diagramas de flujo, stack tecnológico completo, colas asíncronas y pipeline del generador de documentos. |
| 02 | [Modelo de Datos NoSQL](02_modelo_datos_nosql.md) | Colecciones MongoDB, esquemas polimórficos, índices compuestos y la estrategia Multi-Tenant de aislamiento físico de datos. |

### 🔐 Seguridad
| # | Documento | Contenido |
|---|-----------|-----------|
| 03 | [Seguridad y Criptografía](03_seguridad_criptografia.md) | Cifrado End-to-End (AES-256-GCM), cifrado en reposo a nivel de celda, Blind Indexing (SHA-256), Hashing Argon2id, 2FA y auditoría de accesos. |

### 👩‍💻 Para Desarrolladores
| # | Documento | Contenido |
|---|-----------|-----------|
| 04 | [Manual del Desarrollador](04_manual_desarrollador.md) | Instrucciones de instalación local, variables de entorno, estructura de directorios y guía completa del motor de plantillas Tiptap Compiler. |
| 05 | [Especificación de API REST](05_api_endpoints.md) | Catálogo completo de endpoints, convenciones de autenticación, ejemplos de request/response y manejo de errores. |

### 👤 Para Usuarios
| # | Documento | Contenido |
|---|-----------|-----------|
| 06 | [Guía de Usuario y Casos de Uso](06_guia_usuario.md) | Flujos de trabajo por rol (Maestro, Administrador, RRHH, Cliente), matriz de permisos RBAC y uso del Asesor Legal IA. |

### 📊 Validación y Rendimiento
| # | Documento | Contenido |
|---|-----------|-----------|
| 07 | [Reporte de Pruebas de Carga](07_reporte_pruebas_rendimiento.md) | Resultados empíricos completos de pruebas de estrés, escalabilidad vertical, horizontal y análisis de diminishing returns. Incluye datos de 5 escenarios distintos. |

### 📋 Copias de READMEs del Repositorio
| # | Documento | Contenido |
|---|-----------|-----------|
| 08 | [README Raíz del Proyecto](08_readme_raiz.md) | README original del monorepositoriocon guía rápida de arranque. |
| 09 | [README Backend](09_readme_backend.md) | README del servidor Node.js/Express con detalles de workers y configuración. |

---

## Tecnologías Principales en un Vistazo

```
Frontend:  React 18 · Vite · Redux Toolkit · Tailwind CSS · Framer Motion · TipTap
Backend:   Node.js · Express 5 · MongoDB (Driver Nativo) · BullMQ · Redis
Seguridad: AES-256-GCM · Argon2id · SHA-256 (Blind Index) · Opaque Tokens
IA:        Groq SDK · LLaMA-3.1-8b-instant (inferencia ultrarrápida)
Correos:   Nodemailer · SMTP propio (mail.solunex.cl)
Proxy:     Nginx (Reverse Proxy + TLS Termination)
```

---

## Roles del Sistema

| Rol | Descripción |
|-----|-------------|
| **Maestro** | Superusuario con acceso global a todos los tenants |
| **Administrador** | Gestiona su empresa, usuarios y planes contratados |
| **RRHH (Gestor)** | Emite formularios, crea plantillas y revisa solicitudes |
| **Usuario / Cliente** | Responde trámites, sube comprobantes y consulta la IA |

---

*Documentación generada para el Comité de Titulación — Proyecto de Título 2026*
