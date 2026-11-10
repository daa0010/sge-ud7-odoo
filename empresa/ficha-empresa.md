# Ficha de Empresa: FitZone S.L.

Este documento recoge los datos generales, la actividad comercial y el catálogo base configurado en Odoo 18 para la
empresa del proyecto.

---

## 1. Datos Generales

* **Nombre Comercial:** FitZone
* **Razón Social:** FitZone S.L.
* **Sector:** Salud, Bienestar y Deporte
* **Actividad Principal:** Gestión de instalaciones deportivas, prestación de servicios de entrenamiento y venta al por
  menor de suplementación y accesorios fitness.

---

## 2. Datos Fiscales y de Contacto

Estos son los datos exactos configurados en los ajustes de la compañía en Odoo:

* **NIF:** B98765432
* **Dirección:** Calle Innovación 14, Polígono Industrial Este
* **Código Postal:** 41020
* **Ciudad:** Sevilla
* **Provincia:** Sevilla (ES)
* **País:** España
* **Teléfono:** +34 954 12 34 56
* **Correo Electrónico:** contacto@fitzone.es
* **Sitio Web:** https://www.fitzone.es
* **Moneda:** EUR

---

## 3. Catálogo de Productos y Servicios

El catálogo inicial dado de alta en el módulo de Inventario/Ventas se divide en dos categorías según su naturaleza
logística e impositiva. Todos los artículos tienen aplicado por defecto un 21% de IVA.

### Servicios (Intangibles)

No requieren gestión de inventario ni generan albaranes de entrega física.

| Referencia Interna | Nombre del Producto                 | Precio de Venta | Categoría |
|:-------------------|:------------------------------------|:----------------|:----------|
| `[CUOTA-01]`       | Cuota Mensual FitZone               | 39,99 €         | Services  |
| `[BONO-05]`        | Bono 5 Sesiones Entrenador Personal | 120,00 €        | Services  |

### Productos Almacenables (Físicos)

Requieren control de stock, se descuentan del inventario al validar las salidas y generan albaranes logísticos.

| Referencia Interna | Nombre del Producto       | Precio de Venta | Categoría |
|:-------------------|:--------------------------|:----------------|:----------|
| `[PROT-01]`        | Bote Proteína Whey 1kg    | 30,00 €         | Goods     |
| `[TOALLA-01]`      | Toalla Microfibra FitZone | 12,00 €         | Goods     |

---

## 4. Cartera de Clientes

Para validar los flujos de facturación y cumplir con los requisitos operativos, se han dado de alta clientes de distinta
naturaleza:

### Cliente Particular (B2C)

* **Nombre:** Carlos Mendoza Ruiz
* **Tipo:** Individuo
* **NIF:** 47891234K
* **Contacto:** carlos.mendoza@email.com / +34 612 34 56 78
* **Dirección:** Avenida de las Ciencias 28, 3º B, 41020 Sevilla, España
* **Nota:** Configurado como cliente final y usuario del portal web.

### Cliente Empresa (B2B)

* **Razón Social:** InnovaTech Soluciones S.L.
* **Tipo:** Compañía
* **NIF:** B41987654
* **Contacto:** administracion@innovatech.es / +34 954 99 88 77
* **Dirección:** Parque Tecnológico Cartuja, Calle Leonardo da Vinci 8, 41092 Sevilla, España
* **Sitio Web:** https://www.innovatech.es