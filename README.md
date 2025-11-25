# 5.2.5 Rutas del Sistema (Routes)

Contiene la definición de todas las rutas del sistema, incluyendo rutas web, API, canales de broadcasting y comandos de consola para el sistema de fábrica biodegradable.

## 📁 Estructura de Rutas

```
├── 📄 web.php - Rutas web principales de la aplicación
├── 📄 api.php - Rutas de API REST para servicios externos
├── 📄 channels.php - Canales de broadcasting en tiempo real
└── 📄 console.php - Comandos de consola personalizados
```

---

## 🌐 Rutas Web (Interface de Usuario)

### 📄 `routes/web.php`
```php
<?php

use App\Http\Controllers\DashboardController;
use App\Http\Controllers\MaquinaController;
use App\Http\Controllers\WelcomeController;
use App\Http\Controllers\Planta\MonitorMaquinaController;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Rutas Web
|--------------------------------------------------------------------------
|
| Rutas para la interfaz web del sistema usando Inertia.js
| Todas las rutas retornan componentes Vue.js renderizados
|
*/

// Ruta raíz - Redirección a welcome
Route::get('/', function () {
    return redirect('/welcome');
});

// Página de bienvenida pública
Route::get('/welcome', [WelcomeController::class, 'index'])
    ->name('welcome');

// ===== RUTAS PROTEGIDAS POR AUTENTICACIÓN =====
Route::middleware(['auth', 'verified'])->group(function () {
    
    // Dashboard principal del sistema
    Route::get('/dashboard', [DashboardController::class, 'index'])
        ->name('dashboard');

    // ===== GESTIÓN DE MÁQUINAS =====
    Route::resource('maquinas', MaquinaController::class)->names([
        'index' => 'maquinas.index',         // GET /maquinas
        'create' => 'maquinas.create',       // GET /maquinas/create
        'store' => 'maquinas.store',         // POST /maquinas
        'show' => 'maquinas.show',           // GET /maquinas/{id}
        'edit' => 'maquinas.edit',           // GET /maquinas/{id}/edit
        'update' => 'maquinas.update',       // PUT/PATCH /maquinas/{id}
        'destroy' => 'maquinas.destroy'      // DELETE /maquinas/{id}
    ]);

    // ===== MÓDULO DE PLANTA =====
    Route::prefix('planta')->name('planta.')->group(function () {
        
        // Monitor de máquinas en tiempo real
        Route::get('/monitor-maquina', [MonitorMaquinaController::class, 'index'])
            ->name('monitor-maquina.index');
        
        Route::get('/monitor-maquina/{maquina}', [MonitorMaquinaController::class, 'show'])
            ->name('monitor-maquina.show');
    });

    // ===== GESTIÓN DE PRODUCCIÓN =====
    Route::prefix('produccion')->name('produccion.')->group(function () {
        Route::get('/', [ProduccionController::class, 'index'])->name('index');
        Route::get('/crear', [ProduccionController::class, 'create'])->name('create');
        Route::post('/', [ProduccionController::class, 'store'])->name('store');
        Route::get('/{produccion}', [ProduccionController::class, 'show'])->name('show');
    });

    // ===== GESTIÓN DE INVENTARIO =====
    Route::prefix('inventario')->name('inventario.')->group(function () {
        // Materias primas
        Route::resource('materias-primas', MateriaPrimaController::class);
        Route::resource('lotes-materia-prima', LoteMateriaPrimaController::class);
        
        // Productos
        Route::resource('productos', ProductoController::class);
        Route::resource('lotes-productos', LoteProductoController::class);
        
        // Proveedores
        Route::resource('proveedores', ProveedorController::class);
    });

    // ===== GESTIÓN DE MANTENIMIENTO =====
    Route::prefix('mantenimiento')->name('mantenimiento.')->group(function () {
        Route::resource('programados', MantenimientoController::class);
        Route::resource('paradas', ParadaController::class);
        
        // Reportes de mantenimiento
        Route::get('/reportes', [MantenimientoController::class, 'reportes'])->name('reportes');
        Route::get('/calendario', [MantenimientoController::class, 'calendario'])->name('calendario');
    });

    // ===== RECETAS Y FÓRMULAS =====
    Route::prefix('recetas')->name('recetas.')->group(function () {
        Route::resource('/', RecetaController::class)->parameters(['' => 'receta']);
        Route::get('/{receta}/detalles', [RecetaController::class, 'detalles'])->name('detalles');
        Route::post('/{receta}/duplicar', [RecetaController::class, 'duplicar'])->name('duplicar');
    });

    // ===== REPORTES Y ANALYTICS =====
    Route::prefix('reportes')->name('reportes.')->group(function () {
        Route::get('/', [ReporteController::class, 'index'])->name('index');
        Route::get('/produccion', [ReporteController::class, 'produccion'])->name('produccion');
        Route::get('/eficiencia', [ReporteController::class, 'eficiencia'])->name('eficiencia');
        Route::get('/calidad', [ReporteController::class, 'calidad'])->name('calidad');
        Route::get('/costos', [ReporteController::class, 'costos'])->name('costos');
        
        // Exportaciones
        Route::post('/exportar/{tipo}', [ReporteController::class, 'exportar'])->name('exportar');
    });

    // ===== ADMINISTRACIÓN DEL SISTEMA =====
    Route::prefix('admin')->name('admin.')->middleware(['role:Administrador'])->group(function () {
        // Gestión de usuarios
        Route::resource('usuarios', UsuarioController::class);
        Route::post('/usuarios/{usuario}/toggle-status', [UsuarioController::class, 'toggleStatus'])->name('usuarios.toggle-status');
        
        // Configuración del sistema
        Route::get('/configuracion', [ConfiguracionController::class, 'index'])->name('configuracion');
        Route::post('/configuracion', [ConfiguracionController::class, 'update'])->name('configuracion.update');
        
        // Logs del sistema
        Route::get('/logs', [LogController::class, 'index'])->name('logs');
        Route::get('/logs/{archivo}', [LogController::class, 'show'])->name('logs.show');
    });
});

// ===== RUTAS DE AUTENTICACIÓN =====
require __DIR__.'/auth.php';
```

---

## 🔌 Rutas API (Servicios REST)

### 📄 `routes/api.php`
```php
<?php

use App\Http\Controllers\Api\SimulacionController;
use App\Http\Controllers\Api\MaquinaEstadoController;
use App\Http\Controllers\Api\ProduccionApiController;
use App\Http\Controllers\Planta\MonitorMaquinaController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Rutas API
|--------------------------------------------------------------------------
|
| Rutas para servicios REST y integraciones externas
| Algunas rutas requieren autenticación con Laravel Sanctum
|
*/

// ===== RUTA DE USUARIO AUTENTICADO =====
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');

// ===== SIMULACIÓN DE PRODUCCIÓN (Sin autenticación) =====
Route::prefix('simulacion')->group(function () {
    // Endpoint principal para simuladores externos
    Route::post('/produccion', [SimulacionController::class, 'simularProduccion'])
        ->name('api.simulacion.produccion');
    
    // Configuración de simulación
    Route::get('/configuracion', [SimulacionController::class, 'configuracion'])
        ->name('api.simulacion.config');
    
    // Estado de simulación
    Route::get('/estado', [SimulacionController::class, 'estado'])
        ->name('api.simulacion.estado');
    
    // Detener simulación
    Route::post('/detener', [SimulacionController::class, 'detener'])
        ->name('api.simulacion.detener');
});

// ===== ESTADOS DE MÁQUINAS =====
Route::prefix('maquinas')->group(function () {
    // Estado actual de una máquina específica
    Route::get('/{maquina}/estado', [MonitorMaquinaController::class, 'getEstado'])
        ->name('api.maquina.estado');
    
    // Actualizar estado de máquina (Protegido)
    Route::put('/{maquina}/estado', [MaquinaEstadoController::class, 'updateEstado'])
        ->middleware('auth:sanctum')
        ->name('api.maquina.update-estado');
    
    // Configurar modo simulación
    Route::put('/{maquina}/simulacion', [MaquinaEstadoController::class, 'updateSimulacion'])
        ->middleware('auth:sanctum')
        ->name('api.maquina.simulacion');
    
    // Historial de estados
    Route::get('/{maquina}/historial', [MaquinaEstadoController::class, 'historial'])
        ->middleware('auth:sanctum')
        ->name('api.maquina.historial');
});

// ===== PRODUCCIÓN API (Protegidas) =====
Route::middleware('auth:sanctum')->prefix('produccion')->group(function () {
    // Estadísticas de producción
    Route::get('/estadisticas', [ProduccionApiController::class, 'estadisticas'])
        ->name('api.produccion.estadisticas');
    
    // Producción por período
    Route::get('/periodo', [ProduccionApiController::class, 'porPeriodo'])
        ->name('api.produccion.periodo');
    
    // OEE por máquina
    Route::get('/oee/{maquina}', [ProduccionApiController::class, 'oeeByMaquina'])
        ->name('api.produccion.oee');
    
    // Eficiencia general
    Route::get('/eficiencia', [ProduccionApiController::class, 'eficiencia'])
        ->name('api.produccion.eficiencia');
    
    // Crear nuevo registro de producción
    Route::post('/', [ProduccionApiController::class, 'store'])
        ->name('api.produccion.store');
});

// ===== INVENTARIO API (Protegidas) =====
Route::middleware('auth:sanctum')->prefix('inventario')->group(function () {
    // Stock de materias primas
    Route::get('/materias-primas/stock', [InventarioApiController::class, 'stockMateriasPrimas'])
        ->name('api.inventario.stock-mp');
    
    // Stock de productos
    Route::get('/productos/stock', [InventarioApiController::class, 'stockProductos'])
        ->name('api.inventario.stock-productos');
    
    // Movimientos de inventario
    Route::post('/movimientos', [InventarioApiController::class, 'registrarMovimiento'])
        ->name('api.inventario.movimiento');
    
    // Alertas de stock bajo
    Route::get('/alertas', [InventarioApiController::class, 'alertas'])
        ->name('api.inventario.alertas');
});

// ===== MANTENIMIENTO API (Protegidas) =====
Route::middleware('auth:sanctum')->prefix('mantenimiento')->group(function () {
    // Próximos mantenimientos
    Route::get('/proximos', [MantenimientoApiController::class, 'proximos'])
        ->name('api.mantenimiento.proximos');
    
    // Registrar parada de máquina
    Route::post('/paradas', [MantenimientoApiController::class, 'registrarParada'])
        ->name('api.mantenimiento.parada');
    
    // Finalizar mantenimiento
    Route::patch('/finalizar/{mantenimiento}', [MantenimientoApiController::class, 'finalizar'])
        ->name('api.mantenimiento.finalizar');
});

// ===== NOTIFICACIONES API (Protegidas) =====
Route::middleware('auth:sanctum')->prefix('notificaciones')->group(function () {
    // Obtener notificaciones del usuario
    Route::get('/', [NotificacionApiController::class, 'index'])
        ->name('api.notificaciones.index');
    
    // Marcar como leída
    Route::patch('/{notificacion}/leida', [NotificacionApiController::class, 'marcarLeida'])
        ->name('api.notificaciones.leida');
    
    // Marcar todas como leídas
    Route::patch('/todas/leidas', [NotificacionApiController::class, 'marcarTodasLeidas'])
        ->name('api.notificaciones.todas-leidas');
});

// ===== WEBHOOKS (Sin autenticación pero con validación) =====
Route::prefix('webhooks')->group(function () {
    // Webhook para sistemas externos
    Route::post('/produccion', [WebhookController::class, 'produccion'])
        ->middleware('webhook.signature')
        ->name('api.webhook.produccion');
    
    // Webhook para sensores IoT
    Route::post('/sensores', [WebhookController::class, 'sensores'])
        ->middleware('webhook.signature')
        ->name('api.webhook.sensores');
});

// ===== HEALTH CHECK =====
Route::get('/health', function () {
    return response()->json([
        'status' => 'ok',
        'timestamp' => now(),
        'version' => config('app.version', '1.0.0'),
        'environment' => config('app.env')
    ]);
})->name('api.health');

// ===== METRICS PARA MONITOREO =====
Route::get('/metrics', [MetricsController::class, 'prometheus'])
    ->middleware('metrics.auth')
    ->name('api.metrics');
```

---

## 📡 Canales de Broadcasting

### 📄 `routes/channels.php`
```php
<?php

use Illuminate\Support\Facades\Broadcast;

/*
|--------------------------------------------------------------------------
| Canales de Broadcasting
|--------------------------------------------------------------------------
|
| Definición de canales para eventos en tiempo real usando Laravel Echo
| y Laravel Reverb para WebSockets
|
*/

// ===== CANAL PÚBLICO DE PRODUCCIÓN =====
// Canal para transmitir actualizaciones de producción en tiempo real
Broadcast::channel('produccion', function ($user) {
    // Canal público - cualquier usuario autenticado puede escuchar
    return $user ? true : false;
});

// ===== CANALES DE MÁQUINAS =====
// Canal específico para cada máquina individual
Broadcast::channel('maquina.{maquinaId}', function ($user, $maquinaId) {
    // Verificar que el usuario tenga permiso para ver esta máquina
    return $user && $user->can('ver_monitor_maquinas');
});

// Canal para todas las máquinas (dashboard general)
Broadcast::channel('maquinas', function ($user) {
    return $user && $user->can('ver_dashboard');
});

// ===== CANALES DE ALERTAS =====
// Canal para alertas críticas del sistema
Broadcast::channel('alertas.criticas', function ($user) {
    // Solo usuarios con rol de administrador o encargado
    return $user && ($user->hasRole('Administrador') || $user->hasRole('Encargado'));
});

// Canal para alertas de mantenimiento
Broadcast::channel('alertas.mantenimiento', function ($user) {
    return $user && $user->can('gestionar_mantenimiento');
});

// Canal para alertas de calidad
Broadcast::channel('alertas.calidad', function ($user) {
    return $user && $user->can('control_calidad');
});

// ===== CANALES PRIVADOS DE USUARIO =====
// Canal privado para notificaciones específicas del usuario
Broadcast::channel('user.{id}', function ($user, $id) {
    return (int) $user->id === (int) $id;
});

// Canal para el equipo/área de trabajo del usuario
Broadcast::channel('equipo.{equipoId}', function ($user, $equipoId) {
    return $user->equipos->contains($equipoId);
});

// ===== CANALES DE ADMINISTRACIÓN =====
// Canal para eventos del sistema (solo administradores)
Broadcast::channel('sistema.eventos', function ($user) {
    return $user && $user->hasRole('Administrador');
});

// Canal para logs en tiempo real
Broadcast::channel('sistema.logs', function ($user) {
    return $user && $user->can('ver_logs_sistema');
});

// ===== CANALES DE REPORTES =====
// Canal para notificar cuando un reporte está listo
Broadcast::channel('reportes.{userId}', function ($user, $userId) {
    return (int) $user->id === (int) $userId;
});

// ===== CANALES DE PRODUCCIÓN ESPECÍFICA =====
// Canal para seguimiento de una producción específica
Broadcast::channel('produccion.{produccionId}', function ($user, $produccionId) {
    // Verificar que el usuario esté involucrado en esta producción
    $produccion = \App\Models\Produccion::find($produccionId);
    return $produccion && (
        $user->id === $produccion->operador_id ||
        $user->id === $produccion->encargado_id ||
        $user->hasRole('Administrador')
    );
});

// ===== CANALES DE INVENTARIO =====
// Canal para alertas de stock bajo
Broadcast::channel('inventario.alertas', function ($user) {
    return $user && $user->can('gestionar_inventario');
});

// Canal para movimientos de inventario
Broadcast::channel('inventario.movimientos', function ($user) {
    return $user && $user->can('ver_inventario');
});

// ===== CANAL DE PRESENCIA (USUARIOS CONECTADOS) =====
// Canal para mostrar qué usuarios están viendo el dashboard
Broadcast::channel('dashboard.presence', function ($user) {
    if ($user) {
        return [
            'id' => $user->id,
            'name' => $user->name,
            'avatar' => $user->foto_perfil,
            'role' => $user->getRoleNames()->first()
        ];
    }
    return false;
});

// ===== CANALES DE SIMULACIÓN =====
// Canal para eventos de simulación (desarrollo/testing)
Broadcast::channel('simulacion.eventos', function ($user) {
    return $user && (
        config('app.env') !== 'production' || 
        $user->hasRole('Administrador')
    );
});
```

---

## ⚡ Comandos de Consola

### 📄 `routes/console.php`
```php
<?php

use Illuminate\Foundation\Inspiring;
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Schedule;

/*
|--------------------------------------------------------------------------
| Comandos de Consola
|--------------------------------------------------------------------------
|
| Comandos personalizados y programación de tareas para el sistema
| de fábrica biodegradable
|
*/

// ===== COMANDO DE INSPIRACIÓN =====
Artisan::command('inspire', function () {
    $this->comment(Inspiring::quote());
})->purpose('Display an inspiring quote')->hourly();

// ===== COMANDOS DE LIMPIEZA =====
Artisan::command('fabrica:limpiar-datos-antiguos', function () {
    $this->info('Limpiando datos antiguos del sistema...');
    
    // Limpiar producciones antiguas (más de 1 año)
    $producciones = \App\Models\Produccion::where('created_at', '<', now()->subYear())->count();
    \App\Models\Produccion::where('created_at', '<', now()->subYear())->delete();
    $this->info("Eliminadas {$producciones} producciones antiguas");
    
    // Limpiar logs antiguos (más de 3 meses)
    $logs = \Illuminate\Support\Facades\File::glob(storage_path('logs/*.log'));
    $eliminados = 0;
    foreach ($logs as $log) {
        if (filemtime($log) < time() - (90 * 24 * 60 * 60)) {
            unlink($log);
            $eliminados++;
        }
    }
    $this->info("Eliminados {$eliminados} archivos de log antiguos");
    
    $this->info('✅ Limpieza completada');
})->purpose('Limpiar datos antiguos del sistema');

// ===== COMANDOS DE MANTENIMIENTO =====
Artisan::command('fabrica:verificar-mantenimientos', function () {
    $this->info('Verificando mantenimientos pendientes...');
    
    // Buscar mantenimientos que deberían haberse realizado
    $pendientes = \App\Models\Mantenimiento::whereNull('fecha_realizada')
        ->where('fecha_programada', '<=', now())
        ->count();
    
    if ($pendientes > 0) {
        $this->warn("⚠️  Hay {$pendientes} mantenimientos pendientes");
        
        // Enviar notificación a administradores
        $admins = \App\Models\User::role('Administrador')->get();
        foreach ($admins as $admin) {
            $admin->notify(new \App\Notifications\MantenimientosPendientes($pendientes));
        }
    } else {
        $this->info('✅ Todos los mantenimientos están al día');
    }
})->purpose('Verificar mantenimientos pendientes');

// ===== COMANDOS DE REPORTES =====
Artisan::command('fabrica:generar-reporte-diario', function () {
    $this->info('Generando reporte diario...');
    
    // Generar estadísticas del día
    $estadisticas = [
        'fecha' => now()->format('Y-m-d'),
        'produccion_total' => \App\Models\Produccion::whereDate('created_at', today())->sum('kg_producidos'),
        'maquinas_activas' => \App\Models\Maquina::whereHas('estadoVivo', function($q) {
            $q->where('velocidad_actual', '>', 0);
        })->count(),
        'eficiencia_promedio' => \App\Models\Produccion::whereDate('created_at', today())->avg('oee'),
    ];
    
    // Guardar reporte
    \Illuminate\Support\Facades\Storage::disk('local')->put(
        'reportes/diario-' . now()->format('Y-m-d') . '.json',
        json_encode($estadisticas, JSON_PRETTY_PRINT)
    );
    
    $this->info('✅ Reporte diario generado');
    $this->table(['Métrica', 'Valor'], [
        ['Producción Total', $estadisticas['produccion_total'] . ' kg'],
        ['Máquinas Activas', $estadisticas['maquinas_activas']],
        ['Eficiencia Promedio', round($estadisticas['eficiencia_promedio'], 2) . '%'],
    ]);
})->purpose('Generar reporte diario automático');

// ===== COMANDOS DE SIMULACIÓN =====
Artisan::command('fabrica:simular-produccion {maquina} {--duracion=60}', function () {
    $maquinaId = $this->argument('maquina');
    $duracion = $this->option('duracion');
    
    $maquina = \App\Models\Maquina::find($maquinaId);
    if (!$maquina) {
        $this->error('Máquina no encontrada');
        return 1;
    }
    
    $this->info("Simulando producción en {$maquina->nombre} por {$duracion} minutos...");
    
    $inicio = now();
    $contador = 0;
    
    while (now()->diffInMinutes($inicio) < $duracion) {
        // Simular datos de producción
        $datos = [
            'maquina_id' => $maquinaId,
            'kg_incremento' => rand(50, 200) / 100, // 0.5 - 2.0 kg
            'oee' => rand(70, 95), // 70% - 95%
            'velocidad' => rand(800, 1200) / 10, // 80 - 120 kg/h
        ];
        
        // Registrar producción
        app(\App\Services\Contracts\ProduccionServiceInterface::class)
            ->registrarProduccion(
                $datos['maquina_id'],
                $datos['kg_incremento'],
                $datos['oee'],
                $datos['velocidad']
            );
        
        $contador++;
        $this->info("Registro #{$contador} - {$datos['kg_incremento']} kg");
        
        // Esperar 5 segundos antes del siguiente registro
        sleep(5);
    }
    
    $this->info("✅ Simulación completada - {$contador} registros generados");
})->purpose('Simular producción para testing');

// ===== COMANDOS DE BACKUP =====
Artisan::command('fabrica:backup', function () {
    $this->info('Iniciando backup del sistema...');
    
    $fecha = now()->format('Y_m_d_His');
    $nombreBackup = "backup_{$fecha}.sql";
    
    // Comando de mysqldump
    $comando = sprintf(
        'mysqldump -u %s -p%s %s > %s',
        config('database.connections.mysql.username'),
        config('database.connections.mysql.password'),
        config('database.connections.mysql.database'),
        storage_path("backups/{$nombreBackup}")
    );
    
    // Crear directorio si no existe
    if (!is_dir(storage_path('backups'))) {
        mkdir(storage_path('backups'), 0755, true);
    }
    
    // Ejecutar backup
    exec($comando, $output, $returnCode);
    
    if ($returnCode === 0) {
        $this->info("✅ Backup creado: {$nombreBackup}");
        
        // Comprimir archivo
        exec("gzip " . storage_path("backups/{$nombreBackup}"));
        $this->info("✅ Backup comprimido");
    } else {
        $this->error('❌ Error al crear backup');
        return 1;
    }
})->purpose('Crear backup de la base de datos');

// ===== PROGRAMACIÓN DE TAREAS =====
Schedule::command('fabrica:verificar-mantenimientos')->dailyAt('08:00');
Schedule::command('fabrica:generar-reporte-diario')->dailyAt('23:30');
Schedule::command('fabrica:limpiar-datos-antiguos')->weekly()->sundays()->at('02:00');
Schedule::command('fabrica:backup')->daily()->at('01:00');

// Limpiar cache cada hora
Schedule::command('cache:clear')->hourly();

// Procesar colas pendientes cada minuto
Schedule::command('queue:work --stop-when-empty')->everyMinute();
```

---

## 🗺️ Mapa de Rutas del Sistema

### **Estructura Jerárquica**
```
/ (Raíz)
├── /welcome (Página de bienvenida)
├── /dashboard (Panel principal) 🔒
├── /maquinas/ (Gestión de máquinas) 🔒
│   ├── / (Lista)
│   ├── /create (Crear)
│   ├── /{id} (Ver)
│   ├── /{id}/edit (Editar)
│   └── DELETE /{id} (Eliminar)
├── /planta/ (Módulo de planta) 🔒
│   └── /monitor-maquina/ (Monitor)
│       ├── / (Lista de monitores)
│       └── /{id} (Monitor específico)
├── /produccion/ (Gestión de producción) 🔒
├── /inventario/ (Gestión de inventario) 🔒
│   ├── /materias-primas/
│   ├── /productos/
│   └── /proveedores/
├── /mantenimiento/ (Gestión de mantenimiento) 🔒
├── /recetas/ (Recetas y fórmulas) 🔒
├── /reportes/ (Reportes y analytics) 🔒
└── /admin/ (Administración) 🔒👑

API (/api/)
├── /user (Usuario autenticado) 🔒
├── /simulacion/ (Simulación de producción)
├── /maquinas/{id}/estado (Estados de máquinas)
├── /produccion/ (API de producción) 🔒
├── /inventario/ (API de inventario) 🔒
├── /mantenimiento/ (API de mantenimiento) 🔒
├── /notificaciones/ (API de notificaciones) 🔒
├── /webhooks/ (Webhooks externos)
└── /health (Health check)
```

### **Leyenda**
- 🔒 Requiere autenticación
- 👑 Requiere rol de administrador
- Sin icono: Acceso público

---

*Sistema de rutas bien estructurado que proporciona acceso organizado a todas las funcionalidades del sistema de fábrica biodegradable, desde interfaces web hasta servicios API y comunicación en tiempo real.*
