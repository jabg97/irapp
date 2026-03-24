# Solicitudes Module

## Purpose
The `Solicitudes` module manages the creation and lifecycle of client requests for services. A `Solicitud` is a precursor to an `Orden`. When a client makes a request, it remains pending or open until an administrator approves it, at which point it is converted into an actionable `Orden`.

## Routes
All `solicitudes` routes are mapped individually under the `/solicitudes` prefix.
- `GET /solicitudes/index/{estado?}` -> `Actividad\Solicitud\SolicitudController@index` (List all requests, optionally filtered by state).
- `GET /solicitudes/{id}` -> `Actividad\Solicitud\SolicitudController@show` (Show details of a specific request).
- `POST /solicitudes/{id}` -> `Actividad\Solicitud\SolicitudController@approve` (Approve a request and convert it to an order).
- `PUT /solicitudes/{id}` -> `Actividad\Solicitud\SolicitudController@cancel` (Cancel a request).

## Model
- **Table Name**: `solicitudes`
- **Main Columns**:
  - `nombre` (string)
  - `estado` (enum: 'Abierta', 'Cancelada', 'Pendiente')
  - `fecha_inicio` (datetime)
  - `fecha_fin` (datetime, nullable)
  - `direccion_id` (foreign key)
  - `cliente_id` (foreign key)
- **Relationships**:
  - `direccion()`: BelongsTo `Direccion`.
  - `cliente()`: BelongsTo `Cliente`.
  - `detalles()`: HasMany `Detalle_solicitud`.
- **Note**: Unlike `Orden` and `Servicio`, the `Solicitud` model **does not use SoftDeletes**.

## Business Logic
- **Authorization**: Only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can view the list of requests.
- **Conversion to Order**: The `approve` method takes a `Solicitud` and transforms it into an `Orden`. This involves copying data from `solicitudes` to `ordenes` and from `detalles_solicitudes` to `detalles_ordenes`. The frontend UI indicates this interaction but the backend performs the hard state transition.
- **Dynamic Enums**: Similar to `Ordenes`, `Solicitudes` extracts Enum values from the database using raw SQL `SHOW COLUMNS`.
- **JSON Formatting**: The `index` method generates a large `JSON_solicitudes` JSON string via raw SQL for rendering a calendar (mapping states to specific hex colors and icons).

## Database
- **Migrations**: `2014_10_12_000029_create_solicitudes_table.php`, `2014_10_12_000030_create_detalles_solicitudes_table.php`.
- **Raw Queries**: Raw queries are used for calendar payload aggregation (handling SQL `CASE` statements to inject hardcoded colors) and to fetch `enum` possibilities directly from the schema.

## Things to be careful about
- **Schema Coupling**: Changes in table column names or Enum values will break the custom UI rendering logic in the controller because the raw SQL maps directly to specific table names (`direcciones.barrio AS barrio`).
- **No Soft Deletes**: Deleting a `Solicitud` or a `Detalle_solicitud` is a permanent physical delete, unlike most other entities in the system.
- **State Coupling with `Orden`**: Modifying the lifecycle states in `Solicitudes` might require modifying the conversion logic to `Ordenes` inside the `approve` method.
