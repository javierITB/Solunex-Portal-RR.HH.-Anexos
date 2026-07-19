# 06 · Guía de Usuario y Casos de Uso — Solunex Portal RRHH

> **Para:** Administradores de sistema, Gestores de RRHH y Colaboradores finales  
> **Dominio:** `solunex.cl` (Producción) · `localhost:3001` (Desarrollo local)

---

## 1. Modelo de Roles (RBAC — Control de Acceso Basado en Roles)

Solunex gestiona la seguridad mediante un modelo jerárquico de cuatro perfiles. Cada rol hereda las capacidades del nivel inferior y añade sus propias.

```
[Maestro]                ← Superusuario global (acceso a todos los Tenants)
    │
[Administrador]          ← Gestiona su empresa, usuarios y planes
    │
[RRHH (Gestor)]          ← Atiende solicitudes y administra documentos
    │
[Usuario / Cliente]      ← Levanta solicitudes y consulta la IA
```

### Matriz de Permisos

| Funcionalidad | Maestro | Administrador | RRHH | Cliente |
|:--------------|:-------:|:-------------:|:----:|:-------:|
| Acceso Multi-Tenant | 🌐 Global | 🔒 Solo su empresa | 🔒 Solo su empresa | 🔒 Solo su empresa |
| Crear/Eliminar Empresas | ✅ | ❌ | ❌ | ❌ |
| Gestionar Usuarios | ✅ | ✅ | ❌ | ❌ |
| Form Builder y Plantillas | ✅ | ✅ | ✅ | ❌ |
| Revisar y Gestionar Solicitudes | ✅ | ✅ | ✅ | 👁 Solo lectura |
| Portal de Soporte (Tickets) | ✅ | ✅ | ✅ | ✅ Crear tickets |
| Chatbot Asesor Legal IA | ✅ | ✅ | ❌ | ✅ |
| Monitor de Rendimiento | ✅ | ✅ | ❌ | ❌ |
| Domicilio Virtual | ✅ | ✅ | ✅ | ✅ Ver propio |
| Anuncios y Comunicaciones | ✅ | ✅ | ✅ | 👁 Solo lectura |

---

## 2. Casos de Uso por Perfil

### Caso de Uso 1 — Administrador General (Maestro)

El Maestro es el perfil técnico máximo del sistema. Sus responsabilidades y capacidades son:

**Supervisión Global de la Plataforma:**
- Accede a cualquier base de datos de empresa (tenant) desde el mismo panel.
- Visualiza el **Monitor de Sistema** con métricas en tiempo real: CPU de Node.js, RAM del host y latencia de base de datos.
- Ejecuta pruebas de red en vivo (latencia Frontend → Backend → Base de Datos).
- Configura umbrales de alerta: si la CPU supera el 85% o la DB supera 1 segundo de latencia, el sistema notifica automáticamente por plataforma y correo.

**Gestión de la Infraestructura SaaS:**
- Registra nuevas empresas clientes en el registro global `config_empresas`.
- Configura los planes de facturación y límites de uso por empresa.
- Ejecuta migraciones de datos y operaciones administrativas de mantenimiento.

---

### Caso de Uso 2 — Administrador Inquilino (por Empresa)

El Administrador opera estrictamente dentro del entorno aislado de su organización. Sus flujos principales son:

**Gestión de Usuarios y Accesos:**
1. Crea cuentas de usuario asignando el rol correspondiente (RRHH o Cliente).
2. Activa o desactiva cuentas.
3. Visualiza el historial de ingresos de cada usuario (IP, dispositivo, fecha — cifrados).

**Form Builder (Constructor de Formularios):**
1. Diseña formularios dinámicos con preguntas de múltiples tipos (texto, selección, fecha, adjuntos).
2. Crea y asigna plantillas Word a formularios, usando la sintaxis Tiptap Compiler.
3. Configura qué empresas tienen acceso a cada formulario.

**Gestión de Solicitudes:**
1. Revisa las solicitudes entrantes de colaboradores.
2. Cambia estados del flujo: Pendiente → Procesado → Firmado → Aprobado → Archivado.
3. Descarga los documentos `.docx` generados automáticamente.
4. Valida y concilia comprobantes de pago subidos por clientes.

**Domicilio Virtual:**
1. Visualiza el panel general con todos los contratos de domicilio activos, filtros por estado y búsqueda.
2. Extiende contratos vencidos (anual o manual).
3. Importa clientes masivamente desde un archivo CSV.
4. Gestiona el estado del contrato (Firmado físicamente, Informado SII, DICOM, Dado de Baja).

---

### Caso de Uso 3 — Gestor de RRHH

El Gestor opera como ejecutor operativo de los procesos documentales:

**Emisión Administrativa de Formularios:**
1. Crea solicitudes en nombre de un colaborador (flujo admin → cliente).
2. El sistema identifica al destinatario por Blind Index de correo y asigna la solicitud al perfil correcto.
3. El formulario queda disponible en el panel del colaborador.

**Gestión de Tickets de Soporte:**
1. Atiende los tickets creados por clientes.
2. Chateo interno dentro del ticket, con posibilidad de adjuntar documentos.
3. Cambia el estado del ticket: Abierto → En Proceso → Resuelto.

---

### Caso de Uso 4 — Cliente / Colaborador (Usuario Final)

El colaborador es el origen del flujo transaccional. Sus acciones desencadenan la generación automatizada de documentos:

**Completar un Trámite (Flujo Principal):**
1. Ingresa al portal y navega a la sección del trámite correspondiente (Contratos, Remuneraciones, Finiquitos, Teletrabajo, etc.).
2. Responde el formulario reactivo con sus datos personales y laborales.
3. Si el formulario requiere archivos, adjunta documentos (cédula, certificados) mediante subida individual.
4. Presiona "Enviar". El sistema cifra sus respuestas en tránsito y las persiste en MongoDB (estado: pendiente).
5. Un Worker en segundo plano genera el documento `.docx` de forma asíncrona sin bloquear la interfaz.
6. El colaborador recibe una notificación de confirmación y, opcionalmente, un correo de respaldo.

**Subir Comprobante de Pago:**
1. Navega a la sección de pagos.
2. Completa el monto, banco emisor y adjunta captura o PDF de la transferencia.
3. El equipo administrativo recibe alerta y concilia el pago.

**Portal de Soporte:**
1. Crea un ticket seleccionando el departamento (Finanzas, Legal, Soporte Técnico, RRHH).
2. Describe el requerimiento.
3. Recibe respuesta del gestor asignado dentro del hilo del ticket.

**Asesor Legal IA (Chatbot):**
1. Accede a la sección "Asesor IA".
2. Realiza consultas sobre legislación laboral chilena, SII o trámites comerciales.
3. El sistema responde mediante el modelo LLaMA-3.1 con inferencia de baja latencia (Groq).

> **Limitación importante del Chatbot:** El Asesor IA es una **herramienta orientativa** y NO reemplaza la asesoría jurídica de un abogado colegiado. Tiene **prohibición absoluta** de redactar contratos de trabajo o escrituras públicas.

---

## 3. Flujo de Autenticación (Login)

### Acceso Estándar (Sin 2FA)
1. Usuario ingresa correo y contraseña en el formulario.
2. El frontend cifra las credenciales con AES-256-GCM antes de enviarlas.
3. El backend descifra, busca al usuario por Blind Index de correo, verifica la contraseña con Argon2id.
4. Si es correcta, emite un Opaque Token de sesión (válido por 4 horas).
5. El frontend almacena el token y lo envía en el header `Authorization: Bearer <token>` en cada petición.

### Acceso con Doble Factor (2FA)
1. Mismo flujo hasta la verificación de contraseña.
2. El sistema detecta `twoFactorEnabled = true` y **NO** emite el token todavía.
3. Genera un código numérico de 6 dígitos, lo hashea con Argon2id y lo almacena con TTL de 5 minutos.
4. Envía el código al correo registrado del usuario.
5. El usuario ingresa el código en la interfaz.
6. El backend verifica el código contra el hash. Si es válido, emite el Opaque Token de sesión.

---

## 4. Funcionalidades Adicionales

### Activar Autenticación de Doble Factor (2FA)

1. Ir al menú de perfil (esquina superior derecha) → `Mi Perfil`.
2. Ir a la sección de Seguridad.
3. Activar la casilla "Habilitar 2FA".
4. Ingresar la contraseña actual para confirmar.
5. En el próximo inicio de sesión, se solicitará el código enviado al correo.

### Monitor de Sistema (Solo Administrador / Maestro)

1. Navegar a `Rendimiento → Monitor de Sistema`.
2. Visualizar gráficas en tiempo real: CPU, RAM, latencia de DB.
3. Ejecutar "Live Test" para medir la latencia E2E (Front → API → DB) en milisegundos.
4. Configurar umbrales de alerta automática desde el botón "Umbrales".

---

## 5. Glosario Técnico para Usuarios

| Término | Significado |
|:--------|:------------|
| **Tenant** | Empresa cliente con su propia base de datos aislada |
| **RBAC** | Control de Acceso Basado en Roles |
| **2FA** | Autenticación de Dos Factores (código OTP por correo) |
| **OTP** | One-Time Password — Código de uso único |
| **Opaque Token** | Token de sesión aleatorio almacenado en base de datos (más seguro que JWT) |
| **BullMQ** | Sistema de colas que procesa tareas pesadas en segundo plano |
| **Worker** | Proceso en segundo plano que ejecuta las tareas de la cola |
| **Blind Index** | Hash SHA-256 del correo para buscar en datos cifrados |
| **Argon2id** | Algoritmo de hashing de contraseñas de grado industrial |
| **AES-256-GCM** | Algoritmo de cifrado simétrico estándar militar |

---

*[← API Endpoints](05_api_endpoints.md) · [Siguiente: Reporte de Rendimiento →](07_reporte_pruebas_rendimiento.md)*
