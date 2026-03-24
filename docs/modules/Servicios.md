# Servicios Module

## Purpose
The `Servicios` module acts as the catalog of professional services offered by the platform. These services are the entities that are requested (`Solicitudes`) and performed (`Ordenes`). Each service is characterized by its unit value, description, measurement unit, and category. Services can also be associated with images and ratings (`calificaciones`).

## Routes
All `servicios` routes are grouped under the `/servicios` prefix.
- `GET /servicios/deleted/index` -> `Actividad\Servicio\ServicioSoftDeleteController@index` (List deleted services).
- `PUT /servicios/deleted/{id}` -> `Actividad\Servicio\ServicioSoftDeleteController@update` (Restore deleted service).
- `DELETE /servicios/deleted/{id}` -> `Actividad\Servicio\ServicioSoftDeleteController@destroy` (Force delete service).
- `GET /servicios` -> `Actividad\Servicio\ServicioController@index` (List all services).
- `GET /servicios/create` -> `Actividad\Servicio\ServicioController@create` (Show create form).
- `POST /servicios` -> `Actividad\Servicio\ServicioController@store` (Store new service).
- `GET /servicios/{id}` -> `Actividad\Servicio\ServicioController@show` (Show single service).
- `GET /servicios/{id}/edit` -> `Actividad\Servicio\ServicioController@edit` (Show edit form).
- `PUT /servicios/{id}` -> `Actividad\Servicio\ServicioController@update` (Update service).
- `DELETE /servicios/{id}` -> `Actividad\Servicio\ServicioController@destroy` (Soft delete service).
- `GET /servicios/detalles/{id}` -> `Actividad\Servicio\ServicioController@get_servicios_detalles` (Load service details/AJAX).
- `GET /servicios/load/imagenes/{id}` -> `Actividad\Servicio\ServicioController@load_imagenes` (Load all images of a service).
- `GET /servicios/load/row/imagenes/{id}` -> `Actividad\Servicio\ServicioController@load_row_imagenes` (Load a specific image row).
- `POST /servicios/upload/imagenes/{id}` -> `Actividad\Servicio\ServicioController@upload_imagenes` (Upload new image for a service).
- `DELETE /servicios/delete/imagenes/{id}` -> `Actividad\Servicio\ServicioController@delete_imagenes` (Delete an image of a service).

## Model
- **Table Name**: `servicios`
- **Main Columns**:
  - `nombre` (string)
  - `valor_unitario` (double)
  - `descripcion` (text)
  - `categoria_id` (foreign key)
  - `medida_id` (foreign key)
- **Relationships**:
  - `categoria()`: BelongsTo `Categoria` (from the `Clasificacion` domain).
  - `medida()`: BelongsTo `Medida` (from the `Dato_basico` domain).
  - `colaboradores()`: BelongsToMany `Colaborador`.
  - `imagenes()`: HasMany `Imagen_servicio`.
  - `calificaciones()`: HasMany `Calificacion_servicio`.

## Business Logic
- **Authorization**: Hardcoded `ROLE_ROOT` and `ROLE_ADMINISTRADOR` checks in `ServicioController`.
- **Image Processing**: Images are uploaded and processed using the `intervention/image` package (identified from the `use Intervention\Image\ImageManagerStatic as Image` import).
- **Association Mapping**: Services are mapped to `Medida` and `Categoria` instances. When creating a service, empty model instances of `Medida` and `Categoria` are passed to the view to reuse the same form logic.
- **Many-to-Many**: Services can have many collaborators capable of executing them (via the `colaboradores` relation).

## Database
- **Migrations**: `2014_10_12_000022_create_servicios_table.php`, `2014_10_12_000023_create_imagenes_servicios_table.php`, `2014_10_12_000024_create_calificaciones_servicios_table.php`, `2014_10_12_000025_create_cliente_servicio_table.php`, `2014_10_12_000026_create_colaborador_servicio_table.php`.
- **Foreign Keys cascade**: `categoria_id` and `medida_id` are configured with `onDelete('cascade')`.
- **Error in Migration**: In `2014_10_12_000022_create_servicios_table.php`, the `down()` method tries to drop `colaboradores_servicios` instead of `servicios`. This will cause `php artisan migrate:rollback` to fail or leave orphaned tables.

## Things to be careful about
- **Broken Rollback**: Because the `down` method of the `servicios` migration drops the wrong table, migrations cannot be rolled back cleanly.
- **Cascade Deletions**: Deleting a `Categoria` or `Medida` will automatically delete the related `Servicio` from the database (hard-delete at the DB level via constraint `onDelete('cascade')`), skipping the Eloquent SoftDelete events.
- **File System State**: Ensure any storage disk used for `Imagen_servicio` respects the deployment environment, otherwise image upload references will break.
