# 🌐 Guía Maestra y Extendida de AJAX en ElectroShopWeb

**¿Qué es AJAX realmente en nuestro ecosistema moderno?**
Históricamente, **AJAX** (Asynchronous JavaScript And XML) fue inventado para que una página de internet pudiera enviar o pedir un dato al servidor por debajo de la mesa sin que esto significara recargar toda la pestaña del navegador web.

En ElectroShopWeb, hemos evolucionado de `XML` a **`JSON`**. Nuestro ecosistema es 100% asíncrono y reactivo:
- **El Frontend (React):** Usa `fetch` o `axios` para "pedir datos". A esto se le conoce como llamadas a una API.
- **El Backend (Node.js/Express):** Recibe la petición, busca en la base de datos (con Prisma) y envía **SOLO DATOS (JSON)** de regreso (`res.json()`), no páginas enteras.

A continuación, encontrarás un mapeo masivo, archivo por archivo de los más de 30 scripts `.js` y componentes `.jsx` que hacen posible nuestra maquinaria asíncrona, con su respectivo código y explicación paso a paso.

---

## 🚀 1. EL FRONTEND: Motores y Estados Globales (Zustand & Axios)

El origen de la magia. Estas configuraciones permiten enviar los comandos asíncronos de React hacia Express.

### 📂 Carpeta `src/services/` (Comunicaciones HTTP)

| Archivo / Componente | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`apiClient.js`** | Es el "corazón del AJAX global". Inicializa `axios` apuntando al backend local (`http://localhost:3000/api`). Además, inyecta automáticamente el token de seguridad (JWT) en cada petición silenciosa que hagas, para que la sesión asíncrona nunca se pierda. | `const apiClient = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL });`<br><br>`apiClient.interceptors.request.use((config) => {`<br>&nbsp;&nbsp;`const token = Cookies.get('electro_token');`<br>&nbsp;&nbsp;`if (token) config.headers.Authorization = 'Bearer ' + token;`<br>&nbsp;&nbsp;`return config;`<br>`});` |
| **`api.js`** | Un repertorio de funciones asíncronas crudas preparadas para ser usadas en cualquier momento, como obtener todos los productos, sin depender de un estado global. | `export const fetchProductos = async () => {`<br>&nbsp;&nbsp;`const { data } = await apiClient.get('/productos');`<br>&nbsp;&nbsp;`return data;`<br>`};` |

### 📂 Carpeta `src/store/` (Estados Globales Reactivos)

| Archivo / Componente | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`useAuthStore.js`** | Envía usuario y contraseña vía AJAX. Si es exitoso, atrapa el token JSON y actualiza la pantalla a "logueado" (desaparece el modal y aparece el avatar) sin parpadeos. | `const { data } = await apiClient.post('/auth/login', datos);`<br>`Cookies.set('electro_token', data.token);`<br>`set({ user: data.usuario, isAuthenticated: true });` |
| **`useCartStore.js`** | Gestiona peticiones a Base de Datos en la nube (añadir, sumar, limpiar carrito). Mientras la petición asíncrona vuela, activa un `<Loading />` que detiene al usuario de hacer doble-clic rápido. | `set({ loading: true });`<br>`await apiClient.post('/carrito', { producto_id, cantidad });`<br>`await get().fetchCart();`<br>`set({ loading: false });` |
| **`useCatalogStore.js`** | Descarga pesadas listas de productos o menús de categorías, trayéndolas transparentemente desde la API. | `const { data } = await apiClient.get('/categorias');`<br>`set({ categorias: data, isLoading: false });` |

---

## 💳 2. EL FRONTEND: Modales Nativos con API Fetch

No todas las peticiones las hace `axios`. Dentro de React (Vite), los pop-ups usan la API nativa del navegador llamada `fetch` para generar interacciones rápidas y asíncronas.

### 📂 Carpeta `src/components/Modals/`

| Archivo / Componente | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`CheckoutModal.jsx`** | Al darle "Continuar", este AJAX recolecta la dirección y teléfono y lo manda a `compras/crear`. Espera la confirmación JSON del id de venta. | `const response = await fetch('http://localhost:3000/api/compras/crear', {`<br>&nbsp;&nbsp;`method: 'POST', body: JSON.stringify(formData)`<br>`});`<br>`const data = await response.json();` |
| **`CardPaymentModal.jsx`** | Envía los números temporales de la tarjeta hacia el endpoint de confirmación bancaria de segundo plano. | `const response = await fetch('http://localhost:3000/api/compras/confirmar-pago', {`<br>&nbsp;&nbsp;`method: 'POST', body: JSON.stringify({ compraId })`<br>`});` |
| **`TransferModal.jsx`** | Parecido al de tarjeta, pero procesa subidas de comprobantes o validaciones manuales. | `const res = await fetch(`http://localhost:3000/api/compras/${compraId}`);`<br>`const data = await res.json();` |

---

## 🧱 3. EL BACKEND BASE: Configuración Central (Node.js)

Para que el servidor de Express sepa hablar "AJAX JSON" con React, necesita traductores.

### 📂 Archivos Raíz y Librerías

| Archivo / Config | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`server/servidor.js`** | (Central) "Parsea" o descifra el cuerpo de los mensajes AJAX. Evita errores de seguridad de CORS (cuando un puerto AJAX 5173 llama al puerto 3000). | `app.use(cors()); // Habilita la petición cruzada AJAX`<br>`app.use(express.json()); // Transforma el body a JSON` |
| **`server/lib/auth.js`** | No emite JSON, pero es vital. El AJAX del frontend inyecta un encabezado secreto (Header) en formato Bearer. Este archivo frena peticiones AJAX maliciosas que no tengan logueo. | `const token = req.headers.authorization?.split(' ')[1];`<br>`const decoded = jwt.verify(token, process.env.JWT_SECRET);`<br>`req.usuarioId = decoded.id;` |
| **`server/lib/prisma.js`** | La conexión a Base de Datos de forma 100% asíncrona basada en promesas (`await`) garantizando tiempos de respuesta mínimos al cliente AJAX. | `const prisma = new PrismaClient();`<br>`module.exports = prisma;` |

---

## 🛒 4. EL BACKEND (V1): Enrutadores API Directos

### 📂 Carpeta `server/api/`
Toda respuesta de esta carpeta invoca un `res.json(...)`. Si en estos archivos hubiera un `res.render(...)` (para pintar HTML), nuestro frontend se rompería, porque React sólo entiende de listas, textos y números.

| Archivo Controlador | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`api/carrito.js`** | Recibe la petición GET o POST y de devuelve exclusivamente un array de objetos con los items del carrito. | `res.json({ success: true, mensaje: 'Producto añadido con éxito' });` |
| **`api/compras.js`** | Revisa en Base de Datos de manera asíncrona si hay stock justo al momento de pagar. Retorna `409` (Error AJAX de Conflictos) si falla. | `if(faltantes.length) return res.status(409).json({ mensaje: 'Stock insuficiente', faltantes });`<br>`res.json({ compra: { id, codigo, totalBs } });` |
| **`api/categorias.js`** | Envía el árbol genealógico del menú principal. | `const categorias = await prisma.categoria.findMany(...);`<br>`res.json(categorias);` |
| **`api/productos.js`** | Funciona de endpoint de "scroll infinito" despachando todos los productos como lista JSON. | `res.json(productosGenerados);` |
| **`api/login.js`** <br>&<br> **`api/registro.js`** | Endpoints "Puros" de seguridad. Responden al instante un JWT sin que el usuario advierta cambio de página. | `if (!valid) return res.status(401).json({ error: 'Credenciales inválidas' });`<br>`res.json({ token: generateToken(user), usuario: user });` |
| **`api/pagos.js`** | Escucha confirmaciones para integraciones finales. | `res.json({ success: true, message: 'Pago confirmado y actualizado' });` |
| **`api/estado.js`** | Genera un "Ping" de latencia donde el frontend consulta si "sigue vivo" el servidor. | `res.json({ status: 'ok', time: Date.now() });` |

---

## 🏗️ 5. EL BACKEND (V2): Controladores MVC (Lógica Compleja)

Para evitar que los archivos de rutas se vuelvan gigantescos, la app usa la arquitectura MVC. Estos "Controllers" procesan cientos de líneas de cálculo de impuestos, precios y stock interno de Prisma. Sin embargo, su **único propósito de cierre es devolver AJAX JSON**.

### 📂 Carpeta `server/src/controllers/`

| Archivo Intermedio / MVC | Función en Arquitectura AJAX | Código Clave Asíncrono |
|:---|:---|:---|
| **`AuthController.js`** | Reemplaza a `login.js/registro.js` enrutando y encadenando métodos asíncronos para seguridad en base de datos cifrada. | `exports.login = async (req, res) => {`<br>&nbsp;&nbsp;`// logica extensa bcrypt...`<br>&nbsp;&nbsp;`res.json({ mensaje: 'Inicio de sesión exitoso', token... });`<br>`};` |
| **`CarritoController.js`** | Realiza el `reduce()` matemático para actualizar los totales (`total_bs`) que el frontend actualizará en el ícono superior derecho. | `exports.agregarItem = async (req, res) => {`<br>&nbsp;&nbsp;`// logica extensa sumatoria...`<br>&nbsp;&nbsp;`res.json({ success: true, mensaje: 'Añadido con éxito' });`<br>`};` |
| **`CompraController.js`** | Utiliza **Transacciones Múltiples de Prisma (`$transaction`)** bloqueando tablas. La petición AJAX se queda esperando respuesta en modo `fetch` hasta acabar o revertir un error. | `const compra = await prisma.$transaction(async (tx) => { ... });`<br>`res.json(compra);` |
| **`CatalogoController.js`** | Devuelve objetos muy masivos. Aquí el AJAX permite traer descripciones, fotografías y configuraciones del catálogo de un jalón que se "cachea" visualmente en React. | `exports.getProductos = async (req, res) => {`<br>&nbsp;&nbsp;`const productosConStock = await prisma...`<br>&nbsp;&nbsp;`res.json(productosConStock);`<br>`};` |
| **`SystemController.js`** | Retorna variables de entorno y estado como puertos o IPs y diagnósticos al panel admin del frontend. | `exports.getSystemStatus = (req, res) => {`<br>&nbsp;&nbsp;`res.json({ status: 'ok', localIp: LOCAL_IP, port: PORT });`<br>`};` |

---

## 🎯 Síntesis Absoluta

Si analizas cualquiera de los +30 archivos arriba listados, descubrirás un patrón innegable, la perfecta sinfonía asíncrona moderna:

1. **(El Botón React) Frontend:** Llama una URL asíncrona, usando `await fetch()` o `apiClient.get()`.
2. **(El Portero) `servidor.js`:** Lee el `req.body` gracias a que activó `express.json()`.
3. **(El Gerente) `lib/auth.js`:** Verifica inmediatamente si tienes permisos.
4. **(El Obrero) `controllers/`:** Con Prisma, la Base de Datos responde a NodeJS velozmente mediante **Promises**.
5. **(El Mesero) Entrega JSON:** Por fin se emite `res.json(...)`.
6. En React (`store / Modals`), las variables captan esa información pura y "Redibujan" la pantalla visualmente, dando lugar a una experiencia sin recarga 100% AJAX.
