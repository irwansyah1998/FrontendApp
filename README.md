<p align="center">
	<img src="src/assets/logo.png" alt="FrontendApp Vue logo" width="96">
</p>

<h1 align="center">FrontendApp</h1>

<p align="center">
	A modern Vue.js interface for authenticated product management through BackendApp.
</p>

<p align="center">
	<a href="https://github.com/irwansyah1998/BackendApp">BackendApp API</a>
</p>

## Overview

FrontendApp provides a focused workspace for signing in and managing products through a Laravel BackendApp API. The interface includes a responsive dashboard, token-based authentication, and complete product CRUD operations.

## Features

- Email and password login through BackendApp.
- Bearer token authentication stored in `localStorage`.
- Session restoration after a page refresh.
- Logout and token removal.
- Product list loaded from the API.
- Create, update, and delete product operations.
- Delete confirmation to prevent accidental changes.
- Loading, success, and API validation error states.
- Responsive UI for desktop, tablet, and mobile screens.

## Technology Stack

| Technology | Version |
| --- | --- |
| Vue | `3.2.13` |
| Vue CLI | `5` |
| Axios | `1.7.7` |
| Node.js | `18+` recommended |
| npm | `9+` recommended |

## Backend Requirements

Make sure BackendApp is running and exposes these routes:

```text
POST   /api/login
GET    /api/products
POST   /api/products
PUT    /api/products/{id}
PATCH  /api/products/{id}
DELETE /api/products/{id}
```

The login endpoint is expected to accept:

```json
{
	"email": "user@example.com",
	"password": "password"
}
```

The frontend supports these token response formats:

```json
{ "token": "..." }
```

or:

```json
{ "access_token": "..." }
```

Product records use these fields:

```json
{
	"id": 1,
	"name": "Example product",
	"price": 1000000,
	"description": "Product description"
}
```

## Installation

Clone the repository and open the project directory:

```bash
git clone <repository-url>
cd FrontendApp
```

Install dependencies:

```bash
npm install
```

Create the local environment file.

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS/Linux:

```bash
cp .env.example .env
```

Configure `.env` with your BackendApp address:

```env
VUE_APP_BACKEND=http://127.0.0.1:8000
VUE_APP_LOGIN_ENDPOINT=/api/login
VUE_APP_PRODUCTS_ENDPOINT=/api/products
```

The token is returned by the login endpoint and stored automatically by the application. Never commit `.env` or place a real secret in any `VUE_APP_*` variable: Vue CLI embeds these values into the browser bundle.

## Development

Start the development server:

```bash
npm run serve
```

Open the URL printed by Vue CLI, usually:

```text
http://localhost:8080
```

If BackendApp uses the same port, change the frontend or backend port to avoid a conflict.

## Production Build

Create an optimized production build:

```bash
npm run build
```

The compiled application is generated in `dist/` and can be deployed to Nginx, Apache, or another static web server.

## Code Validation

Run ESLint:

```bash
npm run lint
```

## Troubleshooting

### Login fails or the API cannot be reached

- Make sure BackendApp is running.
- Check the `VUE_APP_BACKEND` value.
- Confirm that `/api/login` is available.
- Confirm that backend CORS allows the frontend origin.
- Restart `npm run serve` after changing `.env`.

### Database price overflow

If BackendApp returns `SQLSTATE[22003]` for the `price` column, increase the database column size through a BackendApp migration, for example:

```php
$table->decimal('price', 12, 2)->change();
```

The frontend cannot correct a database schema limit.