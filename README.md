# 5.2.2 Base de Datos (Database)

Incluye los archivos de migración, respaldos, semillas y estructura inicial de la base de datos, permitiendo la configuración y gestión de los datos del sistema de fábrica biodegradable.

## 📁 Estructura de la Base de Datos

```
├── 📁 BackUp/
│   ├── 📄 2025_08_18_tech_home.sql - Respaldo del 18 de agosto 2025
│   └── 📄 2025_08_19_tech_home.sql - Respaldo del 19 de agosto 2025
│
├── 📁 factories/
│   └── 📄 UserFactory.php - Factory para generar usuarios de prueba
│
├── 📁 migrations/
│   ├── 📄 0001_01_01_000000_create_users_table.php - Tabla de usuarios del sistema
│   ├── 📄 0001_01_01_000001_create_cache_table.php - Tabla de caché del sistema
│   ├── 📄 0001_01_01_000002_create_jobs_table.php - Tabla de trabajos en cola
│   ├── 📄 2025_11_19_031653_create_tipos_maquinas_table.php - Tipos de máquinas
│   ├── 📄 2025_11_19_031655_create_maquinas_table.php - Máquinas de la fábrica
│   ├── 📄 2025_11_19_031656_create_proveedores_table.php - Proveedores de materias primas
│   ├── 📄 2025_11_19_031657_create_materias_primas_table.php - Materias primas del sistema
│   ├── 📄 2025_11_19_031659_create_lotes_materia_prima_table.php - Lotes de materias primas
│   ├── 📄 2025_11_19_031700_create_productos_table.php - Productos fabricados
│   ├── 📄 2025_11_19_031701_create_recetas_table.php - Recetas de productos
│   ├── 📄 2025_11_19_031703_create_receta_detalles_table.php - Detalles de recetas
│   ├── 📄 2025_11_19_031704_create_producciones_table.php - Registro de producciones
│   ├── 📄 2025_11_19_031705_create_produccion_consumos_table.php - Consumos por producción
│   ├── 📄 2025_11_19_031706_create_lotes_productos_table.php - Lotes de productos terminados
│   ├── 📄 2025_11_19_031707_create_mantenimientos_table.php - Mantenimientos de máquinas
│   ├── 📄 2025_11_19_031708_create_paradas_table.php - Paradas de máquinas
│   ├── 📄 2025_11_19_031800_create_maquinas_estado_vivo_table.php - Estado en tiempo real
│   ├── 📄 2025_11_19_040341_create_permission_tables.php - Sistema de permisos
│   └── 📄 2025_11_19_041311_add_activo_to_users_table.php - Campo activo en usuarios
│
└── 📁 seeders/
    ├── 📄 DatabaseSeeder.php - Seeder principal del sistema
    ├── 📄 datos_iniciales.sql - Datos iniciales del sistema
    └── 📄 permisos_basicos.sql - Permisos y roles básicos
```

---

## 🗄️ Estructura de Tablas Principales

### 👥 **Gestión de Usuarios y Permisos**
```sql
-- Tabla de usuarios del sistema
users (
    id, name, email, email_verified_at, password, 
    activo, foto_perfil, remember_token, 
    created_at, updated_at
)

-- Sistema de permisos (Spatie Permission)
permissions, roles, model_has_permissions, 
model_has_roles, role_has_permissions
```

### 🏭 **Gestión de Máquinas**
```sql
-- Tipos de máquinas disponibles
tipos_maquinas (
    id, nombre, descripcion, created_at, updated_at
)

-- Máquinas de la fábrica
maquinas (
    id, codigo, nombre, descripcion, foto, 
    tipo_maquina_id, capacidad_maxima, 
    velocidad_maxima, created_at, updated_at
)

-- Estado en tiempo real de máquinas
maquinas_estado_vivo (
    id, maquina_id, kg_producidos, oee_actual, 
    velocidad_actual, updated_at, created_at
)
```

### 📦 **Gestión de Inventario**
```sql
-- Proveedores de materias primas
proveedores (
    id, nombre, contacto, telefono, email, 
    direccion, created_at, updated_at
)

-- Materias primas del sistema
materias_primas (
    id, nombre, descripcion, unidad_medida, 
    precio_unitario, proveedor_id, 
    created_at, updated_at
)

-- Lotes de materias primas
lotes_materia_prima (
    id, materia_prima_id, cantidad, 
    fecha_vencimiento, precio_lote, 
    created_at, updated_at
)

-- Productos fabricados
productos (
    id, nombre, descripcion, unidad_medida, 
    precio_venta, created_at, updated_at
)

-- Lotes de productos terminados
lotes_productos (
    id, producto_id, cantidad, fecha_produccion, 
    fecha_vencimiento, produccion_id, 
    created_at, updated_at
)
```

### 🧪 **Recetas y Producción**
```sql
-- Recetas de productos
recetas (
    id, producto_id, nombre, descripcion, 
    cantidad_base, created_at, updated_at
)

-- Detalles de recetas (ingredientes)
receta_detalles (
    id, receta_id, materia_prima_id, 
    cantidad_requerida, created_at, updated_at
)

-- Registro de producciones
producciones (
    id, maquina_id, operador_id, encargado_id, 
    fecha_inicio, fecha_fin, kg_producidos, 
    oee, velocidad, created_at, updated_at
)

-- Consumos por producción
produccion_consumos (
    id, produccion_id, lote_materia_prima_id, 
    cantidad_consumida, created_at, updated_at
)
```

### 🔧 **Mantenimiento y Paradas**
```sql
-- Mantenimientos de máquinas
mantenimientos (
    id, maquina_id, tipo, fecha_programada, 
    fecha_realizada, descripcion, realizado_por, 
    costo, horas_parada, created_at, updated_at
)

-- Paradas de máquinas
paradas (
    id, maquina_id, fecha_inicio, fecha_fin, 
    motivo, tipo, descripcion, 
    created_at, updated_at
)
```

---

## 🏗️ Migraciones Cronológicas

### **Migración Base del Sistema**
```php
// 0001_01_01_000000_create_users_table.php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->timestamp('email_verified_at')->nullable();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});

Schema::create('sessions', function (Blueprint $table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->longText('payload');
    $table->integer('last_activity')->index();
});
```

### **Migración de Máquinas**
```php
// 2025_11_19_031655_create_maquinas_table.php
Schema::create('maquinas', function (Blueprint $table) {
    $table->id();
    $table->string('codigo')->unique();
    $table->string('nombre');
    $table->text('descripcion')->nullable();
    $table->string('foto')->nullable();
    $table->foreignId('tipo_maquina_id')->constrained('tipos_maquinas');
    $table->decimal('capacidad_maxima', 10, 2);
    $table->decimal('velocidad_maxima', 10, 2);
    $table->timestamps();
});
```

### **Migración de Estado en Tiempo Real**
```php
// 2025_11_19_031800_create_maquinas_estado_vivo_table.php
Schema::create('maquinas_estado_vivo', function (Blueprint $table) {
    $table->id();
    $table->foreignId('maquina_id')->unique()->constrained('maquinas');
    $table->decimal('kg_producidos', 10, 2)->default(0);
    $table->decimal('oee_actual', 5, 2)->default(0);
    $table->decimal('velocidad_actual', 10, 2)->default(0);
    $table->timestamps();
});
```

---

## 🌱 Seeders y Datos Iniciales

### **DatabaseSeeder Principal**
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // Crear usuario administrador
        \App\Models\User::factory()->create([
            'name' => 'Administrador',
            'email' => 'admin@fabricabiodegradable.com',
            'activo' => true,
        ]);
        
        // Ejecutar seeders específicos
        $this->call([
            TipoMaquinaSeeder::class,
            MaquinaSeeder::class,
            ProveedorSeeder::class,
            MateriaPrimaSeeder::class,
            ProductoSeeder::class,
            RecetaSeeder::class,
            PermissionSeeder::class,
        ]);
    }
}
```

### **Datos Iniciales del Sistema**
```sql
-- datos_iniciales.sql
INSERT INTO tipos_maquinas (nombre, descripcion) VALUES
('Extrusora', 'Máquina para extrusión de material biodegradable'),
('Mezcladora', 'Máquina para mezcla de materias primas'),
('Prensa', 'Máquina para prensado y moldeado'),
('Cortadora', 'Máquina para corte de material');

INSERT INTO proveedores (nombre, contacto, telefono, email) VALUES
('EcoMateriales SA', 'Juan Pérez', '+591 123456789', 'contacto@ecomateriales.com'),
('BioPack Ltda', 'María García', '+591 987654321', 'ventas@biopack.com');
```

### **Permisos y Roles Básicos**
```sql
-- permisos_basicos.sql
INSERT INTO permissions (name, guard_name) VALUES
('ver_dashboard', 'web'),
('gestionar_maquinas', 'web'),
('gestionar_produccion', 'web'),
('gestionar_inventario', 'web'),
('gestionar_mantenimiento', 'web'),
('administrar_sistema', 'web');

INSERT INTO roles (name, guard_name) VALUES
('Administrador', 'web'),
('Operador', 'web'),
('Encargado', 'web'),
('Mantenimiento', 'web');
```

---

## 🔄 Respaldos Automáticos

### **Configuración de Respaldos**
```bash
# Comando para generar respaldos
mysqldump -u usuario -p fabrica_biodegradable > backup/$(date +%Y_%m_%d)_tech_home.sql

# Restaurar desde respaldo
mysql -u usuario -p fabrica_biodegradable < backup/2025_08_19_tech_home.sql
```

### **Estructura de Respaldos**
- ✅ **Respaldos diarios** automáticos
- ✅ **Nomenclatura estándar** con fecha
- ✅ **Compresión** para optimizar espacio
- ✅ **Retención** de 30 días por defecto

---

## 🏭 Factory para Testing

### **UserFactory**
```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
            'activo' => true,
            'remember_token' => Str::random(10),
        ];
    }
}
```

---

## 🔧 Mantenimiento de Base de Datos

### **Comandos Útiles**
```bash
# Ejecutar migraciones
php artisan migrate

# Ejecutar seeders
php artisan db:seed

# Refrescar base de datos
php artisan migrate:refresh --seed

# Verificar estado de migraciones
php artisan migrate:status

# Rollback de migraciones
php artisan migrate:rollback
```

### **Optimizaciones**
- ✅ **Índices** en campos de búsqueda frecuente
- ✅ **Relaciones** optimizadas con foreign keys
- ✅ **Soft deletes** para datos críticos
- ✅ **Timestamps** automáticos en todas las tablas
- ✅ **Validaciones** a nivel de base de datos

---

*Estructura robusta de base de datos que soporta todas las operaciones del sistema de monitoreo y control de fábrica biodegradable.*
