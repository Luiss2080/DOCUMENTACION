# 5.2.4 Recursos del Frontend (Resources)

Contiene todos los recursos del frontend del sistema, incluyendo hojas de estilo, JavaScript, componentes Vue.js con Inertia.js y vistas Blade para la interfaz de usuario del sistema de fábrica biodegradable.

## 📁 Estructura de Recursos

```
├── 📁 css/
│   └── 📄 app.css - Estilos principales con Tailwind CSS
│
├── 📁 js/
│   ├── 📄 app.js - Configuración principal de la aplicación Vue/Inertia
│   ├── 📄 bootstrap.js - Configuración inicial de librerías
│   ├── 📁 Components/
│   │   ├── 📄 ApplicationLogo.vue - Logo de la aplicación
│   │   ├── 📄 DashboardCard.vue - Tarjetas del dashboard
│   │   ├── 📄 MaquinaEstadoCard.vue - Tarjeta de estado de máquina
│   │   ├── 📄 GraficaProduccion.vue - Gráfica de producción
│   │   ├── 📄 AlertaComponent.vue - Componente de alertas
│   │   └── 📄 NavigationMenu.vue - Menú de navegación
│   ├── 📁 Layouts/
│   │   ├── 📄 AppLayout.vue - Layout principal de la aplicación
│   │   ├── 📄 GuestLayout.vue - Layout para usuarios no autenticados
│   │   └── 📄 AuthenticatedLayout.vue - Layout para usuarios autenticados
│   └── 📁 Pages/
│       ├── 📄 Welcome.vue - Página de bienvenida
│       ├── 📄 Dashboard.vue - Dashboard principal del sistema
│       ├── 📁 Maquinas/
│       │   ├── 📄 Index.vue - Lista de máquinas
│       │   ├── 📄 Create.vue - Formulario de creación
│       │   ├── 📄 Edit.vue - Formulario de edición
│       │   └── 📄 Show.vue - Detalles de máquina
│       └── 📁 Planta/
│           ├── 📄 MonitorMaquinaIndex.vue - Lista de monitores
│           └── 📄 MonitorMaquinaShow_NEW.vue - Monitor en tiempo real
│
└── 📁 views/
    ├── 📄 app.blade.php - Layout base de Blade/Inertia
    └── 📄 welcome.blade.php - Vista de bienvenida estática
```

---

## 🎨 Estilos y Diseño

### 📄 `resources/css/app.css`
```css
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';

/* Estilos personalizados para la aplicación */
:root {
    --color-primary: #22c55e;
    --color-secondary: #064e3b;
    --color-accent: #fbbf24;
    --color-warning: #f59e0b;
    --color-danger: #ef4444;
    --color-success: #10b981;
}

/* Estilos para componentes de máquinas */
.maquina-card {
    @apply bg-white rounded-lg shadow-md p-6 border border-gray-200 hover:shadow-lg transition-shadow;
}

.maquina-estado-activo {
    @apply bg-green-100 border-green-300 text-green-800;
}

.maquina-estado-parada {
    @apply bg-red-100 border-red-300 text-red-800;
}

.maquina-estado-mantenimiento {
    @apply bg-yellow-100 border-yellow-300 text-yellow-800;
}

/* Estilos para dashboard */
.dashboard-metric {
    @apply bg-gradient-to-r from-green-500 to-green-600 text-white rounded-lg p-6 shadow-lg;
}

.dashboard-metric-value {
    @apply text-3xl font-bold mb-2;
}

.dashboard-metric-label {
    @apply text-green-100 text-sm uppercase tracking-wide;
}

/* Estilos para gráficas */
.chart-container {
    @apply bg-white rounded-lg shadow p-6 border border-gray-200;
}

/* Animaciones personalizadas */
@keyframes pulse-green {
    0%, 100% {
        @apply bg-green-500;
    }
    50% {
        @apply bg-green-400;
    }
}

.maquina-produciendo {
    animation: pulse-green 2s ease-in-out infinite;
}

/* Estilos para formularios */
.form-input {
    @apply block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm 
           focus:outline-none focus:ring-green-500 focus:border-green-500;
}

.form-label {
    @apply block text-sm font-medium text-gray-700 mb-1;
}

.btn-primary {
    @apply bg-green-600 hover:bg-green-700 text-white font-medium py-2 px-4 
           rounded-md focus:outline-none focus:ring-2 focus:ring-green-500 
           focus:ring-offset-2 transition-colors;
}

.btn-secondary {
    @apply bg-gray-600 hover:bg-gray-700 text-white font-medium py-2 px-4 
           rounded-md focus:outline-none focus:ring-2 focus:ring-gray-500 
           focus:ring-offset-2 transition-colors;
}

/* Estilos responsive */
@media (max-width: 640px) {
    .maquina-card {
        @apply p-4;
    }
    
    .dashboard-metric {
        @apply p-4;
    }
}
```

---

## ⚙️ Configuración de JavaScript

### 📄 `resources/js/app.js`
```javascript
import './bootstrap';
import '../css/app.css';

import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';
import { ZiggyVue } from '../../vendor/tightenco/ziggy';

const appName = import.meta.env.VITE_APP_NAME || 'Fábrica Biodegradable';

createInertiaApp({
    title: (title) => `${title} - ${appName}`,
    resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
    setup({ el, App, props, plugin }) {
        return createApp({ render: () => h(App, props) })
            .use(plugin)
            .use(ZiggyVue)
            .mount(el);
    },
    progress: {
        color: '#22c55e',
        includeCSS: true,
        showSpinner: true,
    },
});
```

### 📄 `resources/js/bootstrap.js`
```javascript
import axios from 'axios';
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

// Configuración de Axios
window.axios = axios;
window.axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';

// Configurar token CSRF
let token = document.head.querySelector('meta[name="csrf-token"]');
if (token) {
    window.axios.defaults.headers.common['X-CSRF-TOKEN'] = token.content;
}

// Configuración de Laravel Echo para tiempo real
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

---

## 🏗️ Layouts de la Aplicación

### 📄 `resources/js/Layouts/AppLayout.vue`
```vue
<template>
    <div class="min-h-screen bg-gray-100">
        <!-- Navegación superior -->
        <nav class="bg-white shadow">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="flex justify-between h-16">
                    <div class="flex items-center">
                        <ApplicationLogo class="h-8 w-auto" />
                        <span class="ml-3 text-xl font-semibold text-gray-900">
                            Fábrica Biodegradable
                        </span>
                    </div>
                    
                    <NavigationMenu :user="$page.props.auth.user" />
                </div>
            </div>
        </nav>

        <!-- Contenido principal -->
        <main class="py-6">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <!-- Alertas -->
                <div v-if="$page.props.flash.success" class="mb-4">
                    <AlertaComponent type="success" :message="$page.props.flash.success" />
                </div>
                
                <div v-if="$page.props.flash.error" class="mb-4">
                    <AlertaComponent type="error" :message="$page.props.flash.error" />
                </div>

                <!-- Slot para contenido de la página -->
                <slot />
            </div>
        </main>
    </div>
</template>

<script setup>
import ApplicationLogo from '@/Components/ApplicationLogo.vue';
import NavigationMenu from '@/Components/NavigationMenu.vue';
import AlertaComponent from '@/Components/AlertaComponent.vue';
</script>
```

### 📄 `resources/js/Layouts/AuthenticatedLayout.vue`
```vue
<template>
    <div class="min-h-screen bg-gray-100">
        <!-- Sidebar -->
        <div class="fixed inset-y-0 left-0 z-50 w-64 bg-white shadow-lg transform transition-transform duration-300 ease-in-out lg:translate-x-0"
             :class="{ '-translate-x-full': !sidebarOpen }">
            <div class="flex items-center justify-center h-16 bg-green-600">
                <ApplicationLogo class="h-8 w-auto text-white" />
                <span class="ml-2 text-lg font-semibold text-white">Panel Admin</span>
            </div>
            
            <nav class="mt-8">
                <div class="px-4 space-y-2">
                    <Link :href="route('dashboard')" 
                          class="sidebar-link" 
                          :class="{ 'sidebar-link-active': $page.component === 'Dashboard' }">
                        📊 Dashboard
                    </Link>
                    
                    <Link :href="route('maquinas.index')" 
                          class="sidebar-link"
                          :class="{ 'sidebar-link-active': $page.component.startsWith('Maquinas') }">
                        ⚙️ Máquinas
                    </Link>
                    
                    <Link :href="route('planta.monitor-maquina.index')" 
                          class="sidebar-link"
                          :class="{ 'sidebar-link-active': $page.component.startsWith('Planta') }">
                        📈 Monitor Planta
                    </Link>
                </div>
            </nav>
        </div>

        <!-- Contenido principal -->
        <div class="lg:pl-64">
            <!-- Header -->
            <div class="bg-white shadow-sm border-b border-gray-200">
                <div class="px-4 sm:px-6 lg:px-8">
                    <div class="flex justify-between items-center py-4">
                        <button @click="sidebarOpen = !sidebarOpen" 
                                class="lg:hidden p-2 rounded-md text-gray-600 hover:text-gray-900 hover:bg-gray-100">
                            ☰
                        </button>
                        
                        <h1 class="text-2xl font-semibold text-gray-900">{{ title }}</h1>
                        
                        <div class="flex items-center space-x-4">
                            <span class="text-sm text-gray-600">{{ user.name }}</span>
                            <form @submit.prevent="logout" class="inline">
                                <button type="submit" class="text-sm text-red-600 hover:text-red-800">
                                    Cerrar Sesión
                                </button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Contenido de la página -->
            <main class="p-6">
                <slot />
            </main>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import { Link, router } from '@inertiajs/vue3';
import ApplicationLogo from '@/Components/ApplicationLogo.vue';

defineProps({
    title: String,
    user: Object
});

const sidebarOpen = ref(false);

const logout = () => {
    router.post('/logout');
};
</script>

<style scoped>
.sidebar-link {
    @apply flex items-center px-3 py-2 text-sm font-medium text-gray-600 rounded-md hover:text-gray-900 hover:bg-gray-50;
}

.sidebar-link-active {
    @apply bg-green-50 text-green-700 border-r-2 border-green-600;
}
</style>
```

---

## 🎯 Páginas Principales

### 📄 `resources/js/Pages/Dashboard.vue`
```vue
<template>
    <AppLayout>
        <Head title="Dashboard" />
        
        <div class="space-y-6">
            <!-- Métricas principales -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                <DashboardCard 
                    title="Producción Hoy"
                    :value="estadisticas.produccion_hoy"
                    unit="kg"
                    icon="📦"
                    color="green"
                />
                
                <DashboardCard 
                    title="Máquinas Activas"
                    :value="estadisticas.maquinas_activas"
                    :total="estadisticas.total_maquinas"
                    icon="⚙️"
                    color="blue"
                />
                
                <DashboardCard 
                    title="OEE Promedio"
                    :value="estadisticas.oee_promedio"
                    unit="%"
                    icon="📊"
                    color="yellow"
                />
                
                <DashboardCard 
                    title="Eficiencia"
                    :value="estadisticas.eficiencia"
                    unit="%"
                    icon="⚡"
                    color="purple"
                />
            </div>

            <!-- Estados de máquinas -->
            <div class="bg-white rounded-lg shadow">
                <div class="px-6 py-4 border-b border-gray-200">
                    <h2 class="text-lg font-medium text-gray-900">Estado de Máquinas</h2>
                </div>
                
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 p-6">
                    <MaquinaEstadoCard 
                        v-for="maquina in estadosMaquinas"
                        :key="maquina.maquina_id"
                        :maquina="maquina"
                        @ver-detalle="verDetalleMaquina"
                    />
                </div>
            </div>

            <!-- Gráfica de producción -->
            <div class="bg-white rounded-lg shadow">
                <div class="px-6 py-4 border-b border-gray-200">
                    <h2 class="text-lg font-medium text-gray-900">Producción por Máquina</h2>
                </div>
                
                <div class="p-6">
                    <GraficaProduccion :data="produccionPorMaquina" />
                </div>
            </div>
        </div>
    </AppLayout>
</template>

<script setup>
import { Head, router } from '@inertiajs/vue3';
import AppLayout from '@/Layouts/AppLayout.vue';
import DashboardCard from '@/Components/DashboardCard.vue';
import MaquinaEstadoCard from '@/Components/MaquinaEstadoCard.vue';
import GraficaProduccion from '@/Components/GraficaProduccion.vue';

defineProps({
    estadisticas: Object,
    estadosMaquinas: Array,
    produccionPorMaquina: Array
});

const verDetalleMaquina = (maquinaId) => {
    router.get(`/planta/monitor-maquina/${maquinaId}`);
};
</script>
```

### 📄 `resources/js/Pages/Maquinas/Index.vue`
```vue
<template>
    <AppLayout>
        <Head title="Máquinas" />
        
        <div class="space-y-6">
            <!-- Header -->
            <div class="flex justify-between items-center">
                <h1 class="text-3xl font-bold text-gray-900">Gestión de Máquinas</h1>
                <Link :href="route('maquinas.create')" class="btn-primary">
                    ➕ Nueva Máquina
                </Link>
            </div>

            <!-- Filtros -->
            <div class="bg-white rounded-lg shadow p-6">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div>
                        <label class="form-label">Buscar por nombre</label>
                        <input v-model="filtros.nombre" 
                               type="text" 
                               class="form-input" 
                               placeholder="Nombre de máquina...">
                    </div>
                    
                    <div>
                        <label class="form-label">Tipo de máquina</label>
                        <select v-model="filtros.tipo" class="form-input">
                            <option value="">Todos los tipos</option>
                            <option value="Extrusora">Extrusora</option>
                            <option value="Mezcladora">Mezcladora</option>
                            <option value="Prensa">Prensa</option>
                        </select>
                    </div>
                    
                    <div class="flex items-end">
                        <button @click="limpiarFiltros" class="btn-secondary">
                            🗑️ Limpiar
                        </button>
                    </div>
                </div>
            </div>

            <!-- Lista de máquinas -->
            <div class="bg-white rounded-lg shadow overflow-hidden">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                                Código
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                                Nombre
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                                Tipo
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                                Capacidad
                            </th>
                            <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                                Estado
                            </th>
                            <th class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase">
                                Acciones
                            </th>
                        </tr>
                    </thead>
                    <tbody class="bg-white divide-y divide-gray-200">
                        <tr v-for="maquina in maquinasFiltradas" :key="maquina.id" class="hover:bg-gray-50">
                            <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                                {{ maquina.codigo }}
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                                {{ maquina.nombre }}
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                {{ maquina.tipo.nombre }}
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                {{ maquina.capacidad_maxima }} kg/h
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap">
                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full"
                                      :class="getEstadoClass(maquina.estado)">
                                    {{ maquina.estado || 'Sin estado' }}
                                </span>
                            </td>
                            <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                                <div class="flex justify-end space-x-2">
                                    <Link :href="route('maquinas.show', maquina.id)" 
                                          class="text-green-600 hover:text-green-900">
                                        👁️ Ver
                                    </Link>
                                    <Link :href="route('maquinas.edit', maquina.id)" 
                                          class="text-blue-600 hover:text-blue-900">
                                        ✏️ Editar
                                    </Link>
                                    <button @click="eliminarMaquina(maquina.id)" 
                                            class="text-red-600 hover:text-red-900">
                                        🗑️ Eliminar
                                    </button>
                                </div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </AppLayout>
</template>

<script setup>
import { ref, computed } from 'vue';
import { Head, Link, router } from '@inertiajs/vue3';
import AppLayout from '@/Layouts/AppLayout.vue';

const props = defineProps({
    maquinas: Array
});

const filtros = ref({
    nombre: '',
    tipo: ''
});

const maquinasFiltradas = computed(() => {
    return props.maquinas.filter(maquina => {
        const matchNombre = !filtros.value.nombre || 
            maquina.nombre.toLowerCase().includes(filtros.value.nombre.toLowerCase());
        const matchTipo = !filtros.value.tipo || 
            maquina.tipo.nombre === filtros.value.tipo;
        
        return matchNombre && matchTipo;
    });
});

const limpiarFiltros = () => {
    filtros.value = { nombre: '', tipo: '' };
};

const getEstadoClass = (estado) => {
    switch(estado) {
        case 'Activa': return 'bg-green-100 text-green-800';
        case 'Parada': return 'bg-red-100 text-red-800';
        case 'Mantenimiento': return 'bg-yellow-100 text-yellow-800';
        default: return 'bg-gray-100 text-gray-800';
    }
};

const eliminarMaquina = (id) => {
    if (confirm('¿Está seguro de eliminar esta máquina?')) {
        router.delete(route('maquinas.destroy', id));
    }
};
</script>
```

---

## 🧩 Componentes Reutilizables

### 📄 `resources/js/Components/MaquinaEstadoCard.vue`
```vue
<template>
    <div class="maquina-card" :class="getCardClass()">
        <div class="flex items-center justify-between mb-4">
            <h3 class="text-lg font-semibold text-gray-900">{{ maquina.maquina.nombre }}</h3>
            <span class="indicator" :class="getIndicatorClass()"></span>
        </div>
        
        <div class="space-y-2 text-sm text-gray-600">
            <div>Código: <span class="font-medium">{{ maquina.maquina.codigo }}</span></div>
            <div>Producido: <span class="font-medium text-green-600">{{ maquina.kg_producidos }} kg</span></div>
            <div>OEE: <span class="font-medium">{{ maquina.oee_actual }}%</span></div>
            <div>Velocidad: <span class="font-medium">{{ maquina.velocidad_actual }} kg/h</span></div>
        </div>
        
        <div class="mt-4 flex justify-end">
            <button @click="$emit('ver-detalle', maquina.maquina_id)" 
                    class="text-xs bg-green-600 text-white px-3 py-1 rounded hover:bg-green-700">
                Ver Monitor
            </button>
        </div>
    </div>
</template>

<script setup>
defineProps({
    maquina: Object
});

defineEmits(['ver-detalle']);

const getCardClass = () => {
    const estado = maquina.estado || 'parada';
    return {
        'border-green-300': estado === 'produciendo',
        'border-red-300': estado === 'parada',
        'border-yellow-300': estado === 'mantenimiento'
    };
};

const getIndicatorClass = () => {
    const estado = maquina.estado || 'parada';
    return {
        'bg-green-500': estado === 'produciendo',
        'bg-red-500': estado === 'parada',
        'bg-yellow-500': estado === 'mantenimiento'
    };
};
</script>

<style scoped>
.indicator {
    @apply w-3 h-3 rounded-full;
}
</style>
```

---

## 📱 Vistas Blade

### 📄 `resources/views/app.blade.php`
```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title inertia>{{ config('app.name', 'Fábrica Biodegradable') }}</title>

    <!-- Favicon -->
    <link rel="icon" href="{{ asset('favicon.ico') }}">

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

    <!-- Scripts -->
    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @inertiaHead
</head>
<body class="font-sans antialiased">
    @inertia
</body>
</html>
```

### 📄 `resources/views/welcome.blade.php`
```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    
    <title>{{ config('app.name', 'Fábrica Biodegradable') }}</title>
    
    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=instrument-sans:400,500,600" rel="stylesheet" />
    
    <!-- Styles / Scripts -->
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-50 text-gray-900">
    <div class="min-h-screen flex flex-col justify-center items-center">
        <!-- Header de navegación -->
        @if (Route::has('login'))
            <div class="fixed top-0 right-0 p-6 text-right">
                @auth
                    <a href="{{ url('/dashboard') }}" 
                       class="font-semibold text-green-600 hover:text-green-800 focus:outline-none focus:ring-2 focus:ring-green-500 rounded-md px-3 py-2">
                        Dashboard
                    </a>
                @else
                    <a href="{{ route('login') }}" 
                       class="font-semibold text-gray-600 hover:text-gray-800 focus:outline-none focus:ring-2 focus:ring-green-500 rounded-md px-3 py-2">
                        Iniciar Sesión
                    </a>

                    @if (Route::has('register'))
                        <a href="{{ route('register') }}" 
                           class="ml-4 font-semibold text-gray-600 hover:text-gray-800 focus:outline-none focus:ring-2 focus:ring-green-500 rounded-md px-3 py-2">
                            Registrarse
                        </a>
                    @endif
                @endauth
            </div>
        @endif

        <!-- Contenido principal -->
        <div class="max-w-2xl mx-auto text-center">
            <h1 class="text-6xl font-bold text-green-600 mb-4">🌱</h1>
            <h2 class="text-4xl font-bold text-gray-900 mb-6">Fábrica Biodegradable</h2>
            <p class="text-xl text-gray-600 mb-8">
                Sistema de monitoreo y control en tiempo real para la producción sostenible
            </p>
            
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mt-12">
                <div class="bg-white rounded-lg shadow-md p-6">
                    <div class="text-3xl mb-4">📊</div>
                    <h3 class="text-lg font-semibold text-gray-900 mb-2">Monitoreo en Tiempo Real</h3>
                    <p class="text-gray-600">Seguimiento continuo de la producción y eficiencia de máquinas</p>
                </div>
                
                <div class="bg-white rounded-lg shadow-md p-6">
                    <div class="text-3xl mb-4">⚙️</div>
                    <h3 class="text-lg font-semibold text-gray-900 mb-2">Gestión de Máquinas</h3>
                    <p class="text-gray-600">Control completo del estado y mantenimiento de equipos</p>
                </div>
                
                <div class="bg-white rounded-lg shadow-md p-6">
                    <div class="text-3xl mb-4">📈</div>
                    <h3 class="text-lg font-semibold text-gray-900 mb-2">Analytics Avanzado</h3>
                    <p class="text-gray-600">Reportes y métricas para optimizar la producción</p>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

---

*Interfaz de usuario moderna y responsiva construida con Vue.js, Inertia.js y Tailwind CSS, proporcionando una experiencia fluida para el monitoreo y control del sistema de fábrica biodegradable.*
