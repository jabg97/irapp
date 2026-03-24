# Proveedores Module

## Purpose
The `Proveedores` module manages the suppliers of products. A provider (supplier) is an entity from which the system administrators can record inventory purchases (`Compras`). Similar to clients and collaborators, a provider is built on top of the generic `Persona` model.

## Routes
All `proveedores` routes are grouped under the `/proveedores` prefix.
- `GET /proveedores/deleted` -> `Contacto\Proveedor\ProveedorSoftDeleteController@index` (List soft-deleted providers).
- `PUT /proveedores/deleted/{id}` -> `Contacto\Proveedor\ProveedorSoftDeleteController@update` (Restore soft-deleted provider).
- `DELETE /proveedores/deleted/{id}` -> `Contacto\Proveedor\ProveedorSoftDeleteController@destroy` (Force delete provider).
- `GET /proveedores` -> `Contacto\Proveedor\ProveedorController@index` (List all providers).
- `GET /proveedores/create` -> `Contacto\Proveedor\ProveedorController@create` (Show create form).
- `POST /proveedores` -> `Contacto\Proveedor\ProveedorController@store` (Store new provider).
- `GET /proveedores/{id}` -> `Contacto\Proveedor\ProveedorController@show` (Show single provider).
- `GET /proveedores/{id}/edit` -> `Contacto\Proveedor\ProveedorController@edit` (Show edit form).
- `PUT /proveedores/{id}` -> `Contacto\Proveedor\ProveedorController@update` (Update provider).
- `DELETE /proveedores/{id}` -> `Contacto\Proveedor\ProveedorController@destroy` (Soft delete provider).
- `GET /proveedores/productos/{id}/{isSearching}` -> `Contacto\Proveedor\ProveedorController@get_productos` (Load products supplied by the provider).
- `POST /proveedores/productos/add` -> `Contacto\Proveedor\ProveedorController@add_productos` (Associate a product to a provider).
- `DELETE /proveedores/productos/delete` -> `Contacto\Proveedor\ProveedorController@delete_productos` (Remove a product from a provider).

## Model
- **Table Name**: `proveedores`
- **Main Columns**:
  - `persona_id` (foreign key)
- **Relationships**:
  - `persona()`: BelongsTo `Persona` (from the `Contacto` domain).
  - `productos()`: BelongsToMany `Servicio` (**Note: This is a bug in the code. The method is called `productos()` but returns a `BelongsToMany(Servicio::class)`**).
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Hardcoded checks ensure only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can view, list, create, and edit providers.
- **Composition**: Operates identically to `Cliente` and `Colaborador`. The creation process attempts to save nested `User`, `Persona`, `Direccion`, and `Ubicacion` instances.
- **Product Supply Mapping**: Providers are linked to the products they supply. This association restricts which products can be purchased from which provider during the `Compra` process.

## Database
- **Migrations**: `2014_10_12_000013_create_proveedores_table.php`, `2014_10_12_000031_create_proveedor_producto_table.php`.
- **Foreign Keys cascade**: `persona_id` is configured with `onDelete('cascade')`.

## Things to be careful about
- **Critical Code Bug in Relationship**: Inside `app/Models/Contacto/Proveedor.php`, the `productos()` relationship method incorrectly returns `return $this->belongsToMany(Servicio::class);` instead of `Producto::class`. This is a severe bug that will cause exceptions or silent failures when attempting to fetch or attach products to a provider. The `use` statement at the top of the file also incorrectly imports `App\Models\Comercio\Servicio` (which doesn't exist, as `Servicio` is under `Actividad`).
- **Destructive Cascades**: Same as the other contact types, deleting a `Persona` physically deletes the `Proveedor`, ignoring Eloquent SoftDeletes.
- **Complex Orchestration**: The controller manually builds and saves `User`, `Persona`, `Direccion`, and `Ubicacion`.
