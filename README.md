# MercaControl — Documentación Técnica

Sistema de control de entrada y salida de mercaderistas para Supertiendas Cañaveral.
Aplicación web tipo PWA (Progressive Web App), sin backend propio — todo corre en el navegador.

---

## 🌐 Despliegue

- **URL producción:** `https://fumaratoo.github.io/mercacontrol1`
- **Plataforma:** GitHub Pages (estático, archivo único `index.html`)
- **Cómo actualizar:** subir nuevo `index.html` al repositorio → GitHub Pages se actualiza en ~2 min

> ⚠️ Los datos de la app viven en `localStorage` del navegador del usuario, atados al dominio `fumaratoo.github.io`. Si el dominio cambia, los datos no se migran automáticamente.

---

## 🔐 Autenticación — Microsoft Azure AD

El login usa **MSAL (Microsoft Authentication Library)** con flujo redirect (sin popups).

### Configuración Azure (líneas 3–24 de MercaControl.jsx)
| Parámetro | Valor |
|-----------|-------|
| Client ID | `0652ef4a-cd47-4994-91ac-afbb86a8a2ba` |
| Tenant ID | `ae0b754a-1500-4ce6-b39e-59d483b9ae9c` |
| Redirect URIs registradas | `https://fumaratoo.github.io/mercacontrol1` · `https://fumaratoo.github.io` · `https://stcanaveral.sharepoint.com` |

### Flujo de login
1. Usuario hace clic en "Iniciar sesión con Microsoft"
2. `loginRedirect()` redirige a Microsoft
3. Microsoft redirige de vuelta con token en la URL
4. El script de pre-inicialización en `index.html` procesa el redirect **antes** de que React monte
5. La cuenta queda en `window.__msalAccount`
6. `LoginScreen` lee esa cuenta y determina el rol

### Pre-inicialización MSAL (en `index.html`, antes del `<script type="text/babel">`)
Existe un script separado que llama `handleRedirectPromise()` antes de que React arranque. Esto es crítico para mobile donde los popups están bloqueados.

---

## 👥 Roles de usuario

Los roles se determinan en `LoginScreen` (línea 258) comparando el email del usuario con las listas almacenadas:

| Rol | Cómo se asigna | Permisos |
|-----|----------------|----------|
| **Admin** | Lista `mreg-v1-admins` en localStorage | Todo: ver, crear registros, gestionar catálogos, panel admin |
| **Comercial** | Lista `mreg-v1-comercials` | Ver dashboard, descargar Excel, gestionar empresas (agregar, activar/desactivar) |
| **Espectador** | Lista `mreg-v1-viewers` | Solo ver dashboard y descargar Excel |
| **Seguridad** | Lista `mreg-v1-guardias` (con sedeId asignado) | Vista de guardia: registrar entradas/salidas en su sede asignada |

> Los guardias **no pueden elegir su sede** — el admin la asigna desde el panel. Si un email de guardia no está en la lista, la app muestra error "contacta al administrador".

---

## 💾 Almacenamiento de datos — localStorage

Toda la data persiste en `localStorage` del navegador. Hook principal: `useSharedStorage` (línea 141).

| Clave localStorage | Contenido |
|--------------------|-----------|
| `mreg-v1-registros` | Array de registros de entrada/salida |
| `mreg-v1-empresas` | Catálogo de empresas `{id, nombre, activa}` |
| `mreg-v1-sedes` | Catálogo de sedes `{id, nombre, activa}` |
| `mreg-v1-admins` | Lista de admins `{email, addedBy, addedAt}` |
| `mreg-v1-viewers` | Lista de espectadores |
| `mreg-v1-guardias` | Lista de guardias `{email, sedeId, addedBy, addedAt}` |
| `mreg-v1-comercials` | Lista de comerciales |
| `mreg-v1-acuerdos` | Acuerdos comerciales `{id, empresaId, sedeId, horas{}, notas}` |

### Estructura de un registro
```json
{
  "id": "reg-1234567890",
  "consecutivo": 42,
  "tipo": "entrada",
  "timestamp": 1700000000000,
  "empresaId": "e123",
  "empresa": "Nombre Empresa",
  "sedeId": "s456",
  "sedeName": "Sede Norte",
  "guardiaName": "Juan Pérez",
  "guardiaId": "juan@stcanaveral.com",
  "gps": { "lat": 10.123, "lng": -75.456 },
  "pendingSync": false,
  "syncedAt": 1700000001000
}
```

### Estructura de un acuerdo comercial
```json
{
  "id": "ac-1234567890",
  "empresaId": "e123",
  "sedeId": "s456",
  "horas": {
    "lunes": 2, "martes": 0, "miércoles": 3,
    "jueves": 0, "viernes": 4, "sábado": 0, "domingo": 0
  },
  "notas": "Mercaderista llega antes de las 8am",
  "updatedAt": 1700000000000
}
```

---

## 🗂️ Componentes — mapa del código

### MercaControl.jsx (archivo principal, ~2750 líneas)

| Línea | Componente / Sección | Descripción |
|-------|---------------------|-------------|
| 3 | `MSAL Config` | Client ID, Tenant ID, configuración Azure |
| 45 | `Global Styles` | CSS global, animaciones, fuentes |
| 68 | `T (Theme)` | Paleta de colores centralizada |
| 102 | `Initial Data` | Datos de ejemplo iniciales (sedes y empresas demo) |
| 135 | `Utils` | Funciones helper: `fmtDate`, `fmtTime`, `fmtDuracion` |
| 141 | `useSharedStorage` | Hook de persistencia en localStorage |
| 161 | `useOnlineStatus` | Hook que detecta si hay internet |
| 174 | `OfflineBanner` | Barra amarilla/roja que aparece sin conexión |
| 228 | `Badge`, `InfoRow`, `Spinner` | Micro-componentes reutilizables |
| 258 | `LoginScreen` | Pantalla de login Microsoft, determina rol |
| 362 | `NuevoRegistroForm` | Formulario para registrar entrada/salida |
| 543 | `SecurityView` | Vista del guardia de seguridad |
| 648 | `DetailModal` | Modal con detalle de un registro |
| 724 | `EditModal` | Modal para editar un registro existente |
| 782 | `AcuerdosView` | Vista de acuerdos comerciales con filtros y export |
| 1100 | `AcuerdoForm` | Formulario crear/editar acuerdo |
| 1191 | `ViewAsBar` | Barra de "Ver como" para el admin |
| 1223 | `AdminDashboard` | Dashboard principal (movimientos, visitas, acuerdos) |
| 2161 | `ComercialEmpresasPanel` | Panel flotante del rol comercial para gestionar empresas |
| 2208 | `AdminPanel` | Panel de configuración del admin (sedes, empresas, usuarios) |
| 2566 | `App` | Componente raíz — maneja routing entre vistas |
| 2745 | `Mount` | `ReactDOM.createRoot` — punto de arranque |

---

## 📊 Módulo de Acuerdos Comerciales

Cada acuerdo conecta **una empresa** con **una sede** y define cuántas horas debe estar el mercaderista cada día de la semana.

- **Quién puede ver:** Admin, Comercial, Espectador
- **Quién puede editar:** Admin y Comercial
- **Export Excel:** 3 modos — toda la compañía, por sede, por empresa
- **Columnas del Excel:** Empresa · Sede · Lun · Mar · Mié · Jue · Vie · Sáb · Dom · Total h/semana · Notas

---

## 📱 Modo offline

- `useOnlineStatus` (línea 161) detecta conexión en tiempo real
- Registros creados sin internet se guardan con `pendingSync: true`
- Al recuperar conexión, `handleSync` en `App` (línea ~2630) los procesa
- `OfflineBanner` (línea 174) muestra el estado y botón de sincronizar

---

## 🔧 Cómo hacer cambios comunes

### Agregar un nuevo rol
1. Agregar lista en `App` con `useSharedStorage` (línea ~2450)
2. Agregar `isNuevoRol` callback como `isComercial` (línea ~2473)
3. Agregar chequeo en `LoginScreen` (línea ~310)
4. Agregar pestaña en `AdminPanel` (línea ~2208)
5. Agregar routing en `App` (línea ~2490)

### Agregar un campo nuevo a los registros
1. Agregar el campo en `NuevoRegistroForm` (línea 362)
2. Agregar al objeto `newReg` en `handleNuevoRegistro` en `App` (línea ~2620)
3. Agregar columna en `exportarExcel` (línea 1221)
4. Opcionalmente mostrar en `DetailModal` (línea 648)

### Cambiar colores
Todos los colores están centralizados en el objeto `T` (Theme) en línea 68:
```js
const T = {
  navy: "#05200b",      // verde muy oscuro — headers
  navyMid: "#1C5A2A",   // verde medio
  brand: "#7dd105",     // verde lima — color principal
  amber: "#f59e0b",     // amarillo — acuerdos, alertas
  ...
}
```

### Agregar una nueva sede/empresa desde código
Modificar los arrays `SEDES_INIT` y `EMPRESAS_INIT` en línea 102. Solo aplica a instalaciones nuevas (localStorage vacío).

---

## 🚀 Tecnologías usadas

| Tecnología | Uso |
|------------|-----|
| React 18 | UI, via CDN (no hay build) |
| Babel (in-browser) | Transpila JSX en el navegador directamente |
| MSAL Browser 2.38 | Autenticación Microsoft Azure AD |
| localStorage | Persistencia de datos |
| CSS-in-JS (inline styles) | Estilos, sin dependencias externas |
| GitHub Pages | Hosting estático gratuito |

> **No hay Node.js, npm, webpack ni build process.** El archivo `index.html` se abre directamente en el navegador. Para modificar: editar `MercaControl.jsx`, reconstruir `index.html` con el script Python del repositorio de desarrollo, subir a GitHub.

---

## ⚙️ Variables de configuración críticas

Si en el futuro se cambia el tenant de Microsoft o la URL de la app, estos son los únicos valores a cambiar:

```js
// MercaControl.jsx líneas 4-5
const MSAL_CLIENT_ID = "0652ef4a-cd47-4994-91ac-afbb86a8a2ba";
const MSAL_TENANT_ID = "ae0b754a-1500-4ce6-b39e-59d483b9ae9c";
```

Y en el portal Azure → App Registration → Authentication → agregar la nueva URL como Redirect URI.

---

*Documentación generada para el equipo técnico de Supertiendas Cañaveral — v1.0*
