# test-manager-library-3239188-infra

Infraestructura de *Manager Library*: contenedores, red, volúmenes y el motor de base de datos **vacío**.
Prueba técnica SDD, ficha ADSO 3239188. El spec y las decisiones están en el repo `docs` (`laravel-react/`).

## Estado

| Pieza | Estado |
|---|---|
| `db` (PostgreSQL 16, vacío, volumen, red, healthcheck) | implementado, **no ejecutado** (no hay Docker en la máquina de desarrollo) |
| `api` (Laravel, ejecuta `migrate --force` y `library:ensure-admin` al arrancar) | implementado, **no ejecutado** |
| `app` (React, perfil `web`) | declarado bajo `profiles: ["web"]`; se activa en T8 cuando `app/` tenga `Dockerfile` |

`docker compose up -d` levanta `db` y `api` con un solo comando (RNF-5).

## Cómo levantar

```bash
cp .env.example .env     # en PowerShell: Copy-Item .env.example .env
# editar .env y rellenar los valores vacíos (ver tabla)
docker compose up -d
docker compose ps        # db debe quedar "healthy"
```

Si falta una variable obligatoria, el compose no arranca y dice cuál (`defina POSTGRES_PASSWORD en .env`).

## Variables de entorno (`.env.example`)

| Variable | Obligatoria | Para qué |
|---|---|---|
| `POSTGRES_DB`, `POSTGRES_USER` | sí | Base y usuario del motor |
| `POSTGRES_PASSWORD` | sí | Clave del motor. Sin valor por defecto |
| `APP_KEY` | sí (con `api`) | Clave de Laravel: `php artisan key:generate --show` |
| `APP_TIMEZONE` | sí (con `api`) | `UTC` |
| `JWT_SECRET` | sí (con `api`) | Firma de los JWT; mínimo 32 caracteres aleatorios |
| `ADMIN_USERNAME`, `ADMIN_PASSWORD` | sí (con `api`) | Admin inicial, creado por `php artisan library:ensure-admin` |
| `CORS_ORIGIN` | sí (con `api`) | Único origen permitido: el del frontend |
| `COVERS_DIR` | no | Carpeta de portadas |
| `API_PORT` | no | Puerto publicado de la API (8000) |

El `.env` real está en `.gitignore`. Solo se versiona `.env.example`, sin secretos (Art. IX).

## Decisiones relevantes

- **La base arranca sin tablas (RNF-5, Art. V, ADR-001).** Este repo no contiene `CREATE TABLE` ni scripts en `docker-entrypoint-initdb.d`.
  Todo el DDL, incluida la creación del esquema `library` y las cinco categorías, vive en las migraciones de `api`.
- **El servidor corre en UTC** (`TZ` y `PGTZ`), como pide `data-model.md`.
- **Compruebe que no hay DDL aquí:** `grep -ri "create table" .` no debe devolver nada (el README no cuenta).
- **Laravel no usa `DATABASE_URL`:** el compose del servicio `api` construirá `DB_HOST`, `DB_DATABASE`, etc. a partir de las variables `POSTGRES_*`.

## Reiniciar desde cero

```bash
docker compose down -v   # borra el volumen library_pgdata: se pierden todos los datos
```
