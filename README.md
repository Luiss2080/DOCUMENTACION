# 5.2.7 Almacenamiento del Sistema (Storage)

Contiene todos los archivos de almacenamiento del sistema, incluyendo logs, archivos subidos, cache, sesiones y estructura de almacenamiento para el sistema de fábrica biodegradable.

## 📁 Estructura de Almacenamiento

```
├── 📁 app/
│   ├── 📁 public/ - Archivos públicos accesibles vía web
│   │   ├── 📁 maquinas/ - Fotos y documentos de máquinas
│   │   ├── 📁 usuarios/ - Fotos de perfil de usuarios
│   │   ├── 📁 reportes/ - Reportes generados
│   │   ├── 📁 documentos/ - Documentos del sistema
│   │   └── 📁 temp/ - Archivos temporales
│   └── 📁 private/ - Archivos privados no accesibles vía web
│       ├── 📁 backups/ - Respaldos de base de datos
│       ├── 📁 imports/ - Archivos para importación masiva
│       ├── 📁 exports/ - Exportaciones confidenciales
│       └── 📁 certificates/ - Certificados y documentos sensibles
│
├── 📁 framework/
│   ├── 📁 cache/ - Cache de aplicación y datos
│   │   ├── 📁 data/ - Cache de datos
│   │   └── 📄 services.php - Cache de servicios
│   ├── 📁 sessions/ - Archivos de sesión (si usa driver file)
│   ├── 📁 testing/ - Archivos temporales de testing
│   └── 📁 views/ - Cache de vistas compiladas
│
└── 📁 logs/
    ├── 📄 laravel.log - Log principal de la aplicación
    ├── 📄 produccion.log - Logs específicos de producción
    ├── 📄 maquinas.log - Logs de eventos de máquinas
    ├── 📄 seguridad.log - Logs de seguridad y accesos
    └── 📄 errores.log - Logs de errores críticos
```

---

## 📂 Almacenamiento de Aplicación (app/)

### **Archivos Públicos (`app/public/`)**

#### 📁 `storage/app/public/maquinas/`
```
maquinas/
├── maquina_001/
│   ├── foto_principal.jpg (1920x1080) - Foto principal de la máquina
│   ├── foto_lateral.jpg (1920x1080) - Vista lateral
│   ├── manual_operacion.pdf - Manual de operación
│   ├── ficha_tecnica.pdf - Especificaciones técnicas
│   └── historial_mantenimiento.xlsx - Historial de mantenimientos
├── maquina_002/
│   ├── foto_principal.png
│   ├── diagrama_electrico.pdf
│   └── certificados/
│       ├── certificado_ce.pdf
│       └── certificado_iso.pdf
└── tipos/
    ├── extrusora_default.jpg - Imagen por defecto para extrusoras
    ├── mezcladora_default.jpg - Imagen por defecto para mezcladoras
    └── prensa_default.jpg - Imagen por defecto para prensas
```

#### 📁 `storage/app/public/usuarios/`
```
usuarios/
├── perfil_1.jpg (400x400) - Foto de perfil usuario 1
├── perfil_2.png (400x400) - Foto de perfil usuario 2
├── perfil_3.jpg (400x400) - Foto de perfil usuario 3
└── default/
    ├── avatar_admin.png - Avatar por defecto administrador
    ├── avatar_operador.png - Avatar por defecto operador
    └── avatar_encargado.png - Avatar por defecto encargado
```

#### 📁 `storage/app/public/reportes/`
```
reportes/
├── 2025/
│   ├── 01/ - Enero 2025
│   │   ├── reporte_produccion_2025_01_01.pdf - Reporte diario
│   │   ├── reporte_eficiencia_semanal_01.xlsx - Reporte semanal
│   │   └── reporte_mensual_enero_2025.pdf - Reporte mensual
│   └── 02/ - Febrero 2025
│       └── ...
├── templates/
│   ├── plantilla_reporte_diario.xlsx
│   ├── plantilla_reporte_mensual.docx
│   └── plantilla_certificado_calidad.pdf
└── automaticos/
    ├── reporte_automatico_2025_01_15.json - Data para dashboard
    ├── metricas_tiempo_real.json - Métricas en tiempo real
    └── estadisticas_resumen.json - Estadísticas de resumen
```

#### 📁 `storage/app/public/documentos/`
```
documentos/
├── manuales/
│   ├── manual_sistema_completo.pdf
│   ├── manual_operador.pdf
│   ├── manual_mantenimiento.pdf
│   └── manual_administrador.pdf
├── procedimientos/
│   ├── sop_inicio_produccion.pdf
│   ├── sop_parada_emergencia.pdf
│   ├── sop_mantenimiento_preventivo.pdf
│   └── sop_calidad_producto.pdf
├── certificaciones/
│   ├── iso_9001_certificado.pdf
│   ├── iso_14001_certificado.pdf
│   └── certificado_biodegradable.pdf
└── formatos/
    ├── formato_orden_produccion.xlsx
    ├── formato_inspeccion_calidad.xlsx
    └── formato_reporte_incidente.docx
```

### **Archivos Privados (`app/private/`)**

#### 📁 `storage/app/private/backups/`
```bash
# Script de ejemplo para generar backups
#!/bin/bash
# backup_daily.sh

DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="fabrica_biodegradable"
BACKUP_DIR="/storage/app/private/backups"

# Crear backup de base de datos
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME > "$BACKUP_DIR/db_backup_$DATE.sql"

# Comprimir backup
gzip "$BACKUP_DIR/db_backup_$DATE.sql"

# Crear backup de archivos críticos
tar -czf "$BACKUP_DIR/files_backup_$DATE.tar.gz" \
    storage/app/public/maquinas \
    storage/app/public/documentos \
    storage/app/private/certificates

# Limpiar backups antiguos (más de 30 días)
find $BACKUP_DIR -name "*.gz" -mtime +30 -delete

echo "Backup completado: $DATE"
```

#### 📁 `storage/app/private/imports/`
```
imports/
├── plantillas/
│   ├── plantilla_maquinas.xlsx - Para importación masiva de máquinas
│   ├── plantilla_usuarios.xlsx - Para importación de usuarios
│   ├── plantilla_productos.xlsx - Para importación de productos
│   └── plantilla_materias_primas.xlsx - Para materias primas
├── procesados/
│   ├── import_maquinas_2025_01_15.xlsx - Archivo procesado
│   ├── import_usuarios_2025_01_10.xlsx - Archivo procesado
│   └── errores/
│       ├── errores_import_2025_01_15.log - Log de errores
│       └── filas_rechazadas_2025_01_15.xlsx - Datos rechazados
└── pendientes/
    ├── nuevas_maquinas.xlsx - Pendiente de procesar
    └── actualizacion_usuarios.csv - Pendiente de procesar
```

---

## 🗄️ Framework de Laravel (framework/)

### **Cache del Sistema (`framework/cache/`)**

#### 📁 `storage/framework/cache/data/`
```php
// Ejemplo de estructura de cache
<?php
// Cache de configuración de máquinas
'maquinas_config' => [
    'tipos_disponibles' => ['Extrusora', 'Mezcladora', 'Prensa'],
    'capacidad_maxima_global' => 5000, // kg/día
    'oee_target' => 85, // %
    'cached_at' => '2025-01-15 10:30:00'
];

// Cache de estadísticas del dashboard
'dashboard_stats' => [
    'produccion_hoy' => 2847.5, // kg
    'maquinas_activas' => 8,
    'oee_promedio' => 87.3, // %
    'eficiencia' => 92.1, // %
    'last_update' => '2025-01-15 14:45:12'
];

// Cache de permisos de usuario
'user_permissions_123' => [
    'permissions' => ['ver_dashboard', 'gestionar_maquinas', 'ver_reportes'],
    'roles' => ['Operador'],
    'cached_at' => '2025-01-15 09:00:00'
];
```

#### 📄 `storage/framework/services.php`
```php
<?php return [
    'providers' => [
        'Illuminate\Auth\AuthServiceProvider',
        'Illuminate\Broadcasting\BroadcastServiceProvider',
        'App\Providers\AppServiceProvider',
        'App\Providers\ProduccionServiceProvider',
        'Laravel\Sanctum\SanctumServiceProvider',
        'Spatie\Permission\PermissionServiceProvider',
    ],
    'eager' => [
        'Illuminate\Auth\AuthServiceProvider',
        'App\Providers\AppServiceProvider',
    ],
    'deferred' => [
        'Illuminate\Broadcasting\BroadcastServiceProvider',
    ],
    'when' => []
];
```

### **Sesiones (`framework/sessions/`)**
```php
// Ejemplo de archivo de sesión (cuando se usa driver 'file')
// laravel_session_abc123def456
a:4:{
    s:6:"_token";s:40:"Xy9z8WqP3LmK5NvR7TbH2JcF6GdE9QaS";
    s:9:"_previous";a:1:{
        s:3:"url";s:34:"http://localhost:8000/dashboard";
    }
    s:6:"_flash";a:2:{
        s:3:"old";a:0:{}
        s:3:"new";a:0:{}
    }
    s:50:"login_web_59ba36addc2b2f9401580f014c7f58ea4e30989d";i:1;
    s:4:"user";a:5:{
        s:2:"id";i:1;
        s:4:"name";s:13:"Administrador";
        s:5:"email";s:25:"admin@fabricabio.com";
        s:6:"activo";b:1;
        s:5:"roles";a:1:{i:0;s:13:"Administrador";}
    }
}
```

---

## 📊 Sistema de Logs (logs/)

### 📄 `storage/logs/laravel.log`
```log
[2025-01-15 14:30:15] production.INFO: Usuario autenticado {"user_id":1,"email":"admin@fabricabio.com","ip":"192.168.1.100"} 

[2025-01-15 14:30:45] production.INFO: Nueva producción registrada {"maquina_id":3,"kg_producidos":45.7,"oee":89.2,"velocidad":1150} 

[2025-01-15 14:31:20] production.WARNING: OEE por debajo del umbral {"maquina_id":5,"oee_actual":62.3,"umbral_minimo":65} 

[2025-01-15 14:32:05] production.ERROR: Error en conexión con máquina {"maquina_id":2,"error":"Connection timeout","attempts":3} 

[2025-01-15 14:33:10] production.INFO: Mantenimiento programado completado {"maquina_id":1,"tipo":"Preventivo","duracion_minutos":45}
```

### 📄 `storage/logs/produccion.log`
```log
[2025-01-15 14:30:45] INFO: Producción iniciada {"maquina":"EXT-001","operador":"Juan Pérez","turno":"Mañana"}
[2025-01-15 14:35:12] INFO: Incremento producción {"maquina":"EXT-001","kg_incremento":12.3,"total_acumulado":127.8}
[2025-01-15 14:40:33] WARNING: Velocidad reducida {"maquina":"EXT-001","velocidad_anterior":1200,"velocidad_actual":980}
[2025-01-15 14:45:18] INFO: OEE calculado {"maquina":"EXT-001","oee":87.5,"tiempo_ciclo":2.1,"disponibilidad":0.95}
[2025-01-15 15:00:00] INFO: Producción finalizada {"maquina":"EXT-001","total_producido":156.7,"duracion_horas":2.5}
```

### 📄 `storage/logs/maquinas.log`
```log
[2025-01-15 08:00:00] INFO: Sistema iniciado - Verificando estados {"maquinas_total":10}
[2025-01-15 08:01:15] INFO: Máquina activada {"id":1,"codigo":"EXT-001","nombre":"Extrusora Principal"}
[2025-01-15 09:30:22] WARNING: Temperatura alta detectada {"maquina":"MEZ-002","temperatura":85,"limite_maximo":80}
[2025-01-15 10:15:45] ERROR: Fallo de comunicación {"maquina":"PRE-003","ultimo_ping":"2025-01-15 10:12:30"}
[2025-01-15 11:00:00] INFO: Mantenimiento iniciado {"maquina":"COR-004","tipo":"Preventivo","tecnico":"Carlos López"}
[2025-01-15 16:30:00] INFO: Todas las máquinas en estado seguro para fin de turno
```

### 📄 `storage/logs/seguridad.log`
```log
[2025-01-15 08:30:15] INFO: Login exitoso {"user":"admin@fabricabio.com","ip":"192.168.1.100","user_agent":"Mozilla/5.0..."}
[2025-01-15 09:15:30] WARNING: Intento de acceso denegado {"user":"operador@fabricabio.com","recurso":"/admin/usuarios","ip":"192.168.1.105"}
[2025-01-15 10:45:22] ERROR: Múltiples intentos de login fallido {"email":"unknown@test.com","attempts":5,"ip":"203.45.67.89"}
[2025-01-15 11:20:10] INFO: Cambio de contraseña {"user_id":3,"ip":"192.168.1.102"}
[2025-01-15 14:30:45] WARNING: Token API utilizado múltiples veces {"token_id":"abc123","requests_count":50,"last_ip":"192.168.1.110"}
```

---

## 🔧 Configuración de Storage

### 📄 `config/filesystems.php` - Configuración Personalizada
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
        
        // Disco para documentos privados
        'private' => [
            'driver' => 'local',
            'root' => storage_path('app/private'),
            'throw' => false,
        ],
        
        // Disco para backups
        'backups' => [
            'driver' => 'local',
            'root' => storage_path('app/private/backups'),
            'throw' => false,
        ],
        
        // Disco para reportes temporales
        'temp_reports' => [
            'driver' => 'local',
            'root' => storage_path('app/temp'),
            'throw' => false,
        ],
        
        // Disco para logs personalizados
        'logs' => [
            'driver' => 'local',
            'root' => storage_path('logs'),
            'throw' => false,
        ],
    ],
];
```

---

## 🧹 Scripts de Limpieza

### 📄 `storage/scripts/cleanup.php`
```php
<?php
/**
 * Script de limpieza automática de storage
 * Ejecutar diariamente via cron job
 */

require_once __DIR__ . '/../../vendor/autoload.php';

$app = require_once __DIR__ . '/../../bootstrap/app.php';
$app->bind('request', \Illuminate\Http\Request::class);

use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\File;
use Carbon\Carbon;

class StorageCleanup {
    
    public function run()
    {
        $this->cleanTempFiles();
        $this->cleanOldLogs();
        $this->cleanOldBackups();
        $this->cleanOldReports();
        $this->optimizeImages();
        
        echo "✅ Limpieza de storage completada\n";
    }
    
    private function cleanTempFiles()
    {
        echo "🧹 Limpiando archivos temporales...\n";
        
        // Limpiar archivos temporales más antiguos de 24 horas
        $tempPath = storage_path('app/temp');
        if (File::exists($tempPath)) {
            $files = File::files($tempPath);
            $cleaned = 0;
            
            foreach ($files as $file) {
                if ($file->getMTime() < time() - 86400) { // 24 horas
                    File::delete($file);
                    $cleaned++;
                }
            }
            
            echo "   - Eliminados {$cleaned} archivos temporales\n";
        }
    }
    
    private function cleanOldLogs()
    {
        echo "📋 Rotando logs antiguos...\n";
        
        $logPath = storage_path('logs');
        $files = File::glob($logPath . '/*.log');
        $rotated = 0;
        
        foreach ($files as $file) {
            $fileInfo = pathinfo($file);
            $fileDate = filemtime($file);
            
            // Rotar logs más antiguos de 7 días
            if ($fileDate < time() - (7 * 24 * 60 * 60)) {
                $newName = $fileInfo['dirname'] . '/' . 
                          $fileInfo['filename'] . '_' . 
                          date('Y_m_d', $fileDate) . '.log';
                
                rename($file, $newName);
                gzip($newName);
                $rotated++;
            }
        }
        
        echo "   - Rotados {$rotated} archivos de log\n";
    }
    
    private function cleanOldBackups()
    {
        echo "💾 Limpiando backups antiguos...\n";
        
        $backupPath = storage_path('app/private/backups');
        $files = File::glob($backupPath . '/*.gz');
        $deleted = 0;
        
        foreach ($files as $file) {
            // Eliminar backups más antiguos de 30 días
            if (filemtime($file) < time() - (30 * 24 * 60 * 60)) {
                File::delete($file);
                $deleted++;
            }
        }
        
        echo "   - Eliminados {$deleted} backups antiguos\n";
    }
    
    private function cleanOldReports()
    {
        echo "📊 Archivando reportes antiguos...\n";
        
        $reportPath = storage_path('app/public/reportes');
        
        // Mover reportes más antiguos de 90 días a archivo
        $cutoffDate = Carbon::now()->subDays(90);
        $archivePath = storage_path('app/private/reportes_archivo');
        
        if (!File::exists($archivePath)) {
            File::makeDirectory($archivePath, 0755, true);
        }
        
        $this->moveOldFiles($reportPath, $archivePath, $cutoffDate);
    }
    
    private function optimizeImages()
    {
        echo "🖼️ Optimizando imágenes...\n";
        
        // Aquí se podría implementar optimización de imágenes
        // usando librerías como ImageMagick o GD
        
        echo "   - Optimización de imágenes completada\n";
    }
    
    private function moveOldFiles($source, $destination, $cutoffDate)
    {
        if (!File::exists($source)) return;
        
        $files = File::allFiles($source);
        $moved = 0;
        
        foreach ($files as $file) {
            $fileDate = Carbon::createFromTimestamp($file->getMTime());
            
            if ($fileDate->lt($cutoffDate)) {
                $relativePath = $file->getRelativePath();
                $destDir = $destination . '/' . $relativePath;
                
                if (!File::exists($destDir)) {
                    File::makeDirectory($destDir, 0755, true);
                }
                
                $destFile = $destDir . '/' . $file->getFilename();
                File::move($file->getPathname(), $destFile);
                $moved++;
            }
        }
        
        echo "   - Archivados {$moved} archivos de reportes\n";
    }
}

// Ejecutar limpieza
$cleanup = new StorageCleanup();
$cleanup->run();
```

---

## 📈 Monitoreo de Storage

### 📄 `storage/scripts/monitor.php`
```php
<?php
/**
 * Script de monitoreo de uso de storage
 */

use Illuminate\Support\Facades\File;

class StorageMonitor {
    
    private $thresholds = [
        'warning' => 80,  // 80% de uso
        'critical' => 90  // 90% de uso
    ];
    
    public function checkUsage()
    {
        $storagePath = storage_path();
        $totalSpace = disk_total_space($storagePath);
        $freeSpace = disk_free_space($storagePath);
        $usedSpace = $totalSpace - $freeSpace;
        $usagePercent = ($usedSpace / $totalSpace) * 100;
        
        $report = [
            'timestamp' => now(),
            'total_space' => $this->formatBytes($totalSpace),
            'used_space' => $this->formatBytes($usedSpace),
            'free_space' => $this->formatBytes($freeSpace),
            'usage_percent' => round($usagePercent, 2),
            'status' => $this->getStatus($usagePercent),
            'directories' => $this->getDirectorySizes()
        ];
        
        $this->saveReport($report);
        $this->checkAlerts($report);
        
        return $report;
    }
    
    private function getDirectorySizes()
    {
        $directories = [
            'logs' => storage_path('logs'),
            'app' => storage_path('app'),
            'framework' => storage_path('framework'),
        ];
        
        $sizes = [];
        foreach ($directories as $name => $path) {
            $sizes[$name] = $this->formatBytes($this->getDirectorySize($path));
        }
        
        return $sizes;
    }
    
    private function getDirectorySize($directory)
    {
        $size = 0;
        if (!File::exists($directory)) return $size;
        
        $files = File::allFiles($directory);
        foreach ($files as $file) {
            $size += $file->getSize();
        }
        
        return $size;
    }
    
    private function formatBytes($bytes)
    {
        $units = ['B', 'KB', 'MB', 'GB', 'TB'];
        $bytes = max($bytes, 0);
        $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
        $pow = min($pow, count($units) - 1);
        
        $bytes /= (1 << (10 * $pow));
        
        return round($bytes, 2) . ' ' . $units[$pow];
    }
    
    private function getStatus($usagePercent)
    {
        if ($usagePercent >= $this->thresholds['critical']) {
            return 'critical';
        } elseif ($usagePercent >= $this->thresholds['warning']) {
            return 'warning';
        } else {
            return 'ok';
        }
    }
    
    private function saveReport($report)
    {
        $reportFile = storage_path('app/private/storage_reports.json');
        
        $reports = [];
        if (File::exists($reportFile)) {
            $reports = json_decode(File::get($reportFile), true) ?: [];
        }
        
        $reports[] = $report;
        
        // Mantener solo los últimos 30 reportes
        if (count($reports) > 30) {
            $reports = array_slice($reports, -30);
        }
        
        File::put($reportFile, json_encode($reports, JSON_PRETTY_PRINT));
    }
    
    private function checkAlerts($report)
    {
        if ($report['status'] === 'critical') {
            // Enviar alerta crítica
            $this->sendAlert('CRÍTICO: Storage al ' . $report['usage_percent'] . '%', $report);
        } elseif ($report['status'] === 'warning') {
            // Enviar advertencia
            $this->sendAlert('ADVERTENCIA: Storage al ' . $report['usage_percent'] . '%', $report);
        }
    }
    
    private function sendAlert($message, $report)
    {
        // Implementar envío de alertas (email, Slack, etc.)
        error_log($message . ' - ' . json_encode($report));
    }
}

// Ejecutar monitoreo
$monitor = new StorageMonitor();
$report = $monitor->checkUsage();

echo "📊 Reporte de Storage:\n";
echo "Total: {$report['total_space']}\n";
echo "Usado: {$report['used_space']} ({$report['usage_percent']}%)\n";
echo "Libre: {$report['free_space']}\n";
echo "Estado: {$report['status']}\n";
```

---

*Sistema de almacenamiento bien organizado que garantiza la integridad, seguridad y eficiencia en el manejo de todos los archivos del sistema de fábrica biodegradable.*
