# Ventas Module

## Purpose
The `Ventas` module represents the core e-commerce sales flow. It captures a transaction where a client (`Cliente`) purchases products from the platform. It tracks the status of the order (Open, Cancelled, Pending, Shipped, Delivered), the destination address, and the specific products bought in the order via its details (`Detalle_venta`).

## Routes
All `ventas` routes are grouped under the `/ventas` prefix, with some client-specific views under `/cliente`.
- `GET /ventas/index/{estado?}` -> `Comercio\Venta\VentaController@index` (List all sales, filterable by state).
- `GET /ventas/create/{fecha?}` -> `Comercio\Venta\VentaController@create` (Show create form).
- `GET /ventas/deleted` -> `Comercio\Venta\CompraSoftDeleteController@index` (List soft-deleted sales. *Note: Typo in route definition points to CompraSoftDeleteController*).
- `PUT /ventas/deleted/{id}` -> `Comercio\Venta\CompraSoftDeleteController@update` (Restore soft-deleted sale).
- `DELETE /ventas/deleted/{id}` -> `Comercio\Venta\CompraSoftDeleteController@destroy` (Force delete sale).
- `POST /ventas` -> `Comercio\Venta\VentaController@store` (Store new sale).
- `GET /ventas/{id}` -> `Comercio\Venta\VentaController@show` (Show single sale).
- `GET /ventas/{id}/edit` -> `Comercio\Venta\VentaController@edit` (Show edit form).
- `PUT /ventas/{id}` -> `Comercio\Venta\VentaController@update` (Update sale).
- `DELETE /ventas/{id}` -> `Comercio\Venta\VentaController@destroy` (Soft delete sale).
- `GET /cliente/compras/pay` -> `Comercio\Venta\VentaController@pay` (Simulate/process payment for a client).
- `PUT /cliente/compras/{id}` -> `Comercio\Venta\VentaController@cancel` (Client cancels their own purchase).
- `PUT /ventas/cancel/{id}` -> `Comercio\Venta\VentaController@cancel_admin` (Admin cancels a sale).
- `PUT /ventas/send/{id}` -> `Comercio\Venta\VentaController@send` (Admin marks sale as Shipped 'Enviado').
- `PUT /ventas/deliver/{id}` -> `Comercio\Venta\VentaController@deliver` (Admin marks sale as Delivered 'Entregado').
- `GET /cliente/compras/{id}/{estado?}` -> `Comercio\Venta\VentaController@info` (Client views their specific purchase).

## Model
- **Table Name**: `ventas`
- **Main Columns**:
  - `fecha` (datetime)
  - `estado` (enum: 'Abierta', 'Cancelada', 'Pendiente', 'Entregado', 'Enviado')
  - `direccion_id` (foreign key)
  - `cliente_id` (foreign key)
- **Relationships**:
  - `direccion()`: BelongsTo `Direccion`.
  - `cliente()`: BelongsTo `Cliente`.
  - `detalles()`: HasMany `Detalle_venta`.
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Split logic. Admin routes require `ROLE_ROOT` or `ROLE_ADMINISTRADOR`. Client routes require `ROLE_CLIENTE`.
- **Dynamic Enums**: Uses the same regex pattern extraction via raw SQL (`SHOW COLUMNS FROM ventas WHERE Field = 'estado'`) as `Ordenes` and `Solicitudes`.
- **JSON Payload Generation**: `index` method builds `JSON_ventas` using raw SQL with hardcoded color variables based on the state (`$this->teal`, `$this->indigo`, etc.) mapping directly to the frontend library used.
- **State Management**: Specific endpoints are provided (`/send`, `/deliver`, `/cancel`) to handle the state transitions of a sale.

## Database
- **Migrations**: `2014_10_12_000034_create_ventas_table.php`, `2014_10_12_000035_create_detalles_ventas_table.php`.
- **Raw SQL**: Heavy use of raw SQL in the index method to format JSON payloads for calendars or dashboards.

## Things to be careful about
- **Typo in Route Definition**: The route `ventas/deleted` uses `CompraSoftDeleteController` instead of `VentaSoftDeleteController`. This is likely a copy-paste error and will break if an administrator attempts to view or restore deleted sales.
- **Data Encapsulation**: Much of the business logic regarding payments (`pay`), state changes, and JSON serialization is baked directly into the controller instead of being separated into Services or Resources.
- **Client vs Admin Vocabulary**: A client viewing a `Venta` calls it a "compra" (e.g., `/cliente/compras`), which overlaps in naming with the actual `Compras` module (supplier purchases). The `VentaController` manages these "client purchases" (sales from the system's perspective).
