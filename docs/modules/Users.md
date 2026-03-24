# Users Module

## Purpose
The `Users` module is responsible for the foundational authentication and authorization capabilities of the platform. It represents a user capable of logging in. A `User` serves as the root entity for all identity-based abstractions, linking to a `Persona` object which further specializes into `Cliente`, `Colaborador`, or `Proveedor`.

## Routes
All authentication-related routes are standard Laravel Auth routes, extended with API functionality and a specific client registration route.
- Standard Auth routes (`/login`, `/register`, `/password/reset`, etc.) handled by `Auth\*Controller` classes.
- `POST /clientes/register` -> `Auth\RegisterController@register_cliente` (Specific endpoint to register a new user that is also specifically a `Cliente`).
- `GET /api/login/{email_aes}/{password_aes}` -> `WebServiceController@login` (Custom API endpoint for mobile/external client authentication using AES-encrypted parameters).

## Model
- **Table Name**: `users`
- **Main Columns**:
  - `name` (string, unique)
  - `email` (string, unique)
  - `password` (string)
  - `imagen` (string, avatar URL or path)
  - `email_verified_at` (timestamp)
- **Relationships / Traits**:
  - Uses `Notifiable` (default Laravel trait).
  - Uses `SoftDeletes` (with conflict resolution for the `restore()` method against Entrust).
  - Uses `EntrustUserTrait` (from `Zizaco/Entrust` for role management: `roles()`).
  - Contains helper methods: `getPersona()`, `getCliente()`, `getColaborador()`.
- **SoftDeletes**: Yes.

## Business Logic
- **Authorization Package**: Heavily relies on `zizaco/entrust`. This package provides traits to check user roles (`$user->hasRole('ROLE_ADMINISTRADOR')`).
- **Custom Role Check**: The `User` model implements a custom `authorizeRoles($roles, $abort)` method that wraps Entrust's `hasAnyRole` and can automatically abort the HTTP request with a 401 response if the user doesn't meet the requirements. This is used extensively at the top of every Controller method instead of Middleware.
- **API Login Encryption**: The `WebServiceController@login` endpoint decrypts the email and password using a hardcoded `LzlDiqCCvXJUQ8j3+CWh1LgMfPMiLuVfIOElqaZ0Zg0=` AES key (via `Blocktrail\CryptoJSAES\CryptoJSAES`) before verifying the hash.
- **Identity Traversal**: To get a user's client profile, the application uses `$user->getCliente()`, which internally looks up `Persona::where('usuario_id', $this->id)` and then `Cliente::where('persona_id', $persona->id)`.

## Database
- **Migrations**: `2014_10_12_000001_create_users_table.php`, `2014_10_12_000002_entrust_setup_tables.php`.
- **SoftDeletes Collision**: Because both `SoftDeletes` and `EntrustUserTrait` define a `restore()` method, the `User` model resolves the conflict using `as private restoreA/restoreB` and creating a custom `restore()` method that calls both.

## Things to be careful about
- **Hardcoded AES Key**: The API login endpoint uses a hardcoded, plaintext AES key in `WebServiceController.php` which is a major security flaw.
- **Missing Entrust Maintenance**: `zizaco/entrust` is severely outdated and incompatible with modern Laravel versions. Migrating away from this will require refactoring the `User` model traits, the migrations, and the custom `authorizeRoles` method used across all controllers.
- **Identity Resolution Fallbacks**: The `getPersona()`, `getCliente()`, and `getColaborador()` methods return a new, empty, unsaved instance of the model (`return ($cliente) ? $cliente : new Cliente;`) if one does not exist in the database. Calling methods or attempting to save these empty instances without populating them may lead to unexpected SQL foreign key constraint failures later in the code.
