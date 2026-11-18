# Capacitaciones Naturisa

Pantalla para planificar y registrar las capacitaciones de las aplicaciones **Parámetros**,
**Cosecha Diaria** y **Combustible** en las 27 camaroneras.

El cronograma y los asistentes se guardan en **Cloudflare D1**, así que todos los que usan
la pantalla ven y escriben lo mismo.

## Cómo se usa

Doble clic en `iniciar.bat` (o `python server.py index.html 3014`) y se abre en el navegador.
Pide el usuario de **AP1**: con esa sesión se consultan los nombres en RRHH y se guarda en D1.

### Cronograma
- **Vista Resumen**: una fila por aplicación con sus tres etapas (capacitación, soporte,
  seguimiento) y, en cada día, el código de las fincas que tocan (`PE`, `MA`, `A1`…).
- **Vista Detalle**: una fila por actividad, agrupable por aplicación, sucursal o cluster.
- Un clic en un día lo da por completo, arrastrarlo lo reprograma, doble clic abre la actividad.

### Asistentes
Se digita la cédula y el nombre, el cargo y el área salen de RRHH. Si la persona no está en
RRHH (o no hay conexión), los datos se escriben a mano. Una misma persona puede quedar
registrada en varias aplicaciones de una sola vez, y la tabla lleva un listado por aplicación.

### Configuración
Una fila por sucursal con la fecha de capacitación de cada aplicación. Solo esa fecha se edita:
el **soporte** (5 días hábiles desde el día siguiente) y el **seguimiento** (otros 5, pegados al
soporte) se calculan solos. Desde ahí también salen el Excel y el respaldo en JSON.

## Datos

| Dónde | Qué |
|---|---|
| D1 `capacitaciones_plan` | el cronograma: una fila por sucursal × aplicación × etapa |
| D1 `capacitaciones_asistentes` | una fila por persona × aplicación × sesión |
| `localStorage` | copia local de lo anterior, para seguir viendo la pantalla sin conexión |

Las tablas están en [`schema_capacitaciones.sql`](schema_capacitaciones.sql). Si algo no se
puede guardar (sin red), queda pendiente, el encabezado lo avisa y se reintenta al volver a entrar.

## Servicios que usa

Todo pasa por el Worker `naturisa-proxy` (repositorio `reportes-naturisa`), que es quien habla
con D1, con AP1 y con RRHH:

| Ruta | Para qué |
|---|---|
| `POST /d1/usuarios/login` | entrar con el usuario de AP1 (sesión firmada) |
| `GET /d1/usuarios/consulta?cedula=` | nombre, cargo y área desde RRHH |
| `GET /d1/capacitaciones` | cronograma y asistentes |
| `PUT /d1/capacitaciones/plan` | guardar actividades |
| `POST·DELETE /d1/capacitaciones/asistentes` | registrar o quitar asistentes |
| `GET /db/sucursales` | catálogo de camaroneras y sus códigos |

En local el `server.py` hace de proxy a ese Worker, porque el Worker no acepta CORS desde
todos los puertos de `localhost`.

## Archivos

- `index.html` — la pantalla completa (sin dependencias; usa SheetJS por CDN para el Excel)
- `server.py` — servidor local y proxy al Worker
- `iniciar.bat` — lo arranca en el puerto 3014
- `crear_icono.py` / `CAP.ico` — icono del acceso directo
- `schema_capacitaciones.sql` — las dos tablas de D1
