# 🚀 Magia AJAX en el Backend: La Guía Completa de ElectroShopWeb

Imagina que estás en un restaurante. El mesero se acerca y sin que tú tengas que pararte de la mesa (recargar la página web), él toma tu orden, va a la cocina y regresa mágicamente con tu plato. ¡Eso es **AJAX**!

En el desarrollo moderno, el Frontend (React) toma el rol del mesero que viaja por detrás usando `fetch` o `axios`, y nuestro **Backend (Express)** es la cocina. Para que esto suceda, la cocina **no puede entregar el restaurante completo (HTML)**, sólo entrega la **comida (Datos puros, o JSON)**.

A continuación vamos a descubrir todos y cada uno de los archivos clave en tu servidor (`server/`) que están preparados para este mágico intercambio de datos.

---

## 🏗️ 1. El Portal Mágico: Entendiendo JSON
### 📄 `server/servidor.js`
Este es el portero de nuestra cocina. Gracias a **`express.json()`**, el servidor por fin puede "leer" el idioma secreto (JSON) en que React le envía la información por debajo de la mesa sin recargar.

```javascript
// Permite que las peticiones AJAX envíen paquetes (bodys) en texto convertido a JSON 
// Si no tuviera esto, el servidor no entendería los "posts" y las variables aparecerían "undefined".
app.use(express.json());
```

---

## 🛒 2. Sistema de Ventas y Carrito

Toda esta sección despacha pedidos al momento: compras, carritos, pasarelas de pagos. Nada aquí recarga el navegador.

### 📄 `server/api/carrito.js` y `server/src/controllers/CarritoController.js`
Es como tu bolsa de súper digital.
- **¿Cómo usa AJAX?**: Responde inmediatamente con la información del carrito con un mágico `res.json(items)`. O si añades algo con AJAX POST, te devuelve `res.json({ success: true, mensaje: 'Producto añadido...' })` haciendo que React lanze una pequeña alerta.
```javascript
// Respondiendo los productos y el costo total a la "mochila" en segundo plano
res.json({
    items: items,
    resumen: { total_bs: total, cantidad_total: cantidadTotal }
});
```

### 📄 `server/api/compras.js` y `server/src/controllers/CompraController.js`
Toda la transaccionalidad bancaria necesita no cortarse. 
- **¿Cómo usa AJAX?**: Analiza inventarios en mili-segundos y si todo está bien nos manda tu "ticket de compra en JSON" o manda un error por falta de stock.  
```javascript
// Emite el éxito de la compra al momento (React lo recibe y muestra la ventana de pago exitoso)
res.json({
    compra: { id: compra.id, codigo: compra.codigo, total_bs: compra.totalBs... }
});
```

### 📄 `server/api/pagos.js` y `server/src/controllers/PagoController.js`
Se coordinan con las confirmaciones de tarjeta o el sistema de QR detrás de cámaras.
- **¿Cómo usa AJAX?**: Informa de pagos "completados" sin la necesidad de salir y volver a la app. 
```javascript
// La app capta esto silenciosamente y cierra el modal QR automáticamente
res.json({ success: true, message: 'Pago confirmado y stock actualizado' });
```

---

## 👥 3. Seguridad e Identidad (Auth)

El logueo (Inicio de Sesión) ya no te bloquea cambiándote de vista web, lo hace en tiempo real.

### 📄 `server/api/login.js` , `server/api/registro.js` y `server/src/controllers/AuthController.js`
Cuando pones mal tu contraseña, el cuadro se pone rojo de inmediato. ¿Cómo lo supieron tan rápido?
- **¿Cómo usa AJAX?**: La llamada asíncrona validó contra la BD y arrojó el resultado JSON incluyendo tu salvoconducto de seguridad (Token).
```javascript
// Respuesta al Frontend si te la pones mal (error)
return res.status(401).json({ error: 'Credenciales inválidas' });

// Respuesta de que estás adentro sin recargar la página (éxito)
res.json({
    mensaje: 'Inicio de sesión exitoso',
    token: generateToken(user),
    usuario: { nombre: user.nombre }
});
```

### 📄 `server/lib/auth.js`
Aunque no despacha rutas como `res.json()`, es el engranaje detrás del escenario. Lee los headers (*Authentication: Bearer*) que inyecta AJAX en cada petición privada.

---

## 📦 4. Despliegue Dinámico: Catálogo y Productos

A las tiendas grandes no les gusta cargar todas las fotos o productos de golpe. Traen más productos conforme bajas en la pantalla (Scroll Infinito).

### 📄 `server/api/productos.js`
- **¿Cómo usa AJAX?**: Solo manda información JSON en crudo con nombres, precios y ligas de imágenes.
```javascript
// React usa este arreglo JSON para "dividirlo" e incrustar las "Cards" visuales en el HTML.
res.json(productosGenerados);
```

### 📄 `server/api/categorias.js` y `server/src/controllers/CatalogoController.js`
Gestiona tanto menús interactivos como listados extensos.
- **¿Cómo usa AJAX?**: Un menú desplegable puede invocar esto tras de escena cuando acercas el mouse y se pobla de categorías dinámicamente enviando `res.json(categorias)`.

### 📄 `server/api/catalogo-config.js`
Sirve para saber en qué etapa del embudo de compra vas, permitiéndote cambiar de pantalla animadamente en la interfaz web gracias al flujo dictado por JSON `res.json({ pasosFlujo })`.

---

## 🛠️ 5. Sistema y Monitoreo Interno

### 📄 `server/api/estado.js` y `server/src/controllers/SystemController.js`
Como tener el pulso de un doctor. 
- **¿Cómo usa AJAX?**: Para que un Frontend muestre una lucecita verde si "el sistema está online", manda mini-mensajes por AJAX de latencia cada tantos segundos.
```javascript
// Tu Front va preguntando "Hey, sigues vivo?" y el Back le manda el Ok sin recargar nada.
res.json({ status: 'ok', localIp: LOCAL_IP, port: PORT });
```

---

## 🏁 En Resumen, ¿Cómo sé si mi Backend usa AJAX?
El corazón de AJAX en un Backend programado en **NodeJS/Express** (como el tuyo) es su **habilidad para escupir JSON** y no páginas (No hay un `res.render('vista.html')` por ninguna esquina).

Absolutamente todos tus controladores **(`.js`)** tienen esta estructura:  
1. **Llega Datos**: Mediante `req.body` gracias a `express.json()` que parseó.  
2. **Procesa Mágico**: A través de `prisma` para conectarte a las tablas.  
3. **Escupe JSON**: Y en su línea final de su bloque despacha `res.json({ tuMensaje: 'Llegué sin interrumpir al usuario' })`.
