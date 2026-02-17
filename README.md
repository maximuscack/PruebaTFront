# Frontend Vue 3 - Gestión de Tareas

Frontend moderno para la aplicación de gestión de tareas desarrollado con Vue 3, Vite y Axios.

## 📋 Requisitos

- Node.js 18+
- npm o yarn
- Git

## 🚀 Instalación

### 1. Clonar el repositorio
```bash
git clone https://github.com/maximuscack/PruebaTFront.git
cd PruebaT/frontendV3
```

### 2. Instalar dependencias
```bash
npm install
```

### 3. Instalar Axios (si no está incluido)
```bash
npm install axios
```

## ⚙️ Configuración

### Archivo de configuración principal
El archivo principal es `src/App.vue` que contiene toda la lógica de la aplicación.

### Configuración de API
La URL del backend está configurada en:
```javascript
const API_BASE_URL = 'http://localhost:8000/api';
```

Para cambiar la URL del backend, modifica esta constante en `src/App.vue`.

## 🚀 Ejecución

### Iniciar servidor de desarrollo
```bash
npm run dev
```

El servidor estará disponible en: **http://localhost:3000** o **http://localhost:5173**

### En puerto específico
```bash
npm run dev -- --port=3001
```

### Accesible desde red
```bash
npm run dev -- --host
```

## 📱 Características de la Aplicación

### 🎨 Diseño y UX
- **Interfaz moderna** con fondos claros y texto oscuro
- **Diseño responsive** para móviles y tablets
- **Animaciones suaves** y transiciones
- **Cards visuales** para las tareas
- **Indicadores de estado** (completada, vencida, pendiente)

### 🔐 Autenticación
- **Registro de usuarios** con validación en tiempo real
- **Inicio de sesión** con tokens JWT
- **Cierre de sesión** seguro
- **Manejo de errores** específicos (422, 500, etc.)

### 📋 Gestión de Tareas
- **Crear tareas** con título, descripción y fecha/hora
- **Ver lista** de tareas con paginación
- **Editar tareas** en modales
- **Eliminar tareas** con confirmación
- **Marcar como completada** con un clic
- **Buscar tareas** en tiempo real
- **Ordenar** por fecha, título o creación

### 🎯 Estados Visuales
- **Tareas completadas:** Verde con check ✅
- **Tareas vencidas:** Rojo con advertencia ⚠️
- **Tareas pendientes:** Gris normal
- **Tareas próximas a vencer:** Amarillo ⏰

## 🗂️ Estructura del Proyecto

```
frontendV3/
├── src/
│   ├── App.vue              # Componente principal (SPA completa)
│   └── main.js             # Punto de entrada de Vue
├── public/
│   └── index.html          # Plantilla HTML
├── package.json           # Dependencias y scripts
├── vite.config.js         # Configuración de Vite
└── dist/                 # Build de producción (generado)
```

## 🌐 Comunicación con el Backend

### Endpoints utilizados
```javascript
// Autenticación
POST /api/register    // Registrar usuario
POST /api/login       // Iniciar sesión
POST /api/logout      // Cerrar sesión
GET  /api/user       // Obtener usuario

// Tareas
GET    /api/tareas           // Listar tareas
POST   /api/tareas           // Crear tarea
GET    /api/tareas/{id}      // Ver tarea
PUT    /api/tareas/{id}      // Actualizar tarea
DELETE /api/tareas/{id}      // Eliminar tarea
PATCH  /api/tareas/{id}/toggle-complete // Marcar completada
```

### Configuración de Axios
```javascript
// Configuración base
axios.defaults.baseURL = 'http://localhost:8000/api';

// Interceptor para incluir token
axios.interceptors.request.use(config => {
    const token = localStorage.getItem('token');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});
```

## 🎨 Componentes y Funcionalidades

### Componente Principal (App.vue)
El proyecto utiliza un único componente `App.vue` que contiene:

#### Secciones principales:
1. **Header con navegación**
2. **Formulario de autenticación** (login/register)
3. **Lista de tareas** con filtros
4. **Modal para crear/editar tareas**
5. **Paginación**

#### Métodos principales:
- `register()` - Registro de usuarios
- `login()` - Inicio de sesión
- `logout()` - Cierre de sesión
- `fetchTareas()` - Obtener tareas
- `createTarea()` - Crear nueva tarea
- `updateTarea()` - Actualizar tarea
- `deleteTarea()` - Eliminar tarea
- `toggleComplete()` - Marcar como completada

## 🎯 Estados y Validaciones

### Estados de la aplicación
```javascript
data() {
    return {
        user: null,                    // Usuario autenticado
        token: null,                   // Token de autenticación
        tareas: [],                    // Lista de tareas
        loading: false,                 // Estado de carga
        errors: {},                    // Errores de validación
        // ... más estados
    }
}
```

### Validaciones en frontend
- **Contraseñas coinciden** en registro
- **Campos requeridos** validados
- **Formato de email** verificado
- **Fecha de vencimiento** no anterior a hoy

## 🐛 Solución de Problemas

### Error: "No se puede conectar al backend"
**Solución:**
1. Verifica que el backend esté corriendo en `http://localhost:8000`
2. Revisa la configuración CORS en el backend
3. Limpia el cache del navegador (Ctrl+Shift+R)

### Error: "Token no válido"
**Solución:**
1. Cierra sesión y vuelve a iniciar
2. Limpia el localStorage del navegador
3. Verifica que el token no haya expirado

### Error: "Puerto en uso"
**Solución:**
```bash
npm run dev -- --port=3001
```

### Error: "Dependencias no encontradas"
**Solución:**
```bash
rm -rf node_modules package-lock.json
npm install
```

## 📊 Monitoreo y Debug

### Herramientas de desarrollador
1. **Abre DevTools** (F12)
2. **Pestaña Console** para errores
3. **Pestaña Network** para ver peticiones API
4. **Pestaña Application** para verificar localStorage

### Logs personalizados
La aplicación incluye logs en consola para:
- Errores de API
- Cambios de estado
- Eventos de usuario

## 🔧 Configuración Adicional

### Variables de entorno
Crea un archivo `.env.local` para variables específicas:
```env
VITE_API_URL=http://localhost:8000/api
VITE_APP_NAME=Gestión de Tareas
```

### Build para producción
```bash
npm run build
```

### Preview del build
```bash
npm run preview
```

### Linting del código
```bash
npm run lint
```

## 🎨 Personalización

### Cambiar colores
Modifica las variables CSS en `App.vue`:
```css
:root {
    --primary-color: #3b82f6;
    --success-color: #10b981;
    --danger-color: #ef4444;
    --warning-color: #f59e0b;
}
```

### Agregar nuevas funcionalidades
1. **Agrega nuevos métodos** en el componente
2. **Crea nuevos estados** en `data()`
3. **Agrega nuevas rutas API** en el backend
4. **Actualiza la UI** con nuevos elementos

## 📱 Responsive Design

### Breakpoints
- **Móvil:** < 768px
- **Tablet:** 768px - 1024px
- **Desktop:** > 1024px

### Adaptaciones
- **Cards apilados** en móvil
- **Modales fullscreen** en pantallas pequeñas
- **Botones táctiles** optimizados
- **Texto escalable** para accesibilidad

## 🚀 Despliegue

### Build de producción
```bash
npm run build
```

### Desplegar en servidor web
1. Copia los archivos de la carpeta `dist/`
2. Configura el servidor para servir archivos estáticos
3. Configura rutas para SPA (fallback a index.html)

### Configuración de Nginx
```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Configuración de Apache
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteRule ^index\.html$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.html [L]
</IfModule>
```

## 📞 Soporte

Si encuentras problemas:

1. **Revisa la consola** del navegador (F12)
2. **Verifica Network** para peticiones fallidas
3. **Limpia cache** del navegador
4. **Reinicia el servidor** de desarrollo
5. **Verifica que el backend** esté funcionando

## 🎉 Características Técnicas

### Tecnologías utilizadas
- **Vue 3** con Composition API
- **Vite** para build rápido
- **Axios** para peticiones HTTP
- **CSS3** con variables y flexbox
- **ES6+** con sintaxis moderna

### Optimizaciones
- **Lazy loading** de componentes
- **Debouncing** en búsquedas
- **Paginación** eficiente
- **Cache local** de datos
- **Minificación** automática en build

---

**Frontend Vue 3 listo para usar 🚀**
