# Documentación del Ciclo de Ventas: FitZone S.L.

Este documento describe la ejecución y verificación del flujo completo de ventas (*Quote to Cash*) implementado en Odoo
18 para la empresa **FitZone S.L.**, demostrando la segregación de funciones (*Segregation of Duties - SoD*), el control
de inventario y la trazabilidad documental entre los distintos departamentos.

---

## 1. Matriz de Roles y Responsabilidades

El ciclo comercial involucra a cuatro entidades diferenciadas para garantizar la auditoría y el principio de mínimo
privilegio:

| Rol                              | Usuario             | Cuenta / Identificador       | Operación asignada                                                        |
|:---------------------------------|:--------------------|:-----------------------------|:--------------------------------------------------------------------------|
| **Comercial / Recepción**        | Laura Gómez         | `laura.comercial@fitzone.es` | Creación del presupuesto inicial y aplicación de condiciones comerciales. |
| **Responsable de Ventas**        | David               | `david@fitzone.es` (Admin)   | Supervisión y confirmación del pedido de venta (`sale.order`).            |
| **Almacén / Logística**          | Sistema / David     | Operaciones internas         | Validación del albarán de entrega y salida física de existencias.         |
| **Administración / Facturación** | Elena Martín        | `administracion@fitzone.es`  | Generación del borrador, publicación de factura y conciliación del cobro. |
| **Cliente (Socio)**              | Carlos Mendoza Ruiz | `carlos.mendoza@email.com`   | Receptor del servicio/productos y pagador.                                |

---

## 2. Desarrollo Paso a Paso del Ciclo Comercial

```text
 [Presupuesto S00025] ──► [Pedido Confirmado] ──► [Albarán My Co/OUT/00001] ──► [Factura INV/2026/00001] ──► [Cobro Registrado]
   (Laura Gómez)              (David)                  (Inventario)                 (Elena Martín)               (Elena Martín)
```

### Paso 1: Creación del Presupuesto (Laura Gómez)

* **Documento generado:** `S00025`
* **Cliente:** Carlos Mendoza Ruiz (Avenida de las Ciencias 28, 3º B, 41020 Sevilla, NIF: `47891234K`)
* **Condiciones:**
    * **Fecha de vencimiento:** 6 de diciembre
    * **Condiciones de pago:** Pago inmediato
* **Desglose de líneas:**

| Producto                                |    Tipo     | Cantidad | Precio unitario | Impuestos | Descuento |  Subtotal   |
|:----------------------------------------|:-----------:|:--------:|:---------------:|:---------:|:---------:|:-----------:|
| `[CUOTA-01] Cuota Mensual FitZone`      |  Servicio   |   1,00   |     39,99 €     |  IVA 21%  |  0,00 %   |   39,99 €   |
| `[PROT-01] Bote Proteína Whey 1kg`      | Almacenable |   1,00   |     30,00 €     |  IVA 21%  |  0,00 %   |   30,00 €   |
| `[TOALLA-01] Toalla Microfibra FitZone` | Almacenable |   1,00   |     12,00 €     |  IVA 21%  |  20,00 %  |   9,60 €    |
| **Total Base Imponible**                |             |          |                 |           |           | **79,59 €** |
| **Impuestos (IVA 21%)**                 |             |          |                 |           |           | **16,71 €** |
| **Total Pedido**                        |             |          |                 |           |           | **96,30 €** |

* **Evidencia documental:** `venta/presupuesto.pdf`

---

### Paso 2: Validación y Confirmación de Pedido (David)

* **Documento de origen:** `S00025`
* **Acción ejecutada:** El responsable de ventas audita las condiciones comerciales y presiona el botón **Confirmar**.
* **Efecto en el sistema:**
    * El documento pasa formalmente de estado *Presupuesto* a **Pedido de venta**.
    * Se bloquean las líneas de venta frente a ediciones no autorizadas.
    * Odoo dispara el aprovisionamiento automático, reservando el stock de los productos físicos y generando la orden de
      entrega asociada.

---

### Paso 3: Expedición y Control de Stock (Inventario)

* **Documento de entrega (Albarán):** `My Co/OUT/00001`
* **Tipo de operación:** Órdenes de entrega
* **Líneas gestionadas:**
    * `[PROT-01] Bote Proteína Whey 1kg`: Demanda = 1,00 | Realizado = 1,00
    * `[TOALLA-01] Toalla Microfibra FitZone`: Demanda = 1,00 | Realizado = 1,00 *(La cuota mensual queda excluida de la
      orden logística por ser un producto de naturaleza intangible/servicio).*
* **Validación:**
    * Se confirma la entrega de material marcando las cantidades como hechas y validando el movimiento.
    * Estado final del albarán: **Hecho**.
    * **Impacto en inventario:** Comprobación en la ficha de producto de que el stock a mano se reduce en una unidad por
      artículo entregado.

---

### Paso 4: Emisión de Factura y Cobro (Elena Martín)

* **Documento fiscal:** `INV/2026/00001`
* **Diario de imputación:** Facturas de clientes / Ventas FitZone
* **Plan contable aplicado:** Plan General Contable Español para Pymes (2008)
* **Secuencia de operaciones:**
    1. **Generación del borrador:** Elena Martín accede al pedido `S00025` tras la entrega y ejecuta la acción **Crear
       factura** (Factura regular).
    2. **Comprobación y validación:** Se revisa la asignación a las cuentas del subgrupo 70 (Ventas de mercaderías e
       ingresos por servicios) y la cuota de IVA repercutido (477).
    3. **Publicación:** Confirmación de la factura, adquiriendo la numeración oficial `INV/2026/00001` y pasando de
       estado *Borrador* a **Registrado / Publicado**.
    4. **Registro de pago:**
        * **Método de cobro:** Banco / Efectivo
        * **Importe liquidado:** 96,30 €
        * **Estado final:** **Pagado**
* **Evidencia documental:** `venta/factura.pdf`

---

## 3. Trazabilidad Documental y Auditoría

La arquitectura relacional de Odoo vincula la totalidad de los asientos e identificadores técnicos generados a lo largo
del proceso:

```text
[Presupuesto / Pedido] S00025
       │
       ├──► [Albarán de Salida] My Co/OUT/00001 (Estado: Hecho)
       │           └── Movimiento físico de stock: -1 Ud. Proteína / -1 Ud. Toalla
       │
       └──► [Factura de Cliente] INV/2026/00001 (Estado: Pagado)
                   └── Asiento Contable y Registro de Cobro: 96,30 €
```

### Conclusiones sobre la implantación

* **Aislamiento de tareas:** Ningún usuario concentró la totalidad del ciclo. Laura generó la propuesta comercial sin
  acceso a contabilidad, David supervisó la orden y Elena consolidó el cobro fiscal sin modificar condiciones de venta.
* **Integridad transaccional:** No fue posible facturar ni alterar existencias sin una orden comercial validada y una
  verificación de albarán previa.
* **Configuración fiscal:** La carga de la localización española (`España - Pymes 2008`) permitió estructurar
  correctamente el diario de ventas (`sale`) y las cuentas predeterminadas de ingresos, resolviendo la emisión legal de
  la factura rectificable y computable.