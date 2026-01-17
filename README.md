# certmagic-sqlstorage

[![GoDoc](https://godoc.org/github.com/yroc92/certmagic-sqlstorage?status.svg)](https://godoc.org/github.com/yroc92/certmagic-sqlstorage)

SQL storage for CertMagic/Caddy TLS data.

Currently supports PostgreSQL using pgx driver with multi-host support for high availability.

Now with support for Caddyfile and environment configuration.

# Features
- PostgreSQL support using pgx driver
- Multi-host configuration for high availability and failover
- Connection string or individual parameter configuration
- Environment variable support
- Caddyfile configuration support

# Example
- Valid values for sslmode are: disable, require, verify-ca, verify-full
- Multi-host format: `host1,host2,host3` or `host1:port1,host2:port2,host3:port3`

## With vanilla JSON config file and single connection string:
```json
{
  "storage": {
    "module": "postgres",
    "connection_string": "postgres://user:password@localhost:5432/postgres?sslmode=disable",
    "disable_ddl": false
  },
  "app": {
    ...
  }
}
```

## With vanilla JSON config file and separate fields (single host):
```json
{
  "storage": {
    "module": "postgres",
    "dbname": "certmagictest",
    "host": "localhost",
    "password": "postgres",
    "port": "5432",
    "sslmode": "disable",
    "user": "postgres",
    "disable_ddl": false
  },
  "app": {
    ...
  }
}
```

## With vanilla JSON config file and multiple hosts (high availability):
```json
{
  "storage": {
    "module": "postgres",
    "dbname": "cdn_manager",
    "hosts": ["ocv5p4.easypanel.host", "backup1.easypanel.host", "backup2.easypanel.host"],
    "password": "4106969cd260dbbb75f9",
    "port": "5445",
    "sslmode": "disable",
    "user": "postgres",
    "disable_ddl": false
  },
  "app": {
    ...
  }
}
```

With Caddyfile (single host):
```Caddyfile
# Global Config

{
  storage postgres {
    connection_string postgres://user:password@localhost:5432/postgres?sslmode=disable
    disable_ddl false
  }
}
```
or
```Caddyfile
{
  storage postgres {
    dbname certmagictest
    host localhost
    password postgres
    port 5432
    sslmode disable
    user postgres
    disable_ddl false
  }
}
```

With Caddyfile (multiple hosts for high availability):
```Caddyfile
{
  storage postgres {
    dbname cdn_manager
    hosts ocv5p4.easypanel.host,backup1.easypanel.host,backup2.easypanel.host
    password 4106969cd260dbbb75f9
    port 5445
    sslmode disable
    user postgres
    disable_ddl false
  }
}
```

From Environment (single host):
```text
POSTGRES_CONN_STRING
POSTGRES_DISABLE_DDL

or

POSTGRES_HOST
POSTGRES_PORT
POSTGRES_USER
POSTGRES_PASSWORD
POSTGRES_DBNAME
POSTGRES_SSLMODE
POSTGRES_DISABLE_DDL
```

From Environment (multiple hosts):
```text
POSTGRES_HOSTS=host1,host2,host3
POSTGRES_PORT=5432
POSTGRES_USER=postgres
POSTGRES_PASSWORD=yourpassword
POSTGRES_DBNAME=yourdb
POSTGRES_SSLMODE=disable
POSTGRES_DISABLE_DDL=false
```

Configuring with labels for usage with Swarm and docker-proxy (https://github.com/lucaslorentz/caddy-docker-proxy):
```yaml
deploy:
  labels:
    # Set Storage definitions
    caddy_0.storage: postgres
    caddy_0.storage.connection_string: postgres://user:password@localhost:5432/postgres?sslmode=disable
    caddy_0.storage.disable_ddl: false
```
or
```yaml
deploy:
  labels:
    # Set Storage definitions
    caddy_0.storage: postgres
    caddy_0.storage.host: localhost
    caddy_0.storage.port: "5432"
    caddy_0.storage.user: postgres
    caddy_0.storage.password: postgres
    caddy_0.storage.dbname: certmagictest
    caddy_0.storage.sslmode: disable
    caddy_0.storage.disable_ddl: false
```

# Build vanilla Docker image
```Dockerfile
# Version to build
ARG CADDY_VERSION="2.6.2"

FROM caddy:${CADDY_VERSION}-builder AS builder

RUN xcaddy build \
    --with github.com/yroc92/postgres-storage

FROM caddy:${CADDY_VERSION}-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

# Build Docker image with docker-proxy support and Cloudflare resolver support
```Dockerfile
# Version to build
ARG CADDY_VERSION="2.6.2"

FROM caddy:${CADDY_VERSION}-builder AS builder

RUN xcaddy build \
    --with github.com/yroc92/postgres-storage            \
    --with github.com/lucaslorentz/caddy-docker-proxy/v2 \
    --with github.com/caddy-dns/cloudflare

FROM caddy:${CADDY_VERSION}-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

CMD ["caddy", "docker-proxy"]
```

# LICENSE

MIT
