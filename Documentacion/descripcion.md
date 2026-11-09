# BookStop — Sistema de gestión comercial para librería y bazar

> **Estado:** Propuesta inicial de producto y alcance funcional.  
> **Tipo de proyecto:** Aplicación de gestión comercial.  
> **Contexto:** BookStop es una tienda real dedicada a la venta de libros, juguetes, artículos de librería, papelería y bazar.

## 1. Descripción general

BookStop es un sistema de gestión comercial cuyo propósito es centralizar la administración de inventario, productos, proveedores, compras, ventas y gastos operacionales de una librería-bazar. También permitirá visualizar indicadores relevantes del negocio para apoyar decisiones de reposición, abastecimiento y gestión financiera.

La aplicación se desarrollará inicialmente para una tienda, con una arquitectura modular que permita agregar nuevas funcionalidades e integraciones sin rehacer el núcleo del sistema. Su evolución podrá considerar el uso en otros comercios, pero **la primera versión estará enfocada en las necesidades reales de BookStop**.

## 2. Problema que busca resolver

En comercios pequeños, la información de existencias, compras, ventas y gastos suele encontrarse distribuida entre registros manuales, planillas y plataformas de cobro. Esto dificulta conocer el stock disponible, identificar productos que requieren reposición, evaluar la rentabilidad y consultar el historial de operaciones.

BookStop busca reunir esa información en una aplicación, mantener trazabilidad sobre los movimientos del negocio y generar alertas e indicadores útiles para tomar decisiones.

## 3. Objetivos

### Objetivo general

Desarrollar una aplicación que permita administrar las operaciones comerciales esenciales de BookStop y generar información confiable sobre inventario, ventas, compras y gastos.

### Objetivos específicos

- Mantener un catálogo centralizado de productos y sus existencias.
- Registrar cada cambio de stock con su origen y responsable.
- Administrar datos de proveedores y compras asociadas.
- Registrar ventas, gastos operacionales y pagos vinculados.
- Incorporar importación de transacciones de proveedores de pago, empezando por TUU.
- Emitir alertas de inventario basadas en stock y actividad de ventas.
- Presentar métricas para evaluar ventas, rotación, gastos y márgenes.
- Establecer roles, permisos y reglas de negocio verificables.

## 4. Alcance funcional

### 4.1. Catálogo e inventario

El sistema deberá permitir:

- Crear y editar productos.
- Consultar productos por nombre, categoría, código interno (SKU) o código de barras, cuando exista.
- Registrar información de libros (por ejemplo, ISBN, autor, editorial y edición) cuando corresponda.
- Gestionar categorías como libros, juguetes, papelería y bazar.
- Registrar costo de adquisición, precio de venta y unidad de medida.
- Consultar el stock actual, reservado (si se incorpora esta función) y disponible.
- Registrar entradas y salidas de stock por compras, ventas, devoluciones, pérdidas, daños y ajustes.
- Desactivar productos en lugar de borrarlos cuando tengan movimientos históricos.
- Consultar el historial de movimientos de cada producto.

**Criterio de diseño:** los cambios de existencias no deben limitarse a editar un número; deben registrarse como movimientos trazables.

### 4.2. Alertas de stock y reposición

Como regla inicial, la aplicación notificará cuando un producto cumpla simultáneamente:

1. Su stock disponible sea **menor a 3 unidades**.
2. Registre **al menos una unidad vendida durante los últimos 14 días**.

Expresión inicial:

```text
alerta_reposicion = (stock_disponible < 3) AND (unidades_vendidas_ultimos_14_dias > 0)
```

Las alertas se mostrarán dentro de la aplicación. En una etapa posterior podrán configurarse umbrales distintos por producto o categoría, así como períodos de análisis ajustables.

Los productos con poco stock **sin ventas recientes** podrán figurar en un informe secundario, sin activar una alerta prioritaria de reposición. Esta regla es una ayuda para decidir, no una orden de compra automática.

### 4.3. Proveedores

El sistema permitirá registrar:

- Nombre o razón social.
- Teléfono y otros datos de contacto.
- Información bancaria necesaria para efectuar pagos, con acceso restringido.
- Productos o categorías que suministra, si corresponde.
- Compras asociadas e historial de operaciones.
- Estado del proveedor (activo/inactivo).

Los datos bancarios deben tratarse como información sensible del negocio: no deben exponerse en vistas públicas, logs ni reportes generales.

### 4.4. Compras y recepción de mercadería

- Registrar compras u órdenes de compra asociadas a un proveedor.
- Agregar productos, cantidades, costos unitarios y totales.
- Mantener estados como pendiente, recibida o cancelada.
- Registrar la recepción total o parcial de mercadería.
- Generar movimientos de entrada de stock únicamente por las cantidades efectivamente recibidas.
- Conservar precios y costos históricos, incluso si se actualiza el catálogo.
- Vincular pagos y gastos asociados, como transporte o envío, sin duplicar registros financieros.

### 4.5. Ventas y medios de pago

- Registrar ventas manuales con fecha, productos, cantidades, precios y descuentos cuando corresponda.
- Identificar el medio de pago y su estado.
- Registrar anulaciones y devoluciones con los ajustes respectivos.
- Consultar historial de ventas y detalles de cada operación.
- Importar transacciones externas sin duplicarlas.
- Permitir conciliación entre las ventas registradas en BookStop y los pagos informados por una plataforma de cobro.

**Distinción fundamental:** una transacción de pago por un monto determinado no siempre identifica los productos vendidos. Por ello, **no se debe descontar stock automáticamente** al importar un pago si no existe un detalle de productos confiable o una asociación validada con una venta.

#### Integración con TUU y otras plataformas

Se propone una interfaz común de integración para incorporar distintas fuentes de transacciones mediante el patrón **Adapter**.

```text
BookStop / Módulo de ventas
            |
            v
Interfaz de importación y normalización de transacciones
            |
    +-------+-------+
    |       |       |
 Adaptador Adaptador Adaptador
    TUU      CSV     futuro
```

La lógica comercial de BookStop no deberá depender directamente del formato de una plataforma específica.

**Estrategia de implementación:**

1. Comenzar con registro de ventas manuales.
2. Incorporar importación de CSV con validación de columnas y detección de duplicados.
3. Implementar, si las credenciales, permisos y datos disponibles lo permiten, una integración mediante API con TUU.
4. Permitir nuevos adaptadores para otros proveedores sin modificar la lógica principal.

La disponibilidad, cobertura y condiciones de acceso a datos de TUU deberán verificarse antes de comprometer una integración automática. Para definir la sincronización, también se deberá confirmar cómo se registran actualmente las ventas en el local (por producto, mediante código de barras o solo por monto).

### 4.6. Gastos y control financiero

La aplicación permitirá ingresar y clasificar egresos tales como:

- Compras a proveedores.
- Envíos y transporte.
- Arriendo.
- Electricidad, agua e internet.
- Comisiones de medios de pago.
- Otros gastos operacionales.

Se deberá registrar fecha, monto, categoría, descripción, comprobante o referencia opcional y estado de pago cuando corresponda.

**Importante:** se distinguirá entre:

- **Ventas e ingresos:** operaciones comerciales e ingresos de dinero.
- **Desembolsos:** salidas de dinero, incluyendo pagos a proveedores.
- **Costo de productos vendidos:** costo de las unidades efectivamente vendidas.
- **Gastos operacionales:** costos asociados al funcionamiento del negocio.

Esta separación evitará calcular incorrectamente la utilidad tratando toda compra de mercadería como un gasto inmediato del período. El sistema tendrá fines de gestión interna y no reemplazará por sí solo la contabilidad ni las obligaciones tributarias del negocio.

### 4.7. Métricas y panel de control

El módulo de métricas mostrará indicadores calculados a partir de los datos efectivamente disponibles:

**Ventas**
- Ventas por día, semana y mes.
- Comparaciones entre períodos.
- Ventas por categoría y por producto.
- Productos más y menos vendidos.
- Ticket promedio, cuando existan ventas completas registradas.

**Inventario**
- Cantidad de productos activos y valor estimado del stock.
- Productos con stock bajo.
- Productos sin movimiento en un período determinado.
- Rotación de inventario y sugerencias de reposición, cuando existan datos suficientes.

**Finanzas y proveedores**
- Egresos por categoría y período.
- Compras por proveedor.
- Margen bruto estimado cuando se cuente con costos confiables.
- Resultado comercial estimado según ingresos, costo de ventas y gastos registrados.
- Comisiones y conciliación de pagos, cuando haya información disponible.

Los paneles deberán indicar el período y filtros utilizados. Las métricas que dependan de datos incompletos deberán identificarse como estimaciones o quedar deshabilitadas, en vez de presentar valores engañosos.

### 4.8. Usuarios, roles y permisos

Se contempla autenticación de usuarios y, al menos, los siguientes perfiles:

- **Administrador:** configuración general, proveedores, compras, gastos, reportes y gestión de usuarios.
- **Vendedor:** acceso a catálogo, consulta de stock y registro de ventas, según permisos asignados.

Las acciones sensibles —como ajustes de inventario, modificación de costos y acceso a cuentas bancarias— deberán tener permisos explícitos y registro del responsable.

## 5. Reglas de negocio iniciales

1. Una venta normal no puede dejar el stock disponible en negativo.
2. Todo cambio de existencias debe generar un movimiento trazable con fecha, motivo y usuario o proceso responsable.
3. Los productos con historial no se eliminan físicamente; se desactivan.
4. Una transacción externa importada no debe registrarse más de una vez.
5. Las devoluciones y anulaciones deben ajustar los registros asociados sin duplicar movimientos.
6. El costo y precio aplicados a una venta deben conservarse como datos históricos de esa operación.
7. La recepción de una compra debe actualizar el stock solo una vez por cada cantidad recibida.
8. Las compras, ventas y actualizaciones de inventario que dependan entre sí deberán ejecutarse con garantías de consistencia transaccional.
9. Los datos bancarios de proveedores y otros datos sensibles solo serán visibles para usuarios autorizados.
10. Las alertas de reposición obedecerán inicialmente a la regla de stock menor a 3 y ventas en los últimos 14 días.
11. Los pagos importados sin detalle de artículos no deberán generar salidas de inventario automáticamente.
12. Una misma operación económica no se debe contabilizar dos veces por aparecer en módulos distintos.

## 6. Arquitectura técnica propuesta

Se propone comenzar con un **monolito modular**, para mantener bajo control la complejidad de despliegue y desarrollo, sin perder separación de responsabilidades.

| Componente | Tecnología propuesta | Motivo |
| --- | --- | --- |
| Frontend | React + TypeScript | Interfaz dinámica y componentes reutilizables. |
| Backend | NestJS + TypeScript | Organización por módulos, servicios e inyección de dependencias. |
| Persistencia | PostgreSQL | Integridad relacional y transacciones. |
| ORM | Prisma | Acceso tipado a datos y migraciones. |
| Integraciones | Interfaces y patrón Adapter | Aislamiento de dependencias externas. |
| API | REST + OpenAPI/Swagger | Contratos documentados para frontend e integraciones. |
| Pruebas | Unitarias y de integración (Jest/Vitest/Supertest, según configuración) | Validación de reglas y flujos críticos. |

**Módulos candidatos del backend:** `auth`, `users`, `catalog`, `inventory`, `suppliers`, `purchases`, `sales`, `payments`, `expenses`, `analytics` e `integrations`.

La separación modular no implica desplegar microservicios. Se favorecerá una arquitectura sencilla con límites de responsabilidad claros y decisiones técnicas documentadas.

### Entidades iniciales por evaluar

- `Product`, `Category`.
- `InventoryMovement`.
- `Supplier`, `Purchase`, `PurchaseItem`, `GoodsReceipt`.
- `Sale`, `SaleItem`.
- `PaymentTransaction`, `PaymentProvider`, `PaymentReconciliation`.
- `Expense`, `ExpenseCategory`.
- `User`, `Role`.

Los nombres y relaciones definitivos se especificarán en el documento de modelo de datos antes de implementarlos. Las métricas pueden calcularse a partir de las operaciones existentes sin requerir inicialmente una tabla independiente por indicador.

## 7. Etapas de implementación

### Versión 1 — MVP

- Catálogo de productos y categorías.
- Registro de inventario y movimientos.
- Gestión básica de proveedores.
- Registro de compras recibidas.
- Registro manual de ventas.
- Registro de gastos.
- Alertas de bajo stock con criterio de ventas recientes.
- Panel de métricas básicas.
- Autenticación y permisos básicos.

**Resultado esperado:** una primera versión utilizable para administrar operaciones esenciales del local con datos de prueba y, posteriormente, datos reales controlados.

### Versión 2 — Integraciones y consolidación

- Importación de transacciones desde archivos CSV.
- Adaptador para TUU, sujeto a validación del acceso y los datos disponibles.
- Conciliación de pagos y ventas.
- Recepciones parciales y mejoras de órdenes de compra.
- Métricas financieras y operacionales avanzadas.
- Exportación de reportes.

### Versión 3 — Evolución del producto

- Gestión de múltiples sucursales o bodegas.
- Transferencias de inventario.
- Umbrales de reposición por producto y recomendaciones basadas en historial.
- Nuevas integraciones con proveedores de pago.
- Posibles funcionalidades de reservas y preventas.

Estas etapas son una propuesta de alcance: las funcionalidades futuras no forman parte de los requisitos obligatorios del MVP.

## 8. Aspectos no funcionales y criterios de calidad

- **Integridad:** las operaciones críticas de venta, compra y stock deben mantener consistencia incluso ante errores o solicitudes simultáneas.
- **Seguridad:** autenticación, permisos, validación de entradas, gestión segura de secretos y protección de información financiera.
- **Trazabilidad:** registro del origen y responsable de cambios relevantes.
- **Usabilidad:** pantallas claras para operaciones frecuentes del negocio.
- **Mantenibilidad:** módulos desacoplados, convenciones de código y documentación de decisiones.
- **Extensibilidad:** integraciones externas reemplazables sin alterar el dominio central.
- **Pruebas:** cobertura de reglas críticas, especialmente no duplicación de importaciones y prevención de stock negativo.
- **Respaldo:** definir procedimientos para copias de seguridad y restauración antes de operar con datos reales.

## 9. Decisiones pendientes antes del desarrollo

- ¿Actualmente TUU registra **productos concretos** o solo el **monto cobrado**?
- ¿BookStop será la fuente principal de verdad del inventario o deberá sincronizarlo con otro sistema?
- ¿Existen códigos de barras en todos los productos, solo en algunos o en ninguno?
- ¿Es necesario distinguir variaciones de productos (por ejemplo, color, tamaño o edición)?
- ¿Se registrarán ventas en efectivo, transferencia y tarjeta desde el inicio?
- ¿Las ventas se ingresarán al momento de vender o se cargarán posteriormente?
- ¿Se necesita registrar devoluciones y ajustes en la primera versión?
- ¿Cuántos usuarios operarán simultáneamente la aplicación?
- ¿Qué datos históricos existen para alimentar las métricas?
- ¿La aplicación se usará solo en un computador o también desde teléfonos y otros equipos?

Estas preguntas deben resolverse en el levantamiento de requisitos; no deben asumirse respuestas arbitrarias para acelerar la implementación.

## 10. Enfoque de desarrollo asistido por agentes de IA

Este proyecto también busca practicar un flujo de desarrollo de software asistido por agentes, manteniendo la responsabilidad técnica sobre el resultado.

Proceso propuesto:

1. Definir requisitos, reglas de negocio y criterios de aceptación.
2. Diseñar modelo de datos, límites de módulos y contratos de API.
3. Documentar las decisiones arquitectónicas y sus alternativas.
4. Dividir el trabajo en tareas pequeñas, con archivos y objetivos delimitados.
5. Utilizar agentes para implementar cambios controlados.
6. Revisar el código generado, ejecutar pruebas y corregir desviaciones.
7. Registrar decisiones, errores relevantes y aprendizajes durante el desarrollo.
8. Preparar una demostración funcional y documentación de instalación y uso.

Se priorizará demostrar **comprensión de la solución, calidad técnica y capacidad de validación**, no simplemente volumen de código generado.

## 11. Documentación prevista

La documentación del repositorio podrá organizarse progresivamente de la siguiente forma:

```text
bookstop/
├── README.md
├── descripcion.md
├── AGENTS.md
├── docs/
│   ├── requisitos-funcionales.md
│   ├── requisitos-no-funcionales.md
│   ├── reglas-negocio.md
│   ├── arquitectura.md
│   ├── modelo-datos.md
│   └── decisiones/
├── src/
└── tests/
```

La estructura de carpetas del código podrá cambiar según el frontend/backend elegidos (por ejemplo, un monorepositorio con `apps/`), pero la documentación del problema y de las decisiones debe permanecer accesible.

---

**Nota:** Este documento describe una propuesta inicial. Su función es orientar el diseño, discutir el alcance y establecer una base común antes de redactar las especificaciones detalladas y comenzar la implementación.
