# Registro de Incidencias y Comprobaciones de Instalación (errores-instalacion.md)

En este documento se recogen las comprobaciones preventivas realizadas durante la puesta en marcha de Odoo con Docker
Compose para evitar fallos de despliegue, así como la reproducción, diagnóstico y resolución de los errores típicos más
comunes en este tipo de infraestructura.

---

## 1. Tabla de Incidencias, Comprobaciones y Errores Típicos

| Error o Mensaje Exacto                                                                                                               | Tipo                            | Causa Raíz                                                                                                                                                             | Solución Aplicada                                                                                                                                                                                                                                             | Fuente Consultada                                             |
|:-------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------|
| `Bind for 0.0.0.0:8200 failed: port is already allocated` / `listen tcp4 0.0.0.0:8200: bind: address already in use`                 | **Problema Típico Reproducido** | Conflicto de socket de red: el puerto `8200` (o `8069`) del anfitrión ya está ocupado por otro servicio local o por un contenedor previo no finalizado.                | 1. Identificar el proceso en PowerShell: `netstat -ano \| findstr :8200`.<br>2. Detener contenedores colgados: `docker-compose down`.<br>3. Alternativa: remapear a otro puerto en `docker-compose.yaml` (ej. `8201:8069`).                                   | Documentación de Docker Engine (*Networking & Port Binding*). |
| `could not connect to server: Connection refused. Is the server running on host "db" and accepting TCP/IP connections on port 5432?` | **Problema Típico Reproducido** | Odoo intenta iniciar el pool de conexiones antes de que PostgreSQL complete la inicialización del cluster, o el hostname `db` no resuelve en la red interna de Docker. | 1. Comprobar que ambos contenedores comparten la red `odoo_default`.<br>2. Incluir la directiva `depends_on: - db` en el servicio Odoo dentro de `docker-compose.yaml`.<br>3. Verificar credenciales (`USER`, `PASSWORD`) coincidentes entre ambos servicios. | Documentación oficial de Odoo (*Deploying Odoo with Docker*). |
| `Internal Server Error` (Código HTTP 500) al abrir el navegador                                                                      | **Problema Típico Reproducido** | Permisos de lectura/escritura denegados en los volúmenes montados (`./config` o `./addons`), o sintaxis incorrecta en la directiva `addons_path` de `odoo.conf`.       | 1. Inspeccionar logs del contenedor con `docker-compose logs odoo`.<br>2. Corregir rutas en `odoo.conf` asegurando la sintaxis `/mnt/extra-addons`.<br>3. Asignar permisos adecuados de lectura en el directorio local sincronizado.                          | Foros oficiales de Odoo Community & Bugtracker de Odoo.       |
| `Virtualization support not detected` / Fallo de arranque del motor de Docker                                                        | **Comprobación Preventiva**     | La virtualización asistida por hardware (SVM Mode en AMD o Intel VT-x) o el motor de WSL 2 no están habilitados en el sistema anfitrión.                               | Verificación previa en Administrador de Tareas (Rendimiento > CPU) de que la virtualización está activa y actualización del subsistema mediante terminal: `wsl --update`.                                                                                     | Documentación técnica de Docker Desktop para Windows.         |
| Detención repentina del contenedor Odoo (`Exit Code 137` / *OOM Killed*)                                                             | **Comprobación Preventiva**     | Asignación de memoria RAM insuficiente en el entorno virtual de WSL 2 para ejecutar simultáneamente PostgreSQL y Odoo.                                                 | Configuración preventiva del archivo `%USERPROFILE%\.wslconfig` estableciendo un límite de memoria RAM adecuado (mínimo 4 GB - 6 GB).                                                                                                                         | Microsoft Learn (*Advanced settings configuration in WSL*).   |

---

## 2. Comprobaciones Preventivas Realizadas

Para asegurar que el despliegue inicial se completara sin interrupciones ni bloqueos de ejecución, se aplicaron las
siguientes validaciones previas al comando de arranque:

1. **Disponibilidad de puertos en el sistema anfitrión:**
   Antes de levantar los contenedores, se comprobó que los puertos `8069` y `8200` estuvieran libres en PowerShell:
   ```powershell
   Get-NetTCPConnection -LocalPort 8200 -ErrorAction SilentlyContinue
   Get-NetTCPConnection -LocalPort 8069 -ErrorAction SilentlyContinue
   ```
   Al no devolver ningún PID activo, se garantizó que el enlace de red de Docker se crearía sin colisiones.

2. **Integridad de directorios y montajes (*Bind Mounts*):**
   Se verificó que la carpeta del proyecto contara con la estructura correcta antes de ejecutar el arranque:
    * `docker-compose.yaml` en la raíz.
    * Carpeta `./config` conteniendo el archivo `odoo.conf` con los parámetros base.
    * Carpeta `./addons` vacía y preparada para recibir módulos personalizados.

3. **Compatibilidad del motor de virtualización:**
   Se validó que Docker Desktop estuviera operando sobre el backend nativo de WSL 2 con una distribución funcional
   (`wsl -l -v`), evitando problemas de rendimiento derivados de Hyper-V tradicional.

---

## 3. Reproducción y Resolución de Problemas Típicos de Odoo

### Caso 1: Conflicto de socket / Puerto ocupado (`port is already allocated`)

* **Descripción del problema:** Al ejecutar `docker-compose up -d`, la consola cancela el despliegue con el siguiente
  error:
  ```text
  Error response from daemon: driver failed programming external connectivity on endpoint odoo:
  Bind for 0.0.0.0:8200 failed: port is already allocated
  ```
* **Diagnóstico técnico:** Un proceso previo (u otra instancia de un contenedor detenido de forma forzada) sigue
  reteniendo el socket TCP en el puerto `8200`.
* **Procedimiento de resolución:**
    1. Identificar el identificador de proceso (PID) que bloquea el puerto:
       ```powershell
       netstat -ano | findstr :8200
       ```
    2. Bajar limpiamente los servicios y redes de Docker vinculadas al stack:
       ```powershell
       docker-compose down --remove-orphans
       ```
    3. Relanzar el stack en segundo plano:
       ```powershell
       docker-compose up -d
       ```
  *(Nota técnica: En caso de requerir mantener el otro servicio local en ejecución, la solución consiste en editar
  `docker-compose.yaml` y cambiar el puerto expuesto a `8201:8069`).*

---

### Caso 2: Fallo de conexión de Odoo con PostgreSQL (`Connection Refused`)

* **Descripción del problema:** El contenedor de Odoo entra en bucle de reinicio (*Restarting*) o finaliza
  inmediatamente. Al consultar los registros (`docker-compose logs odoo`), aparece:
  ```text
  psycopg2.OperationalError: could not connect to server: Connection refused
  Is the server running on host "db" and accepting TCP/IP connections on port 5432?
  ```
* **Diagnóstico técnico:** Odoo arranca más rápido que el proceso de inicialización de tablas de PostgreSQL, o las
  variables de entorno de conexión (`USER`, `PASSWORD`, `HOST`) no coinciden entre ambos servicios.
* **Procedimiento de resolución:**
    1. Verificar que el servicio `db` está en ejecución y sin errores en sus logs:
       ```powershell
       docker-compose logs db
       ```
    2. Asegurar en `docker-compose.yaml` la dependencia explícita:
       ```yaml
       services:
         web:
           image: odoo:latest
           depends_on:
             - db
           environment:
             - HOST=db
             - USER=odoo
             - PASSWORD=odoo
       ```
    3. Reiniciar el stack para forzar el orden correcto de arranque.

---

### Caso 3: Error 500 (*Internal Server Error*) en la interfaz web

* **Descripción del problema:** El navegador accede a `http://localhost:8200`, pero en lugar del formulario de inicio o
  del gestor de base de datos, devuelve un error HTTP 500 o una pantalla en blanco.
* **Diagnóstico técnico:** El archivo de configuración `odoo.conf` tiene una ruta errónea en el parámetro `addons_path`
  o el contenedor no tiene permisos de lectura sobre los archivos montados desde Windows.
* **Procedimiento de resolución:**
    1. Revisar los últimos registros de error de la aplicación:
       ```powershell
       docker-compose logs --tail=50 odoo
       ```
    2. Comprobar que en `./config/odoo.conf` la ruta interna hacia los módulos extra coincide exactamente con el montaje
       del volumen:
       ```ini
       [options]
       addons_path = /mnt/extra-addons
       data_dir = /var/lib/odoo
       ```
    3. Reiniciar el contenedor de Odoo para recargar el archivo de configuración:
       ```powershell
       docker-compose restart odoo
       ```