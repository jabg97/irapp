# Architecture Document

## Project Purpose
This application is a comprehensive e-commerce and service management platform. It manages both physical products and professional services. It tracks sales (`ventas`), purchases (`compras`), service requests (`solicitudes`), and service orders (`ordenes`). It serves multiple types of users defined by roles (ROOT, ADMINISTRADOR, COLABORADOR, CLIENTE) and manages interactions between clients, service collaborators, and providers. The project also acts as a backend for an API (used by a mobile app or frontend) to access user orders and login.

## Actual Structure
The project deviates from the standard flat Laravel structure by implementing a Domain-Driven Design (DDD) approach for Models and Controllers.

- **`app/Models/`**: Grouped into domains:
  - `Actividad`: Models related to service execution (`Orden`, `Servicio`, `Solicitud`).
  - `Clasificacion`: Categorization models (`Categoria`).
  - `Comercio`: Models related to products, sales, and purchases (`Producto`, `Venta`, `Compra`, `Marca`).
  - `Contacto`: Models for actors in the system (`Cliente`, `Colaborador`, `Proveedor`, `Persona`).
  - `Dato_basico`: Core geographical and location data (`Pais`, `Ciudad`, `Direccion`, `Ubicacion`).
  - `Root`: Authentication and authorization models (`User`, `Role`, `Permission`).
- **`app/Http/Controllers/`**: Mirrored structure matching the domains above (e.g., `Actividad\Orden\OrdenController`).
- **`app/Classes/`**: Contains custom business logic classes (e.g., `Store/Filter`).
- **`app/Jobs/`**: No background jobs found. The folder does not exist.
- **`app/Console/Kernel.php`**: No scheduled commands configured.

## Routes Map
Routes are primarily defined in `routes/web.php` and `routes/api.php`.

**Web Routes (`web.php`):**
- **Marcas:** Resource routes (`/marcas`) handled by `Comercio\Marca\MarcaController`. Soft deletes handled by a custom `Comercio\Marca\MarcaSoftDeleteController` (deduced from pattern).
- **Productos:** Resource routes (`/productos`) handled by `Comercio\Producto\ProductoController`. Includes custom routes for loading/uploading images (`/productos/load/imagenes/{id}`) and references.
- **Compras:** Resource routes (`/compras`) handled by `Comercio\Compra\CompraController`. Includes custom routes for details (`/compras/detalles/{id}`).
- **Ventas:** Resource routes (`/ventas`) handled by `Comercio\Venta\VentaController`. Includes specific routes for client purchases (`/cliente/compras/*`).
- **Clientes:** Resource routes (`/clientes`) handled by `Contacto\Cliente\ClienteController`.
- **Colaboradores:** Resource routes (`/colaboradores`) handled by `Contacto\Colaborador\ColaboradorController`. Includes assignment of services (`/colaboradores/servicios/*`).
- **Proveedores:** Resource routes (`/proveedores`) handled by `Contacto\Proveedor\ProveedorController`. Includes assignment of products.
- **Servicios:** Resource routes (`/servicios`) handled by `Actividad\Servicio\ServicioController`. Includes image management.
- **Ordenes:** Resource routes (`/ordenes`) handled by `Actividad\Orden\OrdenController`. Includes management of order details and state updates.
- **Solicitudes:** Resource routes (`/solicitudes`) handled by `Actividad\Solicitud\SolicitudController`. Includes approval and cancellation logic.
- **StoreFront:** Handled by `StoreController`, serving the public-facing catalog/images (`/store/*`).
- **SoftDeletes:** Explicit routes (e.g., `productos/deleted`) managed by dedicated `*SoftDeleteController` controllers for almost all resources.

**API Routes (`api.php`):**
- `/login/{email_aes}/{password_aes}`: Authentication endpoint utilizing AES encryption via URL parameters. Handled by `WebServiceController@login`.
- `/ordenes/{user}`: Endpoint to fetch orders for a given user. Handled by `WebServiceController@ordenes`.

## Models & Database
The application maps the following Eloquent models to database tables based on migrations:

**Actividad**
- `Orden` -> `ordenes`
- `Detalle_orden` -> `detalles_ordenes`
- `Servicio` -> `servicios`
- `Imagen_servicio` -> `imagenes_servicios`
- `Calificacion_servicio` -> `calificaciones_servicios`
- `Solicitud` -> `solicitudes`
- `Detalle_solicitud` -> `detalles_solicitudes`

**Clasificacion**
- `Categoria` -> `categorias`

**Comercio**
- `Producto` -> `productos`
- `Imagen_producto` -> `imagenes_productos`
- `Calificacion_producto` -> `calificaciones_productos`
- `Venta` -> `ventas`
- `Detalle_venta` -> `detalles_ventas`
- `Compra` -> `compras`
- `Detalle_compra` -> `detalles_compras`
- `Marca` -> `marcas`
- `Medida`, `Tipo_medida` -> `medidas`, `tipos_medidas`
- `Tipo_referencia` -> `tipos_referencias`

**Contacto**
- `Persona` -> `personas` (Base entity for personal data)
- `Cliente` -> `clientes` (Belongs to Persona)
- `Colaborador` -> `colaboradores` (Belongs to Persona)
- `Proveedor` -> `proveedores`

**Dato_basico**
- `Pais`, `Departamento`, `Ciudad` -> `paises`, `departamentos`, `ciudades`
- `Direccion`, `Ubicacion` -> `direcciones`, `ubicaciones`

**Root**
- `User`, `Role`, `Permission` -> `users`, `roles`, `permissions` (Entrust setup)

*Relationships:* Pivot tables exist for `cliente_producto`, `cliente_servicio`, `colaborador_servicio`, `proveedor_producto`, `permission_role`, and `role_user`.

## External Dependencies
- **`zizaco/entrust`**: Manages Roles and Permissions. Critical for all access control (`authorizeRoles()` in User model).
- **`blocktrail/cryptojs-aes-php`**: Used in `WebServiceController` for decrypting credentials sent via API.
- **`intervention/image`**: Used for processing uploaded images for products and services.
- **`milon/barcode`**: Likely used for generating product barcodes or order tracking codes.
- **`realrashid/sweet-alert`**: Heavily used in controllers to flash UI alert messages instead of standard session flashes.

## Entry Points
- **`HomeController@index`**: The main dashboard for authenticated users. It returns different views and datasets based on the user's role (Admin vs. Client/Collaborator). Contains complex raw DB queries for calendar/dashboard metrics.
- **`StoreController@index`**: The public entry point for the shop/frontend.
- **`WebServiceController`**: The entry point for external API calls, notably handling the custom login mechanism.

## Things that look outdated or unusual
- **Laravel 5.7 & PHP 7.1.3**: The project relies on severely outdated and End-Of-Life versions of PHP and Laravel.
- **Zizaco/Entrust**: This package has been abandoned for a very long time. It is a huge technical debt and security risk. Modern Laravel typically uses `spatie/laravel-permission`.
- **API Credentials in URL**: Passing AES-encrypted passwords directly in the GET URL (`/login/{email_aes}/{password_aes}`) is highly unusual, insecure (logs, history), and prone to encoding issues. Standard practice dictates POST requests with JSON payloads and secure token exchange (e.g., Passport/Sanctum).
- **Domain Controllers for Soft Deletes**: Each resource has an independent controller specifically for soft-deleted records (e.g., `CompraSoftDeleteController`), which adds unnecessary duplication compared to standard controller methods or scoped routes.
- **Fat Controllers**: `HomeController@index` manually checks roles using `if/else` and builds complex raw SQL/DB queries instead of leveraging Eloquent scopes or dedicated service classes.
- **SweetAlert in Controllers**: The application tightly couples backend controllers to frontend presentation by using the `SweetAlert` facade directly within controller logic.
