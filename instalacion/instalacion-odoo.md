# Guía de Instalación de Odoo con Docker

---

## 1. Versión, Edición y Tipo de Instalación

* **Versión y edición:** Odoo Community Edition (v17.0 / `odoo:latest`), de código abierto bajo licencia GNU LGPLv3.
* **Tipo de instalación:** Despliegue en contenedores mediante **Docker** y **Docker Compose**.
* **¿Por qué se ha elegido esta instalación?:**
    * **Orquestación simplificada:** Permite definir, agrupar y enlazar en un único archivo `docker-compose.yaml` tanto
      el contenedor de la aplicación (**Odoo**) como el contenedor de su base de datos obligatoria (**PostgreSQL**).
    * **Aislamiento y portabilidad:** Incluye en las imágenes todas las dependencias y librerías necesarias sin alterar
      ni generar conflictos en el sistema operativo anfitrión.
    * **Persistencia y modularidad:** Garantiza que los datos no se pierdan mediante el uso de volúmenes y facilita la
      personalización sincronizando las carpetas locales `./config` y `./addons`.
    * **Ventajas del ERP abierto:** Coste de implementación accesible sin pago de licencias propietarias, alta
      flexibilidad de adaptación a la empresa (**FitZone**) y soporte activo de la comunidad.

---

## 2. Requisitos de Hardware y Software

| Categoría    | Requisito                 | Detalle Técnico                                                                             |
|:-------------|:--------------------------|:--------------------------------------------------------------------------------------------|
| **Hardware** | **Procesador (CPU)**      | Arquitectura de 64 bits (`x86_64`) con soporte de virtualización activado en BIOS/UEFI.     |
| **Hardware** | **Memoria RAM**           | Mínimo 4 GB de RAM (8 GB recomendados para ejecutar Windows, WSL 2 y los contenedores).     |
| **Hardware** | **Almacenamiento**        | Espacio libre en disco SSD para Docker Desktop, imágenes (`odoo` y `postgres`) y volúmenes. |
| **Software** | **Sistema Operativo**     | Windows 10 o Windows 11 de 64 bits.                                                         |
| **Software** | **Virtualización en SO**  | Subsistema de Windows para Linux 2 (**WSL 2**) habilitado como backend.                     |
| **Software** | **Motor de Contenedores** | **Docker Desktop for Windows (x86_64)** instalado y en ejecución.                           |

---

## 3. Esquema Visual de la Arquitectura en Docker

~~~text
+-------------------------------------------------------------------------+
|                        SISTEMA OPERATIVO ANFITRION                      |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |                              DOCKER                               |  |
|  |                                                                   |  |
|  |  1. IMAGENES (Plantillas con aplicacion, dependencias y librerias)|  |
|  |     [ Imagen Odoo ]                        [ Imagen Postgres ]    |  |
|  |            |                                        |             |  |
|  |            v                                        v             |  |
|  |  2. CONTENEDORES (Unidos mediante docker-compose.yaml)            |  |
|  |     +-----------------------+      Red     +-------------------+  |  |
|  |     | Contenedor Odoo (odoo)| <----------> | Contenedor BD (db)|  |  |
|  |     | Puerto: 8200:8069     |              | PostgreSQL        |  |  |
|  |     +-----------+-----------+              +---------+---------+  |  |
|  |                 |                                    |            |  |
|  |                 v                                    v            |  |
|  |  3. VOLUMENES Y DIRECTORIOS (Persistencia de informacion)         |  |
|  |     - Volumen: odoo-web-data (/var/lib/odoo)                      |  |
|  |     - Volumen: odoo-db-data  (Datos de PostgreSQL)                |  |
|  |     - Carpeta: ./config      <--> /etc/odoo (archivo odoo.conf)   |  |
|  |     - Carpeta: ./addons      <--> /mnt/extra-addons (modulos)     |  |
|  +-------------------------------------------------------------------+  |
+-------------------------------------------------------------------------+
~~~

### Atributos principales de `docker-compose.yaml`

* **`container_name`:** Nombre asignado a cada contenedor.
* **`image`:** Imagen base utilizada (`odoo` y `postgres`).
* **`ports`:** Expone el puerto interno `8069` de Odoo a través del puerto externo en el servidor/anfitrión
  (`8200:8069`).
* **`volumes`:** Enlaza los volúmenes de datos (`odoo-web-data` y `odoo-db-data`) y sincroniza los directorios locales
  `./config` (que contiene `odoo.conf` con la ruta `addons_path`) y `./addons` (módulos extra en `/mnt/extra-addons`).
* **`environment`:** Define los parámetros y variables de entorno de conexión entre los contenedores.

---

## 4. Pasos Principales de Instalación en Orden

~~~text
[1. Instalar Docker] --> [2. Descargar YAML] --> [3. Carpeta odoo] --> [4. docker-compose up -d] --> [5. Verificar y Entrar]
~~~

1. **Descargar e instalar Docker Desktop:**
    * Acceder a la web oficial (`https://docs.docker.com/desktop/install/windows-install/`) y descargar **Docker Desktop
      for Windows - x86_64**.
    * Ejecutar el instalador dejando marcada la opción recomendada **Use WSL 2 instead of Hyper-V (recommended)**.
    * Abrir Docker Desktop, aceptar los términos de servicio (*Subscription Service Agreement*) e iniciar sesión con la
      cuenta de usuario.

2. **Obtener el archivo `docker-compose.yaml`:**
    * Acceder al repositorio de referencia (`https://github.com/jmlcas/odoo` a través de la guía
      `https://labarta.es/como-instalar-odoo-usando-docker-compose/`).
    * Hacer clic en el botón **Code** y seleccionar **Download ZIP**.

3. **Preparar el directorio de trabajo:**
    * Extraer el contenido del archivo ZIP dentro de la carpeta `Documentos`.
    * Renombrar la carpeta resultante con el nombre **`odoo`**, verificando que en su interior se encuentran el archivo
      `docker-compose.yaml` y las carpetas `config` y `addons`.

4. **Levantar los contenedores desde la terminal:**
    * Abrir **Windows PowerShell** en la ruta de la carpeta `odoo` y ejecutar los siguientes comandos:

~~~powershell
cd .\odoo\
docker-compose up -d
~~~

* Esperar a que finalice la descarga de las imágenes (*Pull complete*) y se creen la red (`odoo_default`), los volúmenes
  (`odoo_db-data`, `odoo_odoo-data`) y los contenedores (`db` y `odoo`) en estado **Started**.

5. **Comprobar el estado en Docker Desktop:**
    * Abrir **Docker Desktop** e ir al apartado **Containers**.
    * Verificar que el grupo `odoo` muestra sus 2 contenedores (`db` y `odoo`) activos en estado **Running** (en color
      verde). En caso de estar detenidos, pulsar el botón **Play (Start)**.

---

## 5. Puerto, URL y Credenciales de Acceso

| Parámetro                   | Valor                   | Explicación                                                                                           |
|:----------------------------|:------------------------|:------------------------------------------------------------------------------------------------------|
| **Puerto interno de Odoo**  | `8069`                  | Puerto en el que escucha el servicio dentro del contenedor.                                           |
| **Puerto expuesto en Host** | `8200`                  | Puerto local mapeado en Docker (`8200:8069`).                                                         |
| **URL de acceso**           | `http://localhost:8200` | Dirección web para abrir Odoo desde el navegador (o haciendo clic sobre el puerto en Docker Desktop). |
| **Usuario por defecto**     | `admin`                 | Credencial inicial de administración indicada en el repositorio.                                      |
| **Contraseña por defecto**  | `admin`                 | Clave inicial de acceso al sistema.                                                                   |