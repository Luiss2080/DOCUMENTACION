# 5.2.6 Archivos Públicos (Public)

Contiene los archivos accesibles públicamente del sistema, incluyendo el punto de entrada principal, configuraciones del servidor web y archivos estáticos del sistema de fábrica biodegradable.

## 📁 Estructura de Archivos Públicos

```
├── 📄 index.php - Punto de entrada principal de la aplicación
├── 📄 robots.txt - Configuración para robots de búsqueda
├── 📄 .htaccess - Configuración del servidor Apache
├── 📄 favicon.ico - Icono del sitio web
├── 📄 manifest.json - Configuración para PWA (Progressive Web App)
├── 📁 build/ - Archivos compilados por Vite (CSS/JS)
│   ├── 📄 manifest.json - Manifiesto de assets compilados
│   ├── 📁 assets/
│   │   ├── 📄 app-[hash].css - Estilos compilados
│   │   ├── 📄 app-[hash].js - JavaScript compilado
│   │   └── 📄 [varios]-[hash].[ext] - Assets con hash de cache
│   └── 📁 images/ - Imágenes optimizadas
├── 📁 storage/ - Enlace simbólico a storage/app/public
│   ├── 📁 maquinas/ - Fotos de máquinas subidas
│   ├── 📁 usuarios/ - Fotos de perfil de usuarios
│   ├── 📁 reportes/ - Reportes generados
│   └── 📁 documentos/ - Documentos del sistema
├── 📁 images/ - Imágenes estáticas del sistema
│   ├── 📄 logo.png - Logo principal
│   ├── 📄 logo-white.png - Logo en blanco
│   ├── 📄 placeholder.png - Imagen placeholder
│   └── 📁 icons/ - Iconos del sistema
└── 📁 docs/ - Documentación pública (opcional)
```

---

## 🚀 Punto de Entrada Principal

### 📄 `public/index.php`
```php
<?php

/*
|--------------------------------------------------------------------------
| Punto de Entrada de la Aplicación
|--------------------------------------------------------------------------
|
| Este archivo es el punto de entrada para todas las solicitudes HTTP
| que llegan a la aplicación de fábrica biodegradable. Configura el
| autoloader de Composer, inicia Laravel y maneja la solicitud.
|
*/

use Illuminate\Foundation\Application;
use Illuminate\Http\Request;

// Marcar tiempo de inicio para métricas de performance
define('LARAVEL_START', microtime(true));

// Verificar modo de mantenimiento
if (file_exists($maintenance = __DIR__.'/../storage/framework/maintenance.php')) {
    require $maintenance;
}

// Registrar el autoloader de Composer
require __DIR__.'/../vendor/autoload.php';

// Inicializar Laravel y manejar la solicitud
/** @var Application $app */
$app = require_once __DIR__.'/../bootstrap/app.php';

$app->handleRequest(Request::capture());
```

---

## 🌐 Configuración del Servidor Web

### 📄 `public/.htaccess`
```apache
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews -Indexes
    </IfModule>

    RewriteEngine On

    # ===== HEADERS DE SEGURIDAD =====
    # Manejo de headers de autorización
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    # Manejo de tokens XSRF para protección CSRF
    RewriteCond %{HTTP:x-xsrf-token} .
    RewriteRule .* - [E=HTTP_X_XSRF_TOKEN:%{HTTP:X-XSRF-Token}]

    # ===== REDIRECCIONES =====
    # Redireccionar barras finales si no es una carpeta
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} (.+)/$
    RewriteRule ^ %1 [L,R=301]

    # Enviar solicitudes al controlador frontal
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>

# ===== CONFIGURACIÓN DE CACHÉ =====
<IfModule mod_expires.c>
    ExpiresActive On
    
    # Imágenes - 1 año
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType image/svg+xml "access plus 1 year"
    
    # CSS y JavaScript - 1 mes
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    
    # Fonts - 1 año
    ExpiresByType font/woff "access plus 1 year"
    ExpiresByType font/woff2 "access plus 1 year"
    ExpiresByType application/font-woff "access plus 1 year"
    ExpiresByType application/font-woff2 "access plus 1 year"
</IfModule>

# ===== COMPRESIÓN GZIP =====
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE text/html
    AddOutputFilterByType DEFLATE text/xml
    AddOutputFilterByType DEFLATE text/css
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/xhtml+xml
    AddOutputFilterByType DEFLATE application/rss+xml
    AddOutputFilterByType DEFLATE application/javascript
    AddOutputFilterByType DEFLATE application/x-javascript
</IfModule>

# ===== HEADERS DE SEGURIDAD =====
<IfModule mod_headers.c>
    # Prevenir clickjacking
    Header always append X-Frame-Options SAMEORIGIN
    
    # XSS Protection
    Header set X-XSS-Protection "1; mode=block"
    
    # Content Type Options
    Header set X-Content-Type-Options nosniff
    
    # Referrer Policy
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    
    # Content Security Policy (ajustar según necesidades)
    Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' ws: wss:; font-src 'self' https://fonts.gstatic.com; frame-src 'none';"
</IfModule>

# ===== CONFIGURACIÓN ESPECÍFICA PARA PRODUCCIÓN =====
# Ocultar información del servidor
ServerTokens Prod
Header unset Server
Header unset X-Powered-By

# Bloquear acceso a archivos sensibles
<FilesMatch "^\.">
    Order allow,deny
    Deny from all
</FilesMatch>

<FilesMatch "\.(env|log|json)$">
    Order allow,deny
    Deny from all
</FilesMatch>
```

---

## 🤖 Configuración para Robots

### 📄 `public/robots.txt`
```txt
# Configuración de robots.txt para Fábrica Biodegradable
# Controla el acceso de los crawlers de motores de búsqueda

# Configuración para producción
User-agent: *
Disallow: /admin/
Disallow: /api/
Disallow: /storage/reportes/
Disallow: /storage/documentos/
Disallow: /dashboard/
Disallow: /maquinas/
Disallow: /produccion/
Disallow: /inventario/
Disallow: /mantenimiento/

# Permitir acceso a páginas públicas
Allow: /
Allow: /welcome
Allow: /storage/images/

# Archivos permitidos
Allow: /*.css$
Allow: /*.js$
Allow: /*.png$
Allow: /*.jpg$
Allow: /*.jpeg$
Allow: /*.gif$
Allow: /*.svg$

# Especificar sitemap (cuando esté disponible)
# Sitemap: https://tudominio.com/sitemap.xml

# Configuración específica para desarrollo
# User-agent: *
# Disallow: /

# Tiempo de rastreo (10 segundos entre solicitudes)
Crawl-delay: 10
```

---

## 📱 Configuración PWA

### 📄 `public/manifest.json`
```json
{
  "name": "Fábrica Biodegradable - Sistema de Monitoreo",
  "short_name": "Fábrica Bio",
  "description": "Sistema de monitoreo y control en tiempo real para fábrica de productos biodegradables",
  "start_url": "/dashboard",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#22c55e",
  "orientation": "portrait",
  "scope": "/",
  "lang": "es",
  
  "icons": [
    {
      "src": "/images/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/images/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ],
  
  "categories": ["productivity", "business", "utilities"],
  "screenshots": [
    {
      "src": "/images/screenshots/dashboard.png",
      "sizes": "1280x720",
      "type": "image/png",
      "label": "Dashboard principal del sistema"
    },
    {
      "src": "/images/screenshots/monitor.png",
      "sizes": "1280x720",
      "type": "image/png",
      "label": "Monitor de máquinas en tiempo real"
    }
  ],
  
  "shortcuts": [
    {
      "name": "Dashboard",
      "short_name": "Panel",
      "description": "Ir al dashboard principal",
      "url": "/dashboard",
      "icons": [
        {
          "src": "/images/icons/dashboard-96x96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Monitor Planta",
      "short_name": "Monitor",
      "description": "Ver monitor de máquinas",
      "url": "/planta/monitor-maquina",
      "icons": [
        {
          "src": "/images/icons/monitor-96x96.png",
          "sizes": "96x96"
        }
      ]
    },
    {
      "name": "Máquinas",
      "short_name": "Máquinas",
      "description": "Gestionar máquinas",
      "url": "/maquinas",
      "icons": [
        {
          "src": "/images/icons/machines-96x96.png",
          "sizes": "96x96"
        }
      ]
    }
  ],
  
  "related_applications": [],
  "prefer_related_applications": false
}
```

---

## 🎨 Assets Compilados (Build)

### 📄 `public/build/manifest.json` (Ejemplo)
```json
{
  "resources/css/app.css": {
    "file": "assets/app-7c2d0c84.css",
    "src": "resources/css/app.css",
    "isEntry": true
  },
  "resources/js/app.js": {
    "file": "assets/app-4ed993c7.js",
    "src": "resources/js/app.js",
    "isEntry": true,
    "imports": [
      "_app-b3e8e587.js"
    ],
    "css": [
      "assets/app-7c2d0c84.css"
    ]
  },
  "resources/js/Components/ApplicationLogo.vue": {
    "file": "assets/ApplicationLogo-8e9b4f21.js",
    "src": "resources/js/Components/ApplicationLogo.vue"
  },
  "resources/js/Layouts/AppLayout.vue": {
    "file": "assets/AppLayout-b8c4f3d5.js",
    "src": "resources/js/Layouts/AppLayout.vue",
    "imports": [
      "_app-b3e8e587.js",
      "resources/js/Components/ApplicationLogo.vue"
    ]
  },
  "resources/js/Pages/Dashboard.vue": {
    "file": "assets/Dashboard-f1e6b2a3.js",
    "src": "resources/js/Pages/Dashboard.vue",
    "imports": [
      "_app-b3e8e587.js",
      "resources/js/Layouts/AppLayout.vue"
    ]
  },
  "_app-b3e8e587.js": {
    "file": "assets/app-b3e8e587.js"
  }
}
```

---

## 🖼️ Estructura de Imágenes

### **Directorio `public/images/`**
```
images/
├── logo.png (512x512) - Logo principal en color
├── logo-white.png (512x512) - Logo en blanco para fondos oscuros
├── logo-small.png (64x64) - Logo pequeño para favicon
├── placeholder.png (400x300) - Imagen por defecto
├── banner-welcome.jpg (1920x1080) - Banner de la página de bienvenida
├── 📁 icons/
│   ├── icon-72x72.png - Icono PWA 72x72
│   ├── icon-96x96.png - Icono PWA 96x96
│   ├── icon-128x128.png - Icono PWA 128x128
│   ├── icon-144x144.png - Icono PWA 144x144
│   ├── icon-152x152.png - Icono PWA 152x152
│   ├── icon-192x192.png - Icono PWA 192x192
│   ├── icon-384x384.png - Icono PWA 384x384
│   ├── icon-512x512.png - Icono PWA 512x512
│   ├── dashboard-96x96.png - Icono de dashboard
│   ├── monitor-96x96.png - Icono de monitor
│   └── machines-96x96.png - Icono de máquinas
├── 📁 screenshots/
│   ├── dashboard.png - Screenshot del dashboard
│   ├── monitor.png - Screenshot del monitor
│   └── maquinas.png - Screenshot de gestión de máquinas
└── 📁 backgrounds/
    ├── factory-bg.jpg - Fondo de fábrica
    ├── green-pattern.svg - Patrón verde corporativo
    └── texture-metal.jpg - Textura metálica
```

---

## 📂 Enlace Simbólico de Storage

### **Configuración del Storage Link**
```bash
# Crear enlace simbólico desde storage/app/public a public/storage
php artisan storage:link
```

### **Estructura de `public/storage/`**
```
storage/ -> ../storage/app/public/
├── 📁 maquinas/
│   ├── maquina_001_foto.jpg
│   ├── maquina_002_foto.png
│   └── ...
├── 📁 usuarios/
│   ├── perfil_user_1.jpg
│   ├── perfil_user_2.png
│   └── ...
├── 📁 reportes/
│   ├── reporte_produccion_2025_01.pdf
│   ├── reporte_eficiencia_2025_01.xlsx
│   └── ...
├── 📁 documentos/
│   ├── manual_maquina_001.pdf
│   ├── certificados/
│   └── manuales/
└── 📁 temp/
    ├── uploads_temporales/
    └── cache_images/
```

---

## ⚡ Optimizaciones de Performance

### **Headers de Cache**
```apache
# En .htaccess - Configuración agresiva de cache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresDefault "access plus 1 month"
    
    # Assets con hash - cache muy largo
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType application/javascript "access plus 1 year"
    
    # Imágenes - cache largo
    ExpiresByType image/* "access plus 6 months"
    
    # HTML - sin cache para contenido dinámico
    ExpiresByType text/html "access plus 0 seconds"
</IfModule>
```

### **Compresión y Minificación**
```javascript
// vite.config.js - Configuración de Vite para optimización
export default defineConfig({
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    vendor: ['vue', '@inertiajs/vue3'],
                    charts: ['chart.js'],
                    utils: ['axios', 'lodash']
                }
            }
        },
        cssCodeSplit: true,
        sourcemap: false, // Desactivar en producción
        minify: 'terser',
        terserOptions: {
            compress: {
                drop_console: true, // Remover console.log
                drop_debugger: true
            }
        }
    }
});
```

---

## 🔒 Seguridad en Archivos Públicos

### **Archivos Protegidos**
```apache
# Denegar acceso a archivos sensibles
<FilesMatch "\.(env|log|ini|conf|htaccess|json)$">
    Order Allow,Deny
    Deny from all
</FilesMatch>

# Proteger directorios sensibles
RedirectMatch 403 ^/?(app|bootstrap|config|database|resources|storage|tests|vendor)/.*$
```

### **Validación de Uploads**
```php
// En controladores - validar archivos subidos
$request->validate([
    'foto' => 'required|image|mimes:jpeg,png,jpg|max:2048', // 2MB máximo
    'documento' => 'required|file|mimes:pdf,doc,docx|max:5120' // 5MB máximo
]);
```

---

*Configuración robusta de archivos públicos que garantiza la seguridad, performance y funcionalidad óptima del sistema de fábrica biodegradable.*
