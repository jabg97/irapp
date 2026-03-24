# Colaboradores Module

## Purpose
The `Colaboradores` module defines internal or external workers (collaborators) who provide the professional services (`Servicios`) offered by the platform. A collaborator is essentially a specialized User/Persona assigned to execute service orders (`Ordenes`).

## Routes
All `colaboradores` routes are grouped under the `/colaboradores` prefix.
- `GET /colaboradores/deleted` -> `Contacto\Colaborador\ColaboradorSoftDeleteController@index` (List soft-deleted collaborators).
- `PUT /colaboradores/deleted/{id}` -> `Contacto\Colaborador\ColaboradorSoftDeleteController@update` (Restore soft-deleted collaborator).
- `DELETE /colaboradores/deleted/{id}` -> `Contacto\Colaborador\ColaboradorSoftDeleteController@destroy` (Force delete collaborator).
- `GET /colaboradores` -> `Contacto\Colaborador\ColaboradorController@index` (List all collaborators).
- `GET /colaboradores/create` -> `Contacto\Colaborador\ColaboradorController@create` (Show create form).
- `POST /colaboradores` -> `Contacto\Colaborador\ColaboradorController@store` (Store new collaborator).
- `GET /colaboradores/{id}` -> `Contacto\Colaborador\ColaboradorController@show` (Show single collaborator).
- `GET /colaboradores/{id}/edit` -> `Contacto\Colaborador\ColaboradorController@edit` (Show edit form).
- `PUT /colaboradores/{id}` -> `Contacto\Colaborador\ColaboradorController@update` (Update collaborator).
- `DELETE /colaboradores/{id}` -> `Contacto\Colaborador\ColaboradorController@destroy` (Soft delete collaborator).
- `GET /colaboradores/servicios/{id}/{isSearching}` -> `Contacto\Colaborador\ColaboradorController@get_servicios` (Load services assigned to collaborator, optionally searching).
- `POST /colaboradores/servicios/add` -> `Contacto\Colaborador\ColaboradorController@add_servicios` (Assign a service to a collaborator).
- `DELETE /colaboradores/servicios/delete` -> `Contacto\Colaborador\ColaboradorController@delete_servicios` (Remove a service from a collaborator).

## Model
- **Table Name**: `colaboradores`
- **Main Columns**:
  - `persona_id` (foreign key)
- **Relationships**:
  - `persona()`: BelongsTo `Persona` (from the `Contacto` domain).
  - `servicios()`: BelongsToMany `Servicio` (from the `Actividad` domain).
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization**: Hardcoded checks ensure only `ROLE_ROOT` and `ROLE_ADMINISTRADOR` can view, list, create, and edit collaborators from the backend.
- **Composition**: Like `Cliente`, a `Colaborador` wraps a `Persona` (which wraps a `User`). The creation form instantiates nested `User`, `Persona`, `Direccion`, and `Ubicacion` instances.
- **Service Assignment**: The relationship `colaboradores_servicios` defines *what* a collaborator is capable of doing. This dictates which collaborators are available to be assigned to specific `Ordenes` later in the system.

## Database
- **Migrations**: `2014_10_12_000012_create_colaboradores_table.php`, `2014_10_12_000026_create_colaborador_servicio_table.php`.
- **Foreign Keys cascade**: `persona_id` is configured with `onDelete('cascade')`.

## Things to be careful about
- **Destructive Cascades**: Same as clients, deleting a `Persona` physically deletes the `Colaborador`, ignoring Eloquent SoftDeletes.
- **Complex Orchestration**: The controller manually builds and saves `User`, `Persona`, `Direccion`, and `Ubicacion`. If one model fails to save during `store()`, it leaves orphaned data unless protected by DB transactions.
