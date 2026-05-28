# 🏢 Reserva de Salas — Guía de instalación y configuración

## ¿Qué hace esta app?

- **Vista de calendario semanal/diaria** estilo Teams con las reservas de ambas salas en colores distintos
- **Anti-solapamiento**: impide que dos personas reserven la misma sala al mismo horario
- **Integración real con Microsoft Teams**: crea la reunión en el calendario de Teams con enlace para unirse
- **Visible para todos los usuarios**: al reservar, todos los participantes ven la reunión en su propio calendario de Teams
- **Cancelación de reuniones**: elimina la reunión de Teams y de todos los calendarios
- **PWA instalable**: funciona como app nativa en Windows, Mac, Android e iOS

---

## Paso 1: Registrar la app en Azure Active Directory

1. Ve a https://portal.azure.com → **Azure Active Directory** → **Registros de aplicaciones** → **+ Nuevo registro**
2. Completa:
   - **Nombre**: `SalasApp` (o el que prefieras)
   - **Tipos de cuenta admitidos**: *Cuentas en este directorio organizacional únicamente*
   - **URI de redirección**: selecciona `SPA (aplicación de página única)` y escribe la URL donde alojarás la app (ej. `https://salas.tuempresa.com` o `http://localhost:8080` para pruebas locales)
3. Haz clic en **Registrar**
4. Copia el **Id. de aplicación (cliente)** y el **Id. de directorio (inquilino)**

---

## Paso 2: Configurar permisos de API

1. En tu app registrada, ve a **Permisos de API** → **+ Agregar un permiso**
2. Selecciona **Microsoft Graph** → **Permisos delegados**
3. Agrega estos permisos:
   - `Calendars.ReadWrite`
   - `OnlineMeetings.ReadWrite`
   - `User.Read`
4. Haz clic en **Conceder consentimiento de administrador para [tu organización]** ✅

---

## Paso 3: (Opcional) Configurar salas como recursos de Exchange

Para que las reservas aparezcan también en la agenda de la sala (sala de recursos):

1. En el **Centro de administración de Microsoft 365** → **Salas y equipos** → **+ Agregar sala**
2. Crea las 2 salas, anota sus emails (ej. `sala-directorio@empresa.com`)
3. Ingresa esos emails en la configuración de la app

---

## Paso 4: Alojar la app

### Opción A: Servidor web local (pruebas)
```bash
# Con Python
python3 -m http.server 8080

# Con Node.js
npx serve .
```
Luego abre http://localhost:8080

### Opción B: Azure Static Web Apps (recomendado para producción)
1. Sube los archivos a un repositorio GitHub
2. En Azure Portal → **Static Web Apps** → **+ Crear**
3. Conecta tu repositorio, build output: `/` (no build needed)
4. Actualiza el URI de redirección en Azure AD con la URL de Azure

### Opción C: GitHub Pages
1. Sube los archivos a un repositorio público
2. Habilita GitHub Pages desde Settings → Pages
3. Actualiza el URI de redirección en Azure AD

### Opción D: SharePoint como host
Sube `index.html` como página de SharePoint o web part de contenido incrustado.

---

## Paso 5: Configurar la app

1. Abre la app en tu navegador
2. Haz clic en **"Configura tu Azure App"**
3. Ingresa:
   - Client ID (copiado en Paso 1)
   - Tenant ID (copiado en Paso 1)
   - Nombres de las salas
   - Emails de recursos de Exchange (opcional)
4. Haz clic en **Guardar configuración**
5. Haz clic en **Iniciar sesión con Microsoft**

---

## Instalar como app (PWA)

### En Windows (Chrome/Edge):
- Abre la app en el navegador
- Haz clic en el ícono de instalación en la barra de direcciones (⊕)
- O menú ⋮ → "Instalar Reserva de Salas"

### En iPhone/iPad:
- Abre en Safari
- Toca el botón Compartir → "Añadir a pantalla de inicio"

### En Android:
- Abre en Chrome
- Toca menú ⋮ → "Instalar app" o "Añadir a pantalla de inicio"

---

## Estructura de archivos

```
salas-app/
├── index.html      ← App completa (HTML + CSS + JS)
├── manifest.json   ← Configuración PWA
├── sw.js           ← Service Worker (funcionamiento offline)
└── icons/
    ├── icon-192.png
    └── icon-512.png
```

---

## Cómo funciona la integración con Teams

Cuando un usuario crea una reserva:
1. La app llama a `POST /me/events` de Microsoft Graph
2. El evento se crea con `isOnlineMeeting: true` → Teams genera automáticamente un enlace de reunión
3. Si tienes emails de sala configurados, se agregan como asistentes de tipo "recurso" → aparece en la agenda de la sala
4. El evento aparece en el calendario de Teams/Outlook del organizador
5. Todos los usuarios con acceso a los calendarios compartidos verán la reserva

Para que **todos** los usuarios de la empresa vean las reservas, existen dos estrategias:
- **Calendarios compartidos**: Configura los emails de sala como recursos de Exchange (ver Paso 3)
- **Canal de Teams con calendario compartido**: Crea un canal de Teams dedicado a reservas y usa el tab de Calendar

---

## Permisos necesarios (resumen)

| Permiso | Para qué sirve |
|---------|----------------|
| `User.Read` | Ver el perfil del usuario actual |
| `Calendars.ReadWrite` | Crear/leer/eliminar eventos en el calendario |
| `OnlineMeetings.ReadWrite` | Crear reuniones de Teams con enlace de unión |

---

## Soporte

La app guarda las reservas localmente como respaldo. Si la integración con Graph falla (sin conexión), las reservas se guardan en el navegador y el anti-solapamiento sigue funcionando. Al reconectarse, sincroniza automáticamente.
