# Setup and Dependencies

## Laravel Version
This project runs on **Laravel 5.7.***.
> **Warning:** Laravel 5.7 reached End of Life (EOL) on August 2019. It no longer receives security updates or bug fixes. Upgrading to a supported version of Laravel is highly recommended.

## PHP Version Required
This project requires **PHP ^7.1.3** according to the `composer.json` constraints.
> **Warning:** PHP 7.1 reached End of Life on December 1, 2019. Modern environments will have trouble running this code natively without Docker or virtualization.

## Key Packages
- **`zizaco/entrust: ^1.9`**: Role-based Access Control (RBAC). Provides roles, permissions, and traits for the `User` model. This package is severely outdated and abandoned. The `users` table migration and the `User` model heavily depend on it.
- **`blocktrail/cryptojs-aes-php: ^0.1.0`**: Provides AES-256 decryption capabilities for the mobile/API endpoints. specifically used in `WebServiceController` to decrypt AES credentials passed through URL parameters.
- **`milon/barcode: ^5.3`**: Barcode generator. Used in `ProductoController` to generate visual 1D and 2D barcodes based on product references (`DNS1D` and `DNS2D` facades).
- **`intervention/image: ^2.4`**: Used for processing, resizing, and saving images associated with both `Producto` and `Servicio` models.
- **`realrashid/sweet-alert: ^1.1`**: Used directly in the controllers (`Use SweetAlert;`) instead of standard session flashes to trigger JavaScript UI alerts upon redirects.
- **`fideloper/proxy: ^4.0`**: A standard package in older Laravel versions to trust proxies and load balancers.

## Environment Variables
The `.env.example` file outlines the required configuration:

**Core Application Setup**
- `APP_NAME`: Application name.
- `APP_ENV`: Environment (local, production, etc).
- `APP_KEY`: Unique application key (Base64 string).
- `APP_DEBUG`: Enables detailed error pages (true/false).
- `APP_URL`: Base URL of the application.

**Logging & Connections**
- `LOG_CHANNEL`: Where to log errors (stack, single, daily).
- `BROADCAST_DRIVER`: Driver for real-time events (`log` by default).
- `CACHE_DRIVER`: Where to store cache (`file`).
- `QUEUE_CONNECTION`: Queue driver (`sync` meaning no background queues are actually running).
- `SESSION_DRIVER`: How to store sessions (`file`).
- `SESSION_LIFETIME`: Session timeout in minutes.

**Database Configuration**
- `DB_CONNECTION`: Type of database (mysql).
- `DB_HOST`: Database IP or hostname.
- `DB_PORT`: Database port (3306).
- `DB_DATABASE`: Name of the database schema.
- `DB_USERNAME`: Database user.
- `DB_PASSWORD`: Database password.

**Redis (Optional/Unused)**
- `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`: Connection settings for Redis.

**Mail (SMTP)**
- `MAIL_DRIVER`, `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_ENCRYPTION`: Configuration for sending emails via SMTP.

**Pusher (Websockets - Optional/Unused)**
- `PUSHER_APP_ID`, `PUSHER_APP_KEY`, `PUSHER_APP_SECRET`, `PUSHER_APP_CLUSTER`: Configuration for real-time broadcasting using Pusher.
- `MIX_PUSHER_APP_KEY`, `MIX_PUSHER_APP_CLUSTER`: Frontend exposure for Pusher configuration.

## Scheduled Jobs
Checking `app/Console/Kernel.php`, the `schedule` method is empty.
There are **no scheduled jobs** or cron tasks running in this application.

## Queue Jobs
Checking `app/Jobs/`, the directory does not exist.
There are **no queued jobs** utilized. Background tasks are seemingly executed synchronously during the HTTP request lifecycle.

## Notable Config
- **`config/app.php`**: Contains custom facades/aliases such as `'Entrust' => Zizaco\Entrust\EntrustFacade::class`, `'DNS1D' => Milon\Barcode\Facades\DNS1DFacade::class`, `'DNS2D'`, `'Image'`, and `'SweetAlert'`. It specifies `'cipher' => 'AES-256-CBC'`.
- **`config/entrust.php`**: Defines the role, user, permission models and their respective tables (`roles`, `users`, `permissions`, `role_user`, `permission_role`). The models mapped here point to custom namespaces (`App\Models\Root\Role`, `App\Models\Root\User`, etc.).
