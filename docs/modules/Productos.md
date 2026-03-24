# Productos Module

## Purpose
The `Productos` module manages the catalog of physical items that the e-commerce platform sells. These products are purchased from suppliers (`Compras`) and sold to clients (`Ventas`). It handles product stock levels, alerts for low inventory, categories, measurements, branding (`Marcas`), barcoding (`Tipo_referencia`), and image galleries.

## Routes
All `productos` routes are grouped under the `/productos` prefix.
- `GET /productos/deleted` -> `Comercio\Producto\ProductoSoftDeleteController@index` (List soft-deleted products).
- `PUT /productos/deleted/{id}` -> `Comercio\Producto\ProductoSoftDeleteController@update` (Restore soft-deleted product).
- `DELETE /productos/deleted/{id}` -> `Comercio\Producto\ProductoSoftDeleteController@destroy` (Force delete product).
- `GET /productos` -> `Comercio\Producto\ProductoController@index` (List all products).
- `GET /productos/create` -> `Comercio\Producto\ProductoController@create` (Show create form).
- `POST /productos` -> `Comercio\Producto\ProductoController@store` (Store new product).
- `GET /productos/{id}` -> `Comercio\Producto\ProductoController@show` (Show single product).
- `GET /productos/{id}/edit` -> `Comercio\Producto\ProductoController@edit` (Show edit form).
- `PUT /productos/{id}` -> `Comercio\Producto\ProductoController@update` (Update product).
- `DELETE /productos/{id}` -> `Comercio\Producto\ProductoController@destroy` (Soft delete product).
- `GET /productos/test/referencias` -> `Comercio\Producto\ProductoController@test_referencias` (Test barcode/reference generation).
- `GET /productos/load/referencias/{id}` -> `Comercio\Producto\ProductoController@load_referencias` (Load specific reference graphic).
- `GET /productos/load/imagenes/{id}` -> `Comercio\Producto\ProductoController@load_imagenes` (Load all images of a product).
- `GET /productos/load/row/imagenes/{id}` -> `Comercio\Producto\ProductoController@load_row_imagenes` (Load a specific image row).
- `POST /productos/upload/imagenes/{id}` -> `Comercio\Producto\ProductoController@upload_imagenes` (Upload new image for a product).
- `DELETE /productos/delete/imagenes/{id}` -> `Comercio\Producto\ProductoController@delete_imagenes` (Delete an image of a product).

## Model
- **Table Name**: `productos`
- **Main Columns**:
  - `nombre` (string)
  - `referencia` (string) - Likely used as SKU or Barcode.
  - `descripcion` (text)
  - `valor_unitario` (double) - Current price.
  - `stock` (double) - Current inventory quantity.
  - `alerta` (boolean) - Toggles low stock alerts.
  - `limite` (double) - Minimum stock threshold.
  - `medida_id`, `categoria_id`, `marca_id`, `tipo_referencia_id` (foreign keys).
- **Relationships**:
  - `marca()`: BelongsTo `Marca`.
  - `categoria()`: BelongsTo `Categoria`.
  - `medida()`: BelongsTo `Medida`.
  - `tipo_referencia()`: BelongsTo `Tipo_referencia`.
  - `imagenes()`: HasMany `Imagen_producto`.
  - `calificaciones()`: HasMany `Calificacion_producto`.
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Hardcoded checks ensuring only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can manage the catalog.
- **Stock Tracking**: The `stock` is kept as a denormalized value on the `productos` table. (Note: changes to this should ideally be handled via `Compra` and `Venta` interactions, though direct edits may be possible via standard update routes).
- **Barcode Generation**: Uses `milon/barcode` package (`DNS1D`, `DNS2D` facades in the controller) to dynamically generate and test reference codes based on `referencia` and `tipo_referencia_id`.
- **Image Processing**: Just like services, images are uploaded, processed, and resized using `intervention/image`.

## Database
- **Migrations**: `2014_10_12_000018_create_productos_table.php`, `2014_10_12_000019_create_imagenes_productos_table.php`, `2014_10_12_000020_create_calificaciones_productos_table.php`, `2014_10_12_000021_create_cliente_producto_table.php`, `2014_10_12_000031_create_proveedor_producto_table.php`.
- **Foreign Keys**: `medida_id`, `marca_id`, `categoria_id`, `tipo_referencia_id` have `onDelete('cascade')`.

## Things to be careful about
- **Destructive Cascades**: Since the relationships (`marca`, `categoria`, etc.) have `onDelete('cascade')` set at the database level, deleting a brand or a category will physically delete all related products instantly, ignoring the SoftDeletes logic on the `Producto` model. This is extremely dangerous for an e-commerce platform.
- **Concurrency in Stock**: The denormalized `stock` column may face race conditions if multiple `Ventas` or `Compras` occur simultaneously without atomic increments (`increment()`/`decrement()` DB operations).
