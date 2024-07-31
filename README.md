### README

# Project Overview

This project provides a complete environment using Docker for setting up and running N8N, PostgreSQL, and Redis, all in their community versions. This environment has been tested and used successfully, but it is provided as-is without any warranties.

## Project Structure

### Files

- **docker-compose.yml**: Configuration file to orchestrate the Docker services.
- **.env**: Environment variables file for configuring the services.
- **init-data.sh**: Initialization script for setting up the PostgreSQL database.

### Dependencies

- **N8N**: A workflow automation tool.
- **PostgreSQL**: Relational database.
- **Redis**: In-memory data structure store.

## Docker Compose Configuration

The `docker-compose.yml` file configures and starts the services for N8N, PostgreSQL, and Redis. It also mounts the `.env` file to pass the necessary configuration variables.

### Content of docker-compose.yml

```yaml
version: '3.1'

services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_HOST=postgres
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_BASIC_AUTH_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_BASIC_AUTH_PASSWORD}
    depends_on:
      - postgres
      - redis
    volumes:
      - ~/.n8n:/home/node/.n8n

  postgres:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    volumes:
      - ./init-data.sh:/docker-entrypoint-initdb.d/init-data.sh

  redis:
    image: "redis:6.0"
    restart: always
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

## Environment Variables (.env)

The `.env` file contains the environment variables required for configuring the environment. Here is the provided configuration:

```env
# The top level domain to serve from
DOMAIN_NAME=localhost
# The subdomain to serve from
SUBDOMAIN=n8n

# DOMAIN_NAME and SUBDOMAIN combined decide where n8n will be reachable from
# above example would result in: https://n8n.example.com

# Optional timezone to set which gets used by Cron-Node by default
# If not set New York time will be used
GENERIC_TIMEZONE=America/Sao_Paulo

POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=   

POSTGRES_NON_ROOT_USER=
POSTGRES_NON_ROOT_PASSWORD=
```

## Initialization Script (init-data.sh)

This initialization script is used to set up a non-root user in the PostgreSQL database.

### Content of init-data.sh

```bash
#!/bin/bash
set -e;

if [ -n "${POSTGRES_NON_ROOT_USER:-}" ] && [ -n "${POSTGRES_NON_ROOT_PASSWORD:-}" ]; then
	psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
		CREATE USER ${POSTGRES_NON_ROOT_USER} WITH PASSWORD '${POSTGRES_NON_ROOT_PASSWORD}';
		GRANT ALL PRIVILEGES ON DATABASE ${POSTGRES_DB} TO ${POSTGRES_NON_ROOT_USER};
	EOSQL
else
	echo "SETUP INFO: No Environment variables given!"
fi
```

## Licenses

- **N8N**: Licensed under the [Apache 2.0 License](https://github.com/n8n-io/n8n/blob/master/LICENSE).
- **PostgreSQL**: Licensed under the [PostgreSQL License](https://www.postgresql.org/about/licence/).
- **Redis**: Licensed under the [BSD 3-Clause License](https://github.com/redis/redis/blob/unstable/COPYING).

This project is provided as-is, without any warranties of any kind.