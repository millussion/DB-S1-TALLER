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
