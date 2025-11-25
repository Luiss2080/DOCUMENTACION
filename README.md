# 5.2.1 Aplicación Principal (App)

Contiene la lógica central de la aplicación, incluyendo controladores, modelos, servicios, eventos, middleware y proveedores de servicios del sistema de fábrica biodegradable.

## 📁 Estructura de la Aplicación

```
├── 📁 Events/
│   ├── 📁 Maquina/
│   │   ├── 📄 MaquinaCreada.php - Evento cuando se crea una máquina
│   │   ├── 📄 MaquinaActualizada.php - Evento cuando se actualiza una máquina
│   │   └── 📄 MaquinaEliminada.php - Evento cuando se elimina una máquina
│   └── 📁 Produccion/
│       ├── 📄 ProduccionIniciada.php - Evento al iniciar producción
│       ├── 📄 ProduccionFinalizada.php - Evento al finalizar producción
│       └── 📄 ProduccionActualizada.php - Evento al actualizar producción
│
├── 📁 Http/
│   ├── 📁 Controllers/
│   │   ├── 📄 Controller.php - Controlador base abstracto
│   │   ├── 📄 DashboardController.php - Dashboard principal del sistema
│   │   ├── 📄 WelcomeController.php - Página de bienvenida
│   │   ├── 📄 MaquinaController.php - CRUD de máquinas
│   │   ├── 📁 Api/
│   │   │   ├── 📄 SimulacionController.php - Simulación de producción
│   │   │   └── 📄 MaquinaEstadoController.php - Estados de máquinas API
│   │   └── 📁 Planta/
│   │       └── 📄 MonitorMaquinaController.php - Monitoreo en tiempo real
│   └── 📁 Middleware/
│       └── 📄 HandleInertiaRequests.php - Middleware para Inertia.js SPA
│
├── 📁 Models/
│   ├── 📄 User.php - Usuario del sistema con roles y permisos
│   ├── 📄 Maquina.php - Máquinas de la fábrica
│   ├── 📄 TipoMaquina.php - Tipos de máquinas disponibles
│   ├── 📄 MaquinaEstadoVivo.php - Estado en tiempo real de máquinas
│   ├── 📄 Produccion.php - Registro de producciones
│   ├── 📄 ProduccionConsumo.php - Consumo de materias primas
│   ├── 📄 Producto.php - Productos fabricados
│   ├── 📄 LoteProducto.php - Lotes de productos terminados
│   ├── 📄 MateriaPrima.php - Materias primas del sistema
│   ├── 📄 LoteMateriaPrima.php - Lotes de materias primas
│   ├── 📄 Proveedor.php - Proveedores de materias primas
│   ├── 📄 Receta.php - Recetas de productos
│   ├── 📄 RecetaDetalle.php - Detalles de recetas (ingredientes)
│   ├── 📄 Mantenimiento.php - Mantenimientos de máquinas
│   └── 📄 Parada.php - Paradas de máquinas (programadas/imprevistas)
│
├── 📁 Providers/
│   └── 📄 AppServiceProvider.php - Proveedor principal de servicios
│
└── 📁 Services/
    ├── 📄 ProduccionService.php - Servicio de gestión de producción
    └── 📁 Contracts/
        └── 📄 ProduccionServiceInterface.php - Interfaz del servicio
```

---

## 🎯 Funcionalidades Principales

### 📊 **Dashboard y Monitoreo**
- **Dashboard principal**: Estadísticas y métricas en tiempo real
- **Monitor de máquinas**: Vista detallada del estado de cada máquina
- **Alertas y notificaciones**: Sistema de eventos para cambios críticos

### 🏭 **Gestión de Producción**
- **Registro de producción**: Captura automática de datos de producción
- **Control de calidad**: OEE, velocidad y eficiencia
- **Simulación**: Sistema para pruebas y validación

### ⚙️ **Gestión de Máquinas**
- **CRUD completo**: Crear, leer, actualizar y eliminar máquinas
- **Estados en tiempo real**: Monitoreo continuo del estado
- **Mantenimiento**: Programación y registro de mantenimientos
- **Paradas**: Control de paradas programadas e imprevistas

### 📦 **Gestión de Inventario**
- **Materias primas**: Control de stock y lotes
- **Productos**: Gestión de productos terminados
- **Recetas**: Fórmulas y composición de productos
- **Proveedores**: Gestión de proveedores de materiales

---

## 🔄 Arquitectura de Eventos

### **Eventos de Máquina**
```php
// Disparados automáticamente en operaciones CRUD
MaquinaCreada::class       // Nueva máquina registrada
MaquinaActualizada::class  // Máquina modificada
MaquinaEliminada::class    // Máquina eliminada
```

### **Eventos de Producción**
```php
// Disparados durante el ciclo de producción
ProduccionIniciada::class     // Inicio de nuevo ciclo
ProduccionActualizada::class  // Actualización de métricas
ProduccionFinalizada::class   // Fin de ciclo productivo
```

---

## 🛠️ Servicios y Contratos

### **ProduccionService**
```php
interface ProduccionServiceInterface
{
    public function registrarProduccion(
        int $maquinaId, 
        float $kgIncremento, 
        float $oee, 
        float $velocidad, 
        ?Carbon $fechaProduccion = null, 
        bool $isLastRegister = false
    ): array;
    
    public function getEstadisticasDia(): array;
    public function getProduccionPorMaquina(): array;
}
```

### **Funcionalidades del Servicio**
- ✅ **Registro automático** de producción
- ✅ **Cálculo de estadísticas** en tiempo real
- ✅ **Gestión de estados** de máquinas
- ✅ **Validación de datos** de producción
- ✅ **Transacciones** para integridad de datos

---

## 📊 Modelos y Relaciones

### **Relaciones Principales**
```php
// Máquina -> Estado en tiempo real (1:1)
Maquina::class -> MaquinaEstadoVivo::class

// Máquina -> Producciones (1:N)
Maquina::class -> Produccion::class

// Producción -> Consumos (1:N)
Produccion::class -> ProduccionConsumo::class

// Receta -> Detalles (1:N)
Receta::class -> RecetaDetalle::class

// Usuario -> Producciones (1:N)
User::class -> Produccion::class (operador/encargado)
```

### **Características de los Modelos**
- ✅ **Mutadores y Accessors** para formateo de datos
- ✅ **Casting automático** de tipos de datos
- ✅ **Soft Deletes** para eliminación lógica
- ✅ **Scopes** para consultas complejas
- ✅ **Observers** para eventos automáticos

---

## 🌐 Controladores API vs Web

### **Web Controllers**
- **DashboardController**: Dashboard principal con Inertia.js
- **MaquinaController**: CRUD completo con vistas
- **MonitorMaquinaController**: Monitoreo con SSE
- **WelcomeController**: Página de inicio

### **API Controllers**
- **SimulacionController**: Endpoints para simulación
- **MaquinaEstadoController**: API REST para estados
- **Endpoints sin autenticación**: Para simuladores externos

---

## 🔒 Middleware y Seguridad

### **HandleInertiaRequests**
```php
class HandleInertiaRequests extends Middleware
{
    protected $rootView = 'app';
    
    public function share(Request $request): array
    {
        return [
            'auth' => ['user' => $request->user()],
            'flash' => [
                'success' => fn() => $request->session()->get('success'),
                'error' => fn() => $request->session()->get('error'),
            ],
        ];
    }
}
```

### **Características de Seguridad**
- ✅ **Autenticación** integrada con Laravel Sanctum
- ✅ **Autorización** con Spatie Permission
- ✅ **Validación** de datos en todas las operaciones
- ✅ **CSRF Protection** en formularios web
- ✅ **Rate Limiting** en endpoints API

---

*Núcleo de la aplicación que gestiona toda la lógica de negocio del sistema de monitoreo y control de fábrica biodegradable.*
