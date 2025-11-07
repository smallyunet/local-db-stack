# Local Postgres + Redis stack (Docker Compose)

A tiny Docker Compose setup for local development that brings up:

- PostgreSQL 16 (data persisted to `./data`)
- Redis 7 (AOF enabled, data persisted to `./redis-data`)
- pgAdmin 4 (web UI for Postgres)
- RedisInsight (web UI for Redis)

Tested on macOS with Docker Desktop.

---

## Quick start

```sh
# Start everything
docker compose up -d

# Check container status
docker compose ps

# Stop everything
docker compose down
```

Services will be available at:

- Postgres: `localhost:5432`
- Redis: `localhost:6379`
- pgAdmin: http://localhost:5050
- RedisInsight: http://localhost:5540

---

## Default credentials

Postgres (service `postgres`):
- Username: `devuser`
- Password: `devpass`
- Database: `devdb`

pgAdmin (service `pgadmin`):
- Login: disabled (desktop mode)
- A pre-registered connection named “Local Postgres” points to the Postgres service

Redis (service `redis`):
- No password by default

RedisInsight (service `redisinsight`):
- Add a database with:
  - Host: `redis`
  - Port: `6379`
  - Name: any display name (e.g., `local-redis`)

> Note: inside Docker, services talk to each other via service names (`postgres`, `redis`). From your host, use `localhost` and the mapped ports.

---

## Web UIs

- pgAdmin 4: http://localhost:5050
  - Browse: Servers → Local Postgres → Databases → `devdb`
  - View data: right‑click a table → View/Edit Data → All Rows
  - Create objects without SQL: right‑click schema/table → Create → …

- RedisInsight: http://localhost:5540
  - Add DB using Host `redis`, Port `6379`
  - Explore keys, TTL, memory usage; search/edit values; run commands

---

## Files & layout

```
.
├── docker-compose.yml        # Services and volumes
├── start.sh                  # Convenience: docker compose up -d
├── data/                     # Postgres data (gitignored)
├── redis-data/               # Redis data (gitignored)
├── pgadmin-data/             # pgAdmin state (gitignored)
├── redisinsight-data/        # RedisInsight state (gitignored)
└── pgadmin-servers.json      # Pre-registered pgAdmin server definition
```

---

## Common tasks

```sh
# Start only Postgres and Redis (omit the UIs)
docker compose up -d postgres redis

# Start only the UIs (when DBs already running)
docker compose up -d pgadmin redisinsight

# Tail logs of a service
docker compose logs -f postgres   # or redis/pgadmin/redisinsight

# Open a psql shell
docker exec -it dev-postgres psql -U devuser -d devdb

# Use redis-cli in the container
docker exec -it dev-redis redis-cli
```

---

## Configuration notes

- Postgres variables (in `docker-compose.yml`): `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`.
- Redis runs with AOF: `redis-server --appendonly yes`.
- pgAdmin runs in desktop mode (no login) via `PGADMIN_CONFIG_SERVER_MODE: "False"`.
  - To re‑enable login, remove that line and set:
    ```yaml
    PGADMIN_DEFAULT_EMAIL: your@email.com
    PGADMIN_DEFAULT_PASSWORD: yourpassword
    ```
- A pgAdmin server definition is auto-mounted from `pgadmin-servers.json` on first start.

### Security

- This setup is for local development. Credentials are weak on purpose.
- pgAdmin has no login in desktop mode. If you need protection, either:
  - Re-enable login (see above), or
  - Bind the UI to localhost only (already the case), and guard your host.

---

## Resetting state

If you want to reset any UI/state without touching the databases:

```sh
# Reset pgAdmin UI state (connections, history, etc.)
docker compose stop pgadmin
docker compose rm -f pgadmin
rm -rf ./pgadmin-data

# Reset RedisInsight UI state
docker compose stop redisinsight
docker compose rm -f redisinsight
rm -rf ./redisinsight-data

# Recreate services
docker compose up -d pgadmin redisinsight
```

To wipe database data (destructive):

```sh
docker compose down
rm -rf ./data ./redis-data
```

---

## Troubleshooting

- pgAdmin shows login page even with desktop mode
  - The mode is applied on first start; remove `./pgadmin-data` and recreate the container.
- Cannot connect from pgAdmin to Postgres
  - Use Host `postgres` inside pgAdmin (service name), not `localhost`.
- RedisInsight connection fails
  - Use Host `redis`, Port `6379`. If you add a password later, also set `--requirepass` in Redis and update the connection.

---

## License

This repository is intended for local development and learning. Use at your own discretion.
# local-db-stack
