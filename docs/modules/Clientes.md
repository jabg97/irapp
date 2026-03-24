# Clientes Module

## Purpose
The `Clientes` module manages the customers within the application. It acts as an abstraction built on top of the generic `Persona` model. A client makes requests (`solicitudes`), places orders (`ordenes`), and purchases products (`ventas`). They can also add services and products to a favorite or interested list (represented by many-to-many relationships).

## Routes
All `clientes` routes are grouped under the `/clientes` prefix.
- `GET /clientes/deleted` -> `Contacto\Cliente\ClienteSoftDeleteController@index` (List soft-deleted clients).
- `PUT /clientes/deleted/{id}` -> `Contacto\Cliente\ClienteSoftDeleteController@update` (Restore soft-deleted client).
- `DELETE /clientes/deleted/{id}` -> `Contacto\Cliente\ClienteSoftDeleteController@destroy` (Force delete client).
- `GET /clientes` -> `Contacto\Cliente\ClienteController@index` (List all clients).
- `GET /clientes/create` -> `Contacto\Cliente\ClienteController@create` (Show create form).
- `POST /clientes` -> `Contacto\Cliente\ClienteController@store` (Store new client).
- `GET /clientes/{id}` -> `Contacto\Cliente\ClienteController@show` (Show single client).
- `GET /clientes/{id}/edit` -> `Contacto\Cliente\ClienteController@edit` (Show edit form).
- `PUT /clientes/{id}` -> `Contacto\Cliente\ClienteController@update` (Update client).
- `DELETE /clientes/{id}` -> `Contacto\Cliente\ClienteController@destroy` (Soft delete client).

## Model
- **Table Name**: `clientes`
- **Main Columns**:
  - `persona_id` (foreign key)
- **Relationships**:
  - `persona()`: BelongsTo `Persona` (from the `Contacto` domain).
  - `servicios()`: BelongsToMany `Servicio` (from the `Actividad` domain).
  - `productos()`: BelongsToMany `Producto` (from the `Comercio` domain).
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Hardcoded checks ensure only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can view, list, create, and edit clients from the backend.
- **Composition**: A `Cliente` is fundamentally just a link table tying a `Persona` to `productos`, `servicios`, `ordenes`, and `ventas`. When creating a new client from the backend, the controller attempts to orchestrate the creation of a `User`, a `Persona`, a `Direccion`, an `Ubicacion`, and the `Cliente` wrapper simultaneously, passing nested empty models to the view.
- **Interest Graph**: Clients have a Many-to-Many relationship with `Servicio` and `Producto`. This is likely used for shopping carts, favorites, or wishlist features.

## Database
- **Migrations**: `2014_10_12_000011_create_clientes_table.php`, `2014_10_12_000021_create_cliente_producto_table.php`, `2014_10_12_000025_create_cliente_servicio_table.php`.
- **Foreign Keys cascade**: `persona_id` is configured with `onDelete('cascade')`.

## Things to be careful about
- **Destructive Cascades**: Like products, deleting a `Persona` will cascade down and completely delete the associated `Cliente` record, bypassing Eloquent SoftDeletes.
- **Complex Orchestration**: The creation/update process in `ClienteController` handles many layers of related models (`User`, `Persona`, `Direccion`, `Ubicacion`). Any validation failure or exception in this deep creation chain might result in orphaned records (e.g., a `User` created, but `Cliente` fails to save) unless DB transactions are explicitly used in the controller's `store`/`update` methods.
