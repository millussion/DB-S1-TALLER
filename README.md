# DB-S1-TALLER
- - -
# Riwi Supply — Análisis de Base de Datos

---

## Fase 1 · Comprensión del Negocio

**¿Qué proceso de negocio representa la información?**  
Es el Proceso de Gestión de Ventas, Facturación y Recaudo también conocido en el ámbito empresarial como el Ciclo de Ingresos, estas compras son realizadas por multiples empresas colombianas a través de la empresa proveedora Riwi Supply, quien es la encargada de comercializar y distribuir los productos. 
Pedido → Factura → Pago.

**¿Qué eventos se están registrando?**  
Actualmente se registra un evento de intercambio monetario por productos donde se almacenan datos relacionados con clientes, sucursales, asesores comerciales, productos, categorías, facturas y pagos. 

**¿Qué actores participan?**  
Los representantes de cada empresa que requieren el producto comercializado y la empresa distribuidora que hace la venta.

**¿Qué información se repite?**  
La información que parece repetirse en este escenarios son: SalesRep, CustomerName, Category, UnitPrice, OrderDate, CustomerCity, BranchCity y ProductName.

**¿Qué problemas genera esta estructura?**  
- **Actualización:** cambiar la ciudad de un cliente obliga a editar múltiples filas.  
- **Inserción:** no se puede registrar un producto sin que exista una venta.  
- **Borrado:** eliminar una factura puede eliminar datos de un cliente que ya no tiene otras facturas.
- **Falta de escalabilidad:** El proceso de ventas y registro puede causar incongruencias y mala optimizacion a futuro.

---

## Fase 2 · Identificación de Entidades

| Entidad | Descripción |
|---|---|
| **Cliente** | El cliente es el que compra, en este caso son empresas como Constructora Andinay y demás. |
| **AsesorComercial** | Empleado que gestiona la venta. |
| **Sucursal** | Sede física desde donde se opera. |
| **Categoria** | Clasificación de los productos. |
| **Producto** | Artículo vendido, pertenece a una categoría. |
| **Factura** | Documento de la transacción (fecha, cliente, asesor, pago). |
| **DetalleFactura** | Línea de producto dentro de una factura (cantidad y precio). |

---

## Fase 3 · Identificación de Atributos

### Cliente
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| id_cliente | INT | ✅ | |
| nombre_cliente | VARCHAR(100) | | |
| ciudad_cliente | VARCHAR(60) | | |


### Sucursal
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| id_sucursal | INT | ✅ | |
| nombre_sucursal | VARCHAR(60) | | |
| ciudad_sucursal | VARCHAR(60) | | |

### Categoria
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| id_categoria | INT | ✅ | |
| nombre_categoria | VARCHAR(60) | | |

### Producto
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| id_producto | INT | ✅ | |
| nombre_producto | VARCHAR(100) | | |
| precio_unitario | DECIMAL(12,2) | | |
| id_categoria | INT | | ✅ → Categoria |

### Factura
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| numero_factura | VARCHAR(20) | ✅ | |
| fecha_orden | DATE | | |
| metodo_pago | VARCHAR(30) | | |
| id_cliente | INT | | ✅ → Cliente |
| id_asesor | INT | | ✅ → AsesorComercial |
| id_sucursal | INT | | ✅ → Sucursal |

### DetalleFactura
| Atributo | Tipo | PK | FK |
|---|---|:---:|:---:|
| id_detalle | INT | ✅ | |
| numero_factura | VARCHAR(20) | | ✅ → Factura |
| id_producto | INT | | ✅ → Producto |
| cantidad | INT | | |
| precio_unitario | DECIMAL(12,2) | | |

---

## Fase 4 · Identificación de Relaciones

**Relaciones 1 a Muchos:**

| Padre (1) | Hijo (N) |
|---|---|
| Cliente | Factura |
| AsesorComercial | Factura |
| Sucursal | Factura |
| Categoria | Producto |
| Factura | DetalleFactura |
| Producto | DetalleFactura |

**Relación Muchos a Muchos:**  
Factura ↔ Producto → resuelta con la tabla intermedia `DetalleFactura`.

**Dependencias funcionales:**

| Determinante | → | Depende |
|---|:---:|---|
| numero_factura | → | fecha, metodo_pago, id_cliente, id_asesor, id_sucursal |
| id_producto | → | nombre_producto, precio_unitario, id_categoria |
| id_categoria | → | nombre_categoria |
| id_cliente | → | nombre_cliente, ciudad_cliente |
| id_sucursal | → | nombre_sucursal, ciudad_sucursal |
| (numero_factura, id_producto) | → | cantidad, precio_unitario |

---

## Fase 5. Revisión y Refinamiento del Modelo.

### 1. ¿Existen atributos duplicados?

Sí:

- CustomerName
- CustomerCity
- SalesRep
- Branch
- BranchCity
- ProductName
- Category
- PaymentMethod

Estos datos aparecen repetidamente porque una factura puede contener varios productos y porque varios clientes pueden comprar los mismos productos.

### 2. ¿Existen dependencias parciales?

Sí.

Al utilizar inicialmente **InvoiceNumber** como clave principal, algunos atributos no dependen completamente de ella:

- ProductName → Category, UnitPrice
- Branch → BranchCity
- CustomerName → CustomerCity

Esto indica que existen datos que pertenecen a entidades independientes y no directamente a la factura.

### 3. ¿Existen dependencias transitivas?

Sí.

- CustomerName → CustomerCity
- Branch → BranchCity
- ProductName → Category
- InvoiceNumber → CustomerName → CustomerCity
- InvoiceNumber → Branch → BranchCity

Estas dependencias generan redundancia y posibles inconsistencias en la base de datos.

### 4. ¿Todas las entidades representan conceptos del negocio?

Sí. Las entidades identificadas corresponden a elementos reales del proceso comercial:

- Cliente
- Asesor Comercial
- Sucursal
- Producto
- Categoría
- Factura
- DetalleFactura

### 5. ¿Las relaciones reflejan correctamente la realidad del proceso?

Sí, pero fue necesario separar la información en entidades independientes para evitar duplicidad de datos y facilitar la normalización.

<img width="1377" height="793" alt="image" src="https://github.com/user-attachments/assets/682d2960-58a7-4b27-98d9-dbaf2ee2d09b" />


---

## Fase 6. Normalización

### 1. Primera Forma Normal (1FN)

La tabla plana original se dividió en 8 tablas independientes
•	Cada celda contiene exactamente un valor (sin listas separadas por comas ni campos multivalorados).
•	Cada tabla tiene una clave primaria definida.

Osea que el modelo si cumple la Primera Forma Normal (1FN).

### 2. Segunda Forma Normal (2FN)

### 3. Tercera Forma Normal (3FN)

## Fase 9. Consultas SQL
### 1. Listar todos los clientes.
SELECT id_cliente, nombre_cliente, ciudad_cliente
FROM Cliente
ORDER BY nombre_cliente;

### 2.	Mostrar todos los productos con su categoría.
SELECT p.id_producto, p.nombre_producto, p.precio_unitario,
       c.nombre_categoria
FROM Producto p
JOIN Categoria c ON p.id_categoria = c.id_categoria
ORDER BY c.nombre_categoria, p.nombre_producto;

### 3.	Consultar las facturas emitidas por ciudad.
SELECT s.ciudad_sucursal, COUNT(f.numero_factura) AS total_facturas
FROM Factura f
JOIN Sucursal s ON f.id_sucursal = s.id_sucursal
GROUP BY s.ciudad_sucursal
ORDER BY total_facturas DESC;

### 4.	Obtener el total vendido por cliente.
SELECT c.nombre_cliente,
       SUM(df.cantidad * df.precio_unitario) AS total_vendido
FROM DetalleFactura df
JOIN Factura f  ON df.numero_factura = f.numero_factura
JOIN Cliente c  ON f.id_cliente      = c.id_cliente
GROUP BY c.id_cliente, c.nombre_cliente
ORDER BY total_vendido DESC;

### 5.	Obtener el total vendido por categoría.
SELECT cat.nombre_categoria,
       SUM(df.cantidad * df.precio_unitario) AS total_vendido
FROM DetalleFactura df
JOIN Producto  p   ON df.id_producto   = p.id_producto
JOIN Categoria cat ON p.id_categoria   = cat.id_categoria
GROUP BY cat.id_categoria, cat.nombre_categoria
ORDER BY total_vendido DESC;

### 6.	Mostrar las facturas atendidas por cada asesor comercial.
SELECT a.nombre_asesor,
       f.numero_factura, f.fecha_orden,
       c.nombre_cliente
FROM Factura f
JOIN AsesorComercial a ON f.id_asesor  = a.id_asesor
JOIN Cliente         c ON f.id_cliente = c.id_cliente
ORDER BY a.nombre_asesor, f.fecha_orden;

### 7.	Consultar los productos más vendidos.
SELECT p.nombre_producto,
       SUM(df.cantidad) AS unidades_vendidas,
       SUM(df.cantidad * df.precio_unitario) AS ingresos_total
FROM DetalleFactura df
JOIN Producto p ON df.id_producto = p.id_producto
GROUP BY p.id_producto, p.nombre_producto
ORDER BY unidades_vendidas DESC;

### 8.	Mostrar las sucursales y la cantidad de facturas gestionadas.
SELECT s.nombre_sucursal, s.ciudad_sucursal,
       COUNT(f.numero_factura) AS cantidad_facturas
FROM Sucursal s
LEFT JOIN Factura f ON s.id_sucursal = f.id_sucursal
GROUP BY s.id_sucursal, s.nombre_sucursal, s.ciudad_sucursal
ORDER BY cantidad_facturas DESC;

### 9.	Consultar ventas realizadas mediante un método de pago específico.
Aqui el ejemplo es de una transferencia
SELECT f.numero_factura, f.fecha_orden,
       c.nombre_cliente,
       SUM(df.cantidad * df.precio_unitario) AS total_factura
FROM Factura f
JOIN MetodoPago mp     ON f.id_metodo_pago = mp.id_metodo_pago
JOIN Cliente    c      ON f.id_cliente      = c.id_cliente
JOIN DetalleFactura df ON f.numero_factura  = df.numero_factura
WHERE mp.nombre_metodo = 'Transferencia'
GROUP BY f.numero_factura, f.fecha_orden, c.nombre_cliente
ORDER BY f.fecha_orden;

### 10.	Obtener el valor total de cada factura. 
SELECT f.numero_factura, f.fecha_orden,
       c.nombre_cliente,
       a.nombre_asesor,
       s.nombre_sucursal,
       mp.nombre_metodo,
       SUM(df.cantidad * df.precio_unitario) AS total_factura
FROM Factura f
JOIN Cliente         c  ON f.id_cliente      = c.id_cliente
JOIN AsesorComercial a  ON f.id_asesor       = a.id_asesor
JOIN Sucursal        s  ON f.id_sucursal     = s.id_sucursal
JOIN MetodoPago      mp ON f.id_metodo_pago  = mp.id_metodo_pago
JOIN DetalleFactura  df ON f.numero_factura  = df.numero_factura
GROUP BY f.numero_factura, f.fecha_orden,
         c.nombre_cliente, a.nombre_asesor,
         s.nombre_sucursal, mp.nombre_metodo
ORDER BY f.fecha_orden;



