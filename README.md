# 🌐 Enciclopedia Maestra: Arquitectura AJAX en ElectroShopWeb

**¿Qué significa realmente "Tener AJAX" en nuestro ecosistema moderno?**
Históricamente, **AJAX** (Asynchronous JavaScript And XML) fue inventado para que una página de internet pudiera enviar o pedir un dato al servidor por debajo de la mesa sin que esto significara recargar toda la pestaña del navegador web.

En **ElectroShopWeb**, hemos evolucionado el uso de `XML` hacia **`JSON`** (JavaScript Object Notation), el estándar de oro de la industria. Nuestro ecosistema es 100% asíncrono y reactivo:
- **El Frontend (React):** Usa funciones como `fetch` o bibliotecas robustas como `axios` para "pedir y enviar datos". A esto se le conoce como consumir una API REST.
- **El Backend (Node.js/Express):** Recibe la petición, interroga a la base de datos (usando el ORM Prisma) y envía **SOLO DATOS (JSON)** de regreso (`res.json()`), no entrega páginas enteras manipuladas.

A continuación, encontrarás un mapeo exhaustivo y definitivo. Hemos clasificado absolutamente todos los archivos `.js` y `.jsx` que hacen posible nuestra maquinaria asíncrona, con código clave, explicaciones técnicas y el porqué de cada decisión arquitectónica.

---

## 🚀 1. EL FRONTEND: Motores y Estados Globales (Zustand & Axios)

El origen de la magia. Estas configuraciones envían los comandos asíncronos de React hacia Express manteniéndolo organizado.

### 📂 Carpeta `src/services/` (Los Motores HTTP)

| Archivo / Componente | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`apiClient.js`** | Es el "Gran Interceptor AJAX". Inicializa `axios` y le inyecta automáticamente el token de seguridad (JWT) en los *Headers* de cada petición silenciosa que hagas. Así, el backend sabrá quién eres en cada clic sin tener que enviarlo manualmente cada vez. También detecta si el token venció (Error 401) y te desloguea asíncronamente. | `const apiClient = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL });`<br><br>`apiClient.interceptors.request.use((config) => {`<br>&nbsp;&nbsp;`const token = Cookies.get('electro_token');`<br>&nbsp;&nbsp;`if (token) config.headers.Authorization = 'Bearer ' + token;`<br>&nbsp;&nbsp;`return config;`<br>`});` |
| **`api.js`** | Un repertorio de funciones asíncronas puras. Se exportan para ser usadas en componentes específicos de React, abstrayendo la llamada Axios en una función de javascript limpia (`fetchProductos()`). | `export const fetchProductos = async () => {`<br>&nbsp;&nbsp;`const { data } = await apiClient.get('/productos');`<br>&nbsp;&nbsp;`return data;`<br>`};` |

### 📂 Carpeta `src/store/` (Estados Globales Reactivos)

| Archivo / Componente | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`useAuthStore.js`** | Lanza peticiones de seguridad asíncronas. Si la contraseña es correcta, lee el token JSON que el back responde, lo guarda en `Cookies` y modifica el estado del Header a "Autorizado", todo sin que la pantalla haya parpadeado una sola vez. | `const { data } = await apiClient.post('/auth/login', datos);`<br>`Cookies.set('electro_token', data.token);`<br>`set({ user: data.usuario, isAuthenticated: true });` |
| **`useCartStore.js`** | Orquesta las peticiones a Base de Datos en la nube (añadir, sumar, limpiar carrito). Mientras la petición asíncrona vuela (tarda ms), activa una variable `loading: true` para que tu React oscurezca botones y no te deje sumar productos 2 veces accidentalmente. | `set({ loading: true });`<br>`await apiClient.post('/carrito', { producto_id, cantidad });`<br>`await get().fetchCart(); // Refresca lista silenciosamente`<br>`set({ loading: false });` |
| **`useCatalogStore.js`** | Al invocarlo, "jala" pesadas listas de productos o menús de categorías desde la API. React usa esto para poblar el catálogo central. | `const { data } = await apiClient.get('/categorias');`<br>`set({ categorias: data, isLoading: false });` |

---

## 💳 2. EL FRONTEND: Componentes, Hooks y Fetch Nativo

No todo en React usa Axios Global. Ciertos componentes se encargan de pedir de manera encapsulada para no ensuciar el Estado Global.

### 📂 Componentes (`src/components/`, `src/App.jsx`)

| Concepto Técnico | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **El disparador (Hook `useEffect`)** | Normalmente, la primera petición AJAX se dispara tan pronto navegas a la página principal. Componentes como el Home usan `useEffect` para decirle a Firebase o Node: "Tráeme todo ya mismo mientras pinto el Header". | `useEffect(() => {`<br>&nbsp;&nbsp;`fetchCatalogos(); // Inicia AJAX`<br>`}, []);` |
| **Modales (`src/components/Modals/`)** | Componentes como `CheckoutModal.jsx`, `CardPaymentModal` o `TransferModal` usan el **API Nativa del Navegador `fetch`** en lugar de Axios. Recolectan toda la dirección, piden confirmar el pago en la API y esperan la respuesta `200 OK` para brincar a la ventana de éxito sin salir de la página actual. | `const response = await fetch('http://localhost:3000/api/compras/crear', {`<br>&nbsp;&nbsp;`method: 'POST', body: JSON.stringify(formData)`<br>`});`<br>`const data = await response.json();` |

---

## 🧱 3. EL BACKEND BASE: La Central Recepcionista (Express.js)

Para que el servidor sepa hablar "AJAX", necesita ser un intérprete perfecto de JSON.

### 📂 Archivos Raíz y Librerías Core

| Archivo / Configuración | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`server/servidor.js`** | (Central de Tráfico). El navegador bloquea llamadas asíncronas (`ajax`) a diferentes puertos como medida de seguridad (Cors). Además, sin un *middleware* de Express, el cuerpo AJAX (el `req.body`) llegará vacío al backend. | `app.use(cors()); // Desbloquea la barrera AJAX-Seguridad`<br>`app.use(express.json()); // Traduce el string AJAX a Objeto JSON de JS` |
| **`server/lib/auth.js`** | Guarda espaldas de AJAX. Como no hay páginas en el backend para cerrar e impedir el paso, este archivo averigua, validando el Token oculto en cada petición, si ese "clic fantasma del usuario" tiene permisos administrativos o debe recibir un Error AJAX 401. | `const token = req.headers.authorization?.split(' ')[1];`<br>`const decoded = jwt.verify(token, process.env.JWT_SECRET);`<br>`req.usuarioId = decoded.id;` |
| **`server/lib/prisma.js`** | Prisma es asíncrono y reactivo por naturaleza. Al resolver queries rápidamente y basarse en "Promesas" (`Promise`), permite al backend escupir respuestas al Frontend en apenas milisegundos, manteniendo el AJAX súper veloz. | `const prisma = new PrismaClient();`<br>`module.exports = prisma;` |

---

## 🛍️ 4. EL BACKEND (V1): Enrutadores API (Respondiedo con JSON)

### 📂 Carpeta `server/api/`
¿Cómo sabemos que estos endpoints están hechos para AJAX? Porque **todos terminan con `res.json(...)`** o `res.send(...)` y **NINGUNO usa `res.render('vista')`** que era la vieja escuela de retornar páginas webs monolíticas.

| Archivo Controlador | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`api/carrito.js`** | Responde exclusivamente a peticiones POST (insertar) o GET (listar). Manda arreglos de objetos que actualizan visualmente los números del carrito de compras. | `res.json({ success: true, mensaje: 'Producto añadido con éxito' });` |
| **`api/compras.js`** | Es una bomba de validación. Revisa el stock en el tiempo de la red. Retorna **`409 Conflict`** si falla (React lo lee de inmediato y avisa), o retorna el código único del ticket si todo salió bien (`200`). | `if(faltantes.length) return res.status(409).json({ mensaje: 'Sin stock', faltantes });`<br>`res.json({ compra: { id, codigo, totalBs } });` |
| **`api/categorias.js`** | Entrega información indexada pura. React usa este array asíncrono para mapear e inyectarlo en componentes del Header y sub-menús iterativamente (`.map()`). | `const categorias = await prisma.categoria.findMany(...);`<br>`res.json(categorias);` |
| **`api/productos.js`** | Responsable del "Lazy Loading" (Carga diferida) y de filtros instantáneos. Despacha decenas de artículos mediante JSON a máxima velocidad. | `res.json(productosGenerados);` |
| **`api/login.js` & `api/registro.js`** | Puntos calientes del control de identidad silencioso. Informan si te equivocaste en la clave (`401`) o la emiten (`200` y JWT) sin interrumpir el formulario que estás editando. | `if (!valid) return res.status(401).json({ error: 'Inválido' });`<br>`res.json({ token: generateToken(user), usuario: user });` |
| **`api/estado.js`** | Conocidas como Endpoints de latencia (`/ping` o `/health`). Sirven para que el front haga encuestas AJAX (polling) preguntando continuamente "¿Sigues ahí?". | `res.json({ status: 'ok', time: Date.now() });` |

---

## 🏛️ 5. EL BACKEND (V2): Controladores MVC (Dominio Lógico Avanzado)

Tu sistema creció, y por tanto, usa MVC. En lugar de poner miles de líneas de cálculos de tienda en las enrutadores de la carpeta anterior, se delegaron a los controladores de `server/src/controllers/`.

### 📂 Carpeta `server/src/controllers/`

| Archivo de Lógica MVC | Función en la Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`AuthController.js`** | Procesador central de contraseñas con inyección asíncrona de `Bcrypt`. Cuando acaba, escupe el JWT JSON de regreso a `useAuthStore.js`. | `exports.login = async (req, res) => {`<br>&nbsp;&nbsp;`// logica extensa y bcrypt...`<br>&nbsp;&nbsp;`res.json({ mensaje: 'Éxito', token... });`<br>`};` |
| **`CarritoController.js`** | Tiene la importante misión matemática de calcular las reducciones (`.reduce`) para enviar los totales listos de precio x cantidad (`total_bs`) directos al Navbar de React. | `exports.agregarItem = async (req, res) => {`<br>&nbsp;&nbsp;`// logica extensa sumatoria... `<br>&nbsp;&nbsp;`res.json({ success: true, mensaje: 'Añadido' });`<br>`};` |
| **`CompraController.js`** | Orquesta **Múltiples Transacciones Asíncronas (`$transaction`)** bloqueando filas de SQL temporalmente. Mientras esto pasa, el Front-End muestra un Spinner rotando gracias al estado de carga de Zustand. | `const compra = await prisma.$transaction(async (tx) => { ... });`<br>`res.json(compra);` |
| **`CatalogoController.js`** | Creador masivo de diccionarios. Filtra cruces de sucursales, precios e imágenes gigantes de la tienda. | `exports.getProductos = async (req, res) => {`<br>&nbsp;&nbsp;`const productosConStock = await prisma...`<br>&nbsp;&nbsp;`res.json(productosConStock);`<br>`};` |

---

## 🛡️ 6. Metodologías de Red: "El Semáforo AJAX HTTP"
A diferencia de hace 15 años donde casi todo era "GET" o "POST" desde un WebForm de HTML feo, tu arquitectura AJAX secciona a la perfección cada evento usando Verbos CRUD:
*   🟢 **GET (Pedir):** Trae el catálogo, trae información del usuario autenticado. 
*   🟡 **POST (Crear):** Manda un nuevo producto al carrito, procesa un login.
*   🟠 **PUT/PATCH (Actualizar):** Sube la cantidad de 1 item en tu carrito a 2.
*   🔴 **DELETE (Quitar):** Quitar un registro específico del carrito vía `apiClient.delete('/carrito/5')`.

---

## ⚡ 7. Conclusión: El Viaje Mágico de "Añadir al Carrito" (Paso a Paso Final)

Si pusiéramos en cámara lente cómo trabaja toda tu app (El gran flujo AJAX):
1. Haces Clic en "Añadir". Tu pantalla NO pestañea, pero internamente React ejecuta `addItem` del Store **(`useCartStore.js`)**.
2. Ese Store invoca a **`apiClient.js`** para lanzar una petición súper silenciosa **POST `/carrito`** con tu Token oculto.
3. Express la atrapa en **`servidor.js`** porque reconoce su JSON interior (`express.json`).
4. Pasa por el guarda de seguridad **`auth.js`**, descubre quién eres.
5. Llega hasta **`CarritoController.js`**, guarda en BD el producto mediante **`prisma.js`**.
6. Exhala velozmente el código: **`res.json({ success: true })`**.
7. React cacha esa respuesta 50 milisegundos después, cambia de color el ícono a verde y muestra un "Toast" de *"Añadido exitosamente"*.

¡Esa es toda la sinergia, la complejidad de microservicios y la total madurez asíncrona (AJAX) que está construida piedra sobre piedra en tu proyecto **ElectroShopWeb**!
