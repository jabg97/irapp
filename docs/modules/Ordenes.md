# Ordenes Module

## Purpose
The `Ordenes` module handles service orders within the application. It represents a scheduled service request that a client has made, detailing when and where a service will take place, its current state, and the details (which specific services and collaborators are assigned to it).

## Routes
All `ordenes` routes are grouped under the `/ordenes` prefix.
- `GET /ordenes/index/{estado?}` -> `Actividad\Orden\OrdenController@index` (List all orders, filterable by state).
- `GET /ordenes/create/{fecha?}` -> `Actividad\Orden\OrdenController@create` (Show create form).
- `GET /ordenes/deleted/index/{estado?}` -> `Actividad\Orden\OrdenSoftDeleteController@index` (List deleted orders).
- `PUT /ordenes/deleted/{id}` -> `Actividad\Orden\OrdenSoftDeleteController@update` (Restore deleted order).
- `DELETE /ordenes/deleted/{id}` -> `Actividad\Orden\OrdenSoftDeleteController@destroy` (Force delete order).
- `GET /ordenes` -> `Actividad\Orden\OrdenController@index` (Resource fallback).
- `POST /ordenes` -> `Actividad\Orden\OrdenController@store` (Store new order).
- `GET /ordenes/{id}` -> `Actividad\Orden\OrdenController@show` (Show single order).
- `GET /ordenes/{id}/edit` -> `Actividad\Orden\OrdenController@edit` (Show edit form).
- `PUT /ordenes/{id}` -> `Actividad\Orden\OrdenController@update` (Update order).
- `DELETE /ordenes/{id}` -> `Actividad\Orden\OrdenController@destroy` (Soft delete order).
- `GET /ordenes/detalles/{id}` -> `Actividad\Orden\OrdenController@get_detalles` (Load order details).
- `GET /ordenes/detalles/form/{id}/{editar}` -> `Actividad\Orden\OrdenController@form_detalles` (Show form for order details).
- `GET /ordenes/detalles/servicios/{id}/{editar}` -> `Actividad\Orden\OrdenController@get_servicios` (Load services for order details).
- `GET /ordenes/detalles/colaboradores/{id}/{editar}` -> `Actividad\Orden\OrdenController@get_colaboradores` (Load collaborators for order details).
- `POST /ordenes/detalles/add` -> `Actividad\Orden\OrdenController@add_detalles` (Add detail to order).
- `PUT /ordenes/detalles/update/{id}` -> `Actividad\Orden\OrdenController@update_detalles` (Update order detail).
- `PUT /ordenes/detalles/state/{id}` -> `Actividad\Orden\OrdenController@state_detalles` (Update order detail state).
- `DELETE /ordenes/detalles/delete/{id}` -> `Actividad\Orden\OrdenController@delete_detalles` (Delete order detail).

## Model
- **Table Name**: `ordenes`
- **Main Columns**:
  - `nombre` (string)
  - `estado` (enum: 'Abierta', 'Cerrada', 'Cancelada', 'Pendiente')
  - `fecha_inicio` (datetime)
  - `fecha_fin` (datetime)
  - `direccion_id` (foreign key)
  - `cliente_id` (foreign key)
- **Relationships**:
  - `direccion()`: BelongsTo `Direccion`.
  - `cliente()`: BelongsTo `Cliente`.
  - `detalles()`: HasMany `Detalle_orden`.

## Business Logic
- **Authorization**: Hardcoded in controller using `$user->authorizeRoles()`. Only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can view and create orders.
- **Dynamic Enums**: The `Orden` model uses a raw SQL query (`SHOW COLUMNS FROM ordenes WHERE Field = 'estado'`) to fetch the enum values dynamically for the states.
- **JSON Payload Generation**: The `index` method generates a large `JSON_ordenes` payload via a raw DB query mapping orders directly to full address and client names, seemingly built for a Javascript calendar view (colors, start dates, font-awesome icons mapped manually in SQL).
- **Details Management**: Orders have a 1-to-N relationship with `Detalle_orden`. The controller provides many specific endpoints to add, update, and manage the state of each specific item within the order.

## Database
- **Migrations**: `2014_10_12_000027_create_ordenes_table.php`, `2014_10_12_000028_create_detalles_ordenes_table.php`.
- **Raw Queries**: Used inside the `index()` method of `OrdenController` to format data for frontend calendars (`REPLACE()`, `CASE`, explicit color codes). Another raw query is in `Orden::getEstados()`.

## Things to be careful about
- **Raw SQL Injection Risks**: The usage of raw SQL inside the `index` method, while currently safe (no user input passed directly to the `CASE` statement), makes it brittle if table columns change.
- **Hardcoded Colors**: Hex colors are hardcoded inside the `OrdenController`. Modifying the frontend theme requires updating PHP backend code.
- **Enum Parsing**: `Orden::getEstados()` parses enum definitions via regex. Changing the DB driver (e.g., from MySQL to PostgreSQL) will break this logic completely.
