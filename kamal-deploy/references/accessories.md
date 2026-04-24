# Kamal accessories: databases and Redis

Accessories are sidecar services Kamal manages alongside the app — Kamal pulls the image, runs the container on the configured servers, and exposes it on Kamal's internal overlay network at hostname `<service>-<accessory>` (e.g., `klive-postgres`). The app container reaches it by that hostname.

This file has the team's standard configs for the three most common accessories. Copy into `config/deploy.yml`, edit names/versions, add the secrets.

**Recommendation when the user asks for a database:** suggest **Postgres** first. Use MySQL only if the user explicitly needs it (legacy schema, ORM constraint, vendor requirement). Postgres has stronger types, better concurrent-write behavior, and is the default in most modern app frameworks (Rails 8, Phoenix, Django).

---

## Postgres (default recommendation for SQL)

```yaml
accessories:
  postgres:
    image: postgres:17
    role: web
    port: 5432
    env:
      clear:
        POSTGRES_DB: <app-name>_production
      secret:
        - POSTGRES_USER
        - POSTGRES_PASSWORD
    directories:
      - postgres-data:/var/lib/postgresql/data
```

App connects via `postgres://$POSTGRES_USER:$POSTGRES_PASSWORD@<service>-postgres:5432/<app-name>_production`.

Pin the major version (17, 16, 15). Don't use `:latest` for databases — you don't want a surprise major upgrade in the middle of a deploy.

---

## MySQL (only when Postgres isn't an option)

```yaml
accessories:
  mysql:
    image: mysql:8.0.45
    role: web
    port: 3306
    env:
      clear:
        MYSQL_ROOT_HOST: '%'
      secret:
        - MYSQL_ROOT_PASSWORD
        - MYSQL_DATABASE
    files:
      - config/mysql/my.cnf:/etc/mysql/my.cnf
      - config/mysql/init.sql:/docker-entrypoint-initdb.d/setup.sql
    directories:
      - mysql-data:/var/lib/mysql
```

The `files:` block mounts your config + init SQL into the container at boot. Create them in the repo:
- `config/mysql/my.cnf` — server config (charset, sql_mode, max_connections, etc.)
- `config/mysql/init.sql` — runs once on first container boot (CREATE USER, GRANT, etc.)

Both are committed to the repo (not secrets).

App connects via `mysql://root:$MYSQL_ROOT_PASSWORD@<service>-mysql:3306/$MYSQL_DATABASE`.

---

## Redis (cache, sessions, queues, pub/sub)

```yaml
accessories:
  redis:
    image: bitnami/redis:latest
    role: web
    port: 6379
    env:
      clear:
        REDIS_AOF_ENABLED: "yes"
      secret:
        - REDIS_PASSWORD
    options:
      user: root
    directories:
      - redis-data:/bitnami/redis/data
```

`REDIS_AOF_ENABLED: "yes"` turns on append-only-file persistence — survives restarts. For pure cache (no persistence needed), drop this env var and the `directories:` block; restart wipes everything, container is faster.

App connects via `redis://:$REDIS_PASSWORD@<service>-redis:6379/0`.

---

## Secrets handling

For every accessory, add the secret env vars to `.kamal/secrets` by name (not literal value):

```bash
# .kamal/secrets
KAMAL_REGISTRY_USERNAME=KeqinHQ
KAMAL_REGISTRY_PASSWORD=$KAMAL_REGISTRY_PASSWORD
POSTGRES_USER=$POSTGRES_USER
POSTGRES_PASSWORD=$POSTGRES_PASSWORD
REDIS_PASSWORD=$REDIS_PASSWORD
```

Set the actual values in the user's shell or a gitignored `.env` file:

```bash
export POSTGRES_USER='myapp'
export POSTGRES_PASSWORD="$(openssl rand -base64 24)"
export REDIS_PASSWORD="$(openssl rand -base64 24)"
```

Generate strong passwords with `openssl rand -base64 24` (32-char output). Save them in the team's password manager — losing the DB password means losing the data (or a long restore from backup).

---

## Boot and manage accessories

After adding to `deploy.yml`:

```bash
kamal accessory boot <name>         # e.g., kamal accessory boot postgres
kamal accessory details <name>      # running container info, restart count
kamal accessory logs <name>         # tail logs (-f to follow)
kamal accessory exec <name> <cmd>   # one-off command, e.g.:
kamal accessory exec postgres -i 'psql -U $POSTGRES_USER -d <app>_production'
kamal accessory restart <name>
kamal accessory remove <name>       # tears down container + volume — careful
```

The accessory boots on every host with the matching `role` (`web` in these configs). For multi-host setups where the database should only live on one box, give the DB its own role — e.g., `role: db` — and only declare hosts under `servers.db`.

---

## Backups and migrations — NOT covered here

These configs cover bringing the service up. They do NOT cover:
- **Backups** — schedule `pg_dump` / `mysqldump` / `redis-cli BGSAVE` via cron, push the dumps off the server (S3, COS, OSS).
- **Schema migrations** — your app framework handles this (e.g., `kamal app exec 'bin/rails db:migrate'`). Run after every deploy that ships a migration.

Plan both before going live in production.
