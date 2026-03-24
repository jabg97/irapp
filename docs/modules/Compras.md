# Compras Module

## Purpose
The `Compras` module handles inventory purchasing. These are transactions made by the system administrators to acquire products from suppliers (`Proveedores`). It logs when a purchase was made and records its individual line items (`detalles_compras`).

## Routes
All `compras` routes are prefixed with `/compras` or `/compras/deleted`.
- `GET /compras/deleted` -> `Comercio\Compra\CompraSoftDeleteController@index` (List soft-deleted purchases).
- `PUT /compras/deleted/{id}` -> `Comercio\Compra\CompraSoftDeleteController@update` (Restore soft-deleted purchase).
- `DELETE /compras/deleted/{id}` -> `Comercio\Compra\CompraSoftDeleteController@destroy` (Force delete purchase).
- `GET /compras` -> `Comercio\Compra\CompraController@index` (List all purchases).
- `GET /compras/create` -> `Comercio\Compra\CompraController@create` (Show create form).
- `POST /compras` -> `Comercio\Compra\CompraController@store` (Store new purchase).
- `GET /compras/{id}` -> `Comercio\Compra\CompraController@show` (Show single purchase).
- `GET /compras/{id}/edit` -> `Comercio\Compra\CompraController@edit` (Show edit form).
- `PUT /compras/{id}` -> `Comercio\Compra\CompraController@update` (Update purchase).
- `DELETE /compras/{id}` -> `Comercio\Compra\CompraController@destroy` (Soft delete purchase).
- `GET /compras/detalles/{id}` -> `Comercio\Compra\CompraController@get_detalles` (Load purchase details).
- `GET /compras/detalles/form/{id}/{editar}` -> `Comercio\Compra\CompraController@form_detalles` (Show form for purchase details).
- `GET /compras/detalles/productos/{id}/{editar}` -> `Comercio\Compra\CompraController@get_productos` (Load products for details).
- `GET /compras/detalles/proveedores/{id}/{editar}` -> `Comercio\Compra\CompraController@get_proveedores` (Load providers for details).
- `POST /compras/detalles/add` -> `Comercio\Compra\CompraController@add_detalles` (Add product detail to purchase).
- `PUT /compras/detalles/update/{id}` -> `Comercio\Compra\CompraController@update_detalles` (Update purchase detail).
- `PUT /compras/detalles/state/{id}` -> `Comercio\Compra\CompraController@state_detalles` (Change state of purchase detail).
- `DELETE /compras/detalles/delete/{id}` -> `Comercio\Compra\CompraController@delete_detalles` (Delete purchase detail).

## Model
- **Table Name**: `compras`
- **Main Columns**:
  - `id` (bigIncrements)
  - `fecha` (datetime)
- **Relationships**:
  - `detalles()`: HasMany `Detalle_compra`.
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Access to purchase lists and creation forms is strictly limited to `ROLE_ROOT` and `ROLE_ADMINISTRADOR` roles.
- **Header vs Lines**: Similar to `Ordenes`, `Compras` separates the transaction header (`Compra` with just a date) from the items being purchased (`Detalle_compra`).
- **Inventory Impact**: The creation of a `Detalle_compra` likely involves associating a `Producto` and a `Proveedor` and modifying the available stock of the product (needs to be checked in `CompraController@add_detalles` logic).
- **JSON Formatting**: `CompraController@index` generates a `JSON_compras` object containing formatted start times and hardcoded styling variables (`$this->teal`) mapped via raw SQL.

## Database
- **Migrations**: `2014_10_12_000032_create_compras_table.php`, `2014_10_12_000033_create_detalles_compras_table.php`.
- **Foreign Keys**: `Detalle_compra` has foreign keys to both `compras` and `productos`, and potentially `proveedores`.
- **Raw SQL**: The raw SQL used in the `index` method forces formatting (e.g., `REPLACE(compras.fecha,' ','T') AS start`) specific to the frontend library used.

## Things to be careful about
- **Implicit Details Model**: A `Compra` by itself has very little data. Deleting or modifying a `Compra` implies managing its `Detalle_compra` children properly.
- **No Suppliers at Header Level**: The `Compra` header has no supplier field. Suppliers are associated at the line item level (`Detalle_compra`), meaning one "Purchase" could theoretically involve multiple suppliers. This is unconventional and may cause aggregate reporting issues.
