# 5.2.3 Configuración del Sistema (Config)

Contiene todos los archivos de configuración del sistema, incluyendo base de datos, autenticación, servicios, caché, sesiones y otras configuraciones específicas del framework Laravel.

## 📁 Estructura de Configuración

```
├── 📄 app.php - Configuración principal de la aplicación
├── 📄 auth.php - Configuración de autenticación y guards
├── 📄 broadcasting.php - Configuración de broadcasting en tiempo real
├── 📄 cache.php - Configuración de caché y drivers
├── 📄 database.php - Configuración de conexiones de base de datos
├── 📄 filesystems.php - Configuración de sistemas de archivos
├── 📄 logging.php - Configuración de logs y canales
├── 📄 mail.php - Configuración de correo electrónico
├── 📄 permission.php - Configuración de roles y permisos (Spatie)
├── 📄 queue.php - Configuración de colas de trabajos
├── 📄 reverb.php - Configuración del servidor WebSocket (Laravel Reverb)
├── 📄 sanctum.php - Configuración de autenticación API (Laravel Sanctum)
├── 📄 services.php - Configuración de servicios de terceros
└── 📄 session.php - Configuración de sesiones de usuario
```

---

## ⚙️ Configuraciones Principales

### 📄 `config/app.php`
```php
<?php

return [
    'name' => env('APP_NAME', 'Fábrica Biodegradable'),
    'env' => env('APP_ENV', 'production'),
    'debug' => (bool) env('APP_DEBUG', false),
    'url' => env('APP_URL', 'http://localhost'),
    
    // Configuración de localización
    'locale' => env('APP_LOCALE', 'es'),
    'fallback_locale' => env('APP_FALLBACK_LOCALE', 'es'),
    'faker_locale' => env('APP_FAKER_LOCALE', 'es_BO'),
    
    // Configuración de seguridad
    'cipher' => 'AES-256-CBC',
    'key' => env('APP_KEY'),
    
    // Configuración de mantenimiento
    'maintenance' => [
        'driver' => env('APP_MAINTENANCE_DRIVER', 'file'),
    ],
    
    // Proveedores de servicios
    'providers' => [
        // Proveedores de Laravel
        Illuminate\Auth\AuthServiceProvider::class,
        Illuminate\Broadcasting\BroadcastServiceProvider::class,
        Illuminate\Bus\BusServiceProvider::class,
        Illuminate\Cache\CacheServiceProvider::class,
        // Proveedores de terceros
        Inertia\ServiceProvider::class,
        Laravel\Sanctum\SanctumServiceProvider::class,
        Spatie\Permission\PermissionServiceProvider::class,
        // Proveedores de la aplicación
        App\Providers\AppServiceProvider::class,
    ],
];
```

### 📄 `config/database.php`
```php
<?php

return [
    'default' => env('DB_CONNECTION', 'mysql'),
    
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', '127.0.0.1'),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', 'fabrica_biodegradable'),
            'username' => env('DB_USERNAME', 'root'),
            'password' => env('DB_PASSWORD', ''),
            'charset' => env('DB_CHARSET', 'utf8mb4'),
            'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
            'prefix' => '',
            'strict' => true,
            'engine' => null,
            'options' => extension_loaded('pdo_mysql') ? array_filter([
                PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            ]) : [],
        ],
    ],
    
    'migrations' => 'migrations',
    'redis' => [
        'client' => env('REDIS_CLIENT', 'phpredis'),
        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],
    ],
];
```

---

## 🔐 Configuraciones de Seguridad

### 📄 `config/auth.php`
```php
<?php

return [
    'defaults' => [
        'guard' => env('AUTH_GUARD', 'web'),
        'passwords' => env('AUTH_PASSWORD_BROKER', 'users'),
    ],
    
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'sanctum',
            'provider' => 'users',
        ],
    ],
    
    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => env('AUTH_MODEL', App\Models\User::class),
        ],
    ],
    
    'passwords' => [
        'users' => [
            'provider' => 'users',
            'table' => env('AUTH_PASSWORD_RESET_TOKEN_TABLE', 'password_reset_tokens'),
            'expire' => 60,
            'throttle' => 60,
        ],
    ],
    
    'password_timeout' => env('AUTH_PASSWORD_TIMEOUT', 10800),
];
```

### 📄 `config/sanctum.php`
```php
<?php

return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
        '%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        Laravel\Sanctum\Sanctum::currentApplicationUrlWithPort()
    ))),
    
    'guard' => ['web'],
    'expiration' => null,
    'token_prefix' => env('SANCTUM_TOKEN_PREFIX', ''),
    
    'middleware' => [
        'authenticate_session' => Laravel\Sanctum\Http\Middleware\AuthenticateSession::class,
        'encrypt_cookies' => Illuminate\Cookie\Middleware\EncryptCookies::class,
        'validate_csrf_token' => Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
    ],
];
```

### 📄 `config/permission.php`
```php
<?php

return [
    'models' => [
        'permission' => Spatie\Permission\Models\Permission::class,
        'role' => Spatie\Permission\Models\Role::class,
    ],
    
    'table_names' => [
        'roles' => 'roles',
        'permissions' => 'permissions',
        'model_has_permissions' => 'model_has_permissions',
        'model_has_roles' => 'model_has_roles',
        'role_has_permissions' => 'role_has_permissions',
    ],
    
    'column_names' => [
        'role_pivot_key' => null,
        'permission_pivot_key' => null,
        'model_morph_key' => 'model_id',
        'team_foreign_key' => 'team_id',
    ],
    
    'register_permission_check_method' => true,
    'register_octane_reset_listener' => false,
    'teams' => false,
    'use_passport_client_credentials' => false,
    'display_permission_in_exception' => false,
    'display_role_in_exception' => false,
    'enable_wildcard_permission' => false,
    
    'cache' => [
        'expiration_time' => \DateInterval::createFromDateString('24 hours'),
        'key' => 'spatie.permission.cache',
        'store' => 'default',
    ],
];
```

---

## 📡 Configuraciones de Comunicación

### 📄 `config/broadcasting.php`
```php
<?php

return [
    'default' => env('BROADCAST_CONNECTION', 'reverb'),
    
    'connections' => [
        'reverb' => [
            'driver' => 'reverb',
            'key' => env('REVERB_APP_KEY'),
            'secret' => env('REVERB_APP_SECRET'),
            'app_id' => env('REVERB_APP_ID'),
            'options' => [
                'host' => env('REVERB_HOST'),
                'port' => env('REVERB_PORT', 443),
                'scheme' => env('REVERB_SCHEME', 'https'),
                'useTLS' => env('REVERB_SCHEME', 'https') === 'https',
            ],
            'client_options' => [],
        ],
        
        'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'cluster' => env('PUSHER_APP_CLUSTER'),
                'host' => env('PUSHER_HOST'),
                'port' => env('PUSHER_PORT', 443),
                'scheme' => env('PUSHER_SCHEME', 'https'),
                'encrypted' => true,
                'useTLS' => env('PUSHER_SCHEME', 'https') === 'https',
            ],
        ],
    ],
];
```

### 📄 `config/reverb.php`
```php
<?php

return [
    'default' => env('REVERB_SERVER', 'reverb'),
    
    'servers' => [
        'reverb' => [
            'host' => env('REVERB_HOST', '127.0.0.1'),
            'port' => env('REVERB_PORT', 8080),
            'hostname' => env('REVERB_HOSTNAME'),
            'options' => [
                'tls' => [],
            ],
            'max_request_size' => env('REVERB_MAX_REQUEST_SIZE', 10_000),
            'scaling' => [
                'enabled' => env('REVERB_SCALING_ENABLED', false),
                'channel' => env('REVERB_SCALING_CHANNEL', 'reverb'),
                'server' => [
                    'url' => env('REDIS_URL'),
                    'host' => env('REDIS_HOST', '127.0.0.1'),
                    'port' => env('REDIS_PORT', 6379),
                    'username' => env('REDIS_USERNAME'),
                    'password' => env('REDIS_PASSWORD'),
                ],
            ],
            'pulse_ingest_interval' => env('REVERB_PULSE_INGEST_INTERVAL', 15),
            'telescope_ingest_interval' => env('REVERB_TELESCOPE_INGEST_INTERVAL', 15),
        ],
    ],
    
    'apps' => [
        'provider' => 'config',
        'apps' => [
            [
                'key' => env('REVERB_APP_KEY'),
                'secret' => env('REVERB_APP_SECRET'),
                'app_id' => env('REVERB_APP_ID'),
                'options' => [
                    'host' => env('REVERB_HOST'),
                    'port' => env('REVERB_PORT', 443),
                    'scheme' => env('REVERB_SCHEME', 'https'),
                ],
                'allowed_origins' => ['*'],
                'ping_interval' => env('REVERB_APP_PING_INTERVAL', 60),
                'max_message_size' => env('REVERB_APP_MAX_MESSAGE_SIZE', 10_000),
            ],
        ],
    ],
];
```

---

## 🗄️ Configuraciones de Almacenamiento

### 📄 `config/cache.php`
```php
<?php

return [
    'default' => env('CACHE_STORE', 'database'),
    
    'stores' => [
        'database' => [
            'driver' => 'database',
            'table' => env('CACHE_TABLE', 'cache'),
            'connection' => null,
            'lock_connection' => null,
        ],
        
        'file' => [
            'driver' => 'file',
            'path' => storage_path('framework/cache/data'),
            'lock_path' => storage_path('framework/cache/data'),
        ],
        
        'redis' => [
            'driver' => 'redis',
            'connection' => env('CACHE_REDIS_CONNECTION', 'cache'),
            'lock_connection' => env('CACHE_REDIS_LOCK_CONNECTION', 'default'),
        ],
    ],
    
    'prefix' => env('CACHE_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_cache_'),
];
```

### 📄 `config/filesystems.php`
```php
<?php

return [
    'default' => env('FILESYSTEM_DISK', 'local'),
    
    'disks' => [
        'local' => [
            'driver' => 'local',
            'root' => storage_path('app'),
            'throw' => false,
        ],
        
        'public' => [
            'driver' => 'local',
            'root' => storage_path('app/public'),
            'url' => env('APP_URL').'/storage',
            'visibility' => 'public',
            'throw' => false,
        ],
        
        's3' => [
            'driver' => 's3',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('AWS_BUCKET'),
            'url' => env('AWS_URL'),
            'endpoint' => env('AWS_ENDPOINT'),
            'use_path_style_endpoint' => env('AWS_USE_PATH_STYLE_ENDPOINT', false),
            'throw' => false,
        ],
    ],
    
    'links' => [
        public_path('storage') => storage_path('app/public'),
    ],
];
```

---

## 📊 Configuraciones de Monitoreo

### 📄 `config/logging.php`
```php
<?php

return [
    'default' => env('LOG_CHANNEL', 'stack'),
    
    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => explode(',', env('LOG_STACK', 'single')),
            'ignore_exceptions' => false,
        ],
        
        'single' => [
            'driver' => 'single',
            'path' => storage_path('logs/laravel.log'),
            'level' => env('LOG_LEVEL', 'debug'),
            'replace_placeholders' => true,
        ],
        
        'daily' => [
            'driver' => 'daily',
            'path' => storage_path('logs/laravel.log'),
            'level' => env('LOG_LEVEL', 'debug'),
            'days' => env('LOG_DAILY_DAYS', 14),
            'replace_placeholders' => true,
        ],
        
        'stderr' => [
            'driver' => 'monolog',
            'level' => env('LOG_LEVEL', 'debug'),
            'handler' => Monolog\Handler\StreamHandler::class,
            'formatter' => env('LOG_STDERR_FORMATTER'),
            'with' => [
                'stream' => 'php://stderr',
            ],
        ],
        
        'produccion' => [
            'driver' => 'daily',
            'path' => storage_path('logs/produccion.log'),
            'level' => 'info',
            'days' => 30,
        ],
        
        'maquinas' => [
            'driver' => 'daily',
            'path' => storage_path('logs/maquinas.log'),
            'level' => 'info',
            'days' => 30,
        ],
    ],
];
```

### 📄 `config/queue.php`
```php
<?php

return [
    'default' => env('QUEUE_CONNECTION', 'database'),
    
    'connections' => [
        'database' => [
            'driver' => 'database',
            'connection' => env('DB_QUEUE_CONNECTION'),
            'table' => env('DB_QUEUE_TABLE', 'jobs'),
            'queue' => env('DB_QUEUE', 'default'),
            'retry_after' => env('DB_QUEUE_RETRY_AFTER', 90),
            'after_commit' => false,
        ],
        
        'redis' => [
            'driver' => 'redis',
            'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
            'block_for' => null,
            'after_commit' => false,
        ],
    ],
    
    'batching' => [
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'job_batches',
    ],
    
    'failed' => [
        'driver' => env('QUEUE_FAILED_DRIVER', 'database-uuids'),
        'database' => env('DB_CONNECTION', 'mysql'),
        'table' => 'failed_jobs',
    ],
];
```

---

## 📧 Configuraciones de Comunicación Externa

### 📄 `config/mail.php`
```php
<?php

return [
    'default' => env('MAIL_MAILER', 'log'),
    
    'mailers' => [
        'smtp' => [
            'transport' => 'smtp',
            'url' => env('MAIL_URL'),
            'host' => env('MAIL_HOST', '127.0.0.1'),
            'port' => env('MAIL_PORT', 2525),
            'encryption' => env('MAIL_ENCRYPTION', 'tls'),
            'username' => env('MAIL_USERNAME'),
            'password' => env('MAIL_PASSWORD'),
            'timeout' => null,
            'local_domain' => env('MAIL_EHLO_DOMAIN', parse_url(env('APP_URL', 'http://localhost'), PHP_URL_HOST)),
        ],
        
        'log' => [
            'transport' => 'log',
            'channel' => env('MAIL_LOG_CHANNEL'),
        ],
    ],
    
    'from' => [
        'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
        'name' => env('MAIL_FROM_NAME', env('APP_NAME')),
    ],
];
```

### 📄 `config/services.php`
```php
<?php

return [
    'postmark' => [
        'key' => env('POSTMARK_API_KEY'),
    ],
    
    'resend' => [
        'key' => env('RESEND_API_KEY'),
    ],
    
    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],
    
    'slack' => [
        'notifications' => [
            'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
            'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
        ],
    ],
    
    // Servicios específicos del proyecto
    'simulacion' => [
        'enabled' => env('SIMULACION_ENABLED', true),
        'interval' => env('SIMULACION_INTERVAL', 5000), // milisegundos
        'max_maquinas' => env('SIMULACION_MAX_MAQUINAS', 10),
    ],
    
    'monitoring' => [
        'enabled' => env('MONITORING_ENABLED', true),
        'webhook_url' => env('MONITORING_WEBHOOK_URL'),
        'alert_threshold_oee' => env('ALERT_THRESHOLD_OEE', 60),
    ],
];
```

---

## 🔧 Variables de Entorno

### 📄 `.env` - Configuración de Desarrollo
```env
# Aplicación
APP_NAME="Fábrica Biodegradable"
APP_ENV=local
APP_KEY=base64:CLAVE_GENERADA_AUTOMATICAMENTE
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000
APP_LOCALE=es
APP_FALLBACK_LOCALE=es
APP_FAKER_LOCALE=es_BO

# Base de datos
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=fabrica_biodegradable
DB_USERNAME=root
DB_PASSWORD=

# Sesiones
SESSION_DRIVER=database
SESSION_LIFETIME=120
SESSION_ENCRYPT=false

# Caché
CACHE_STORE=database

# Colas
QUEUE_CONNECTION=database

# Broadcasting (Tiempo Real)
BROADCAST_CONNECTION=reverb
REVERB_APP_ID=347819
REVERB_APP_KEY=3aymhakmwulvttdk1ha9
REVERB_APP_SECRET=o16ydqnhriaf8pbi9kzu
REVERB_HOST=127.0.0.1
REVERB_PORT=8081
REVERB_SCHEME=http

# Logging
LOG_CHANNEL=stack
LOG_STACK=single
LOG_LEVEL=debug

# Autenticación
AUTH_GUARD=web
SANCTUM_STATEFUL_DOMAINS=localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1

# Servicios específicos
SIMULACION_ENABLED=true
SIMULACION_INTERVAL=5000
MONITORING_ENABLED=true
ALERT_THRESHOLD_OEE=60
```

---

## 🚀 Configuraciones de Producción

### **Diferencias en Producción**
```env
# Seguridad
APP_ENV=production
APP_DEBUG=false
SESSION_SECURE_COOKIE=true
SESSION_ENCRYPT=true

# Performance
CACHE_STORE=redis
QUEUE_CONNECTION=redis
SESSION_DRIVER=redis

# Logging
LOG_CHANNEL=daily
LOG_LEVEL=error

# Broadcasting
REVERB_SCHEME=https
REVERB_PORT=443
```

---

*Configuraciones centralizadas que permiten la personalización y optimización del sistema según el entorno de ejecución (desarrollo, pruebas, producción).*
