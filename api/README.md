# API v3

The [NeedForHeat Server API](https://github.com/energiegedrag-nl/needforheat-server-api) is an open souce solution that serves the NeedForHeat REST API to the [NeedForHeat GearUp App](https://github.com/energiegedrag-nl/needforheat-gearup-app) and NeedForHeat measurement devices based on NeedForHeat firmware and that used a NeedForHeat database based on MariaDB. 

Follow the steps in the [deploying section of the main README](../README.md#deploying) to create the stack on Portainer, using the compose path and environment variables below.

> **Info**
> Click [here](https://github.com/energiegedrag-nl/needforheat-server-configuration/tree/36ebeff11f1cb7c0d57a48db7ac1254c6b9c2061#api) for environment variables for API v3.
>
> The compose path there is `api/v3/<env>/docker-compose.yml`.

## Compose path

The compose path for this stack is:

### Production
```
api/v3/prd/docker-compose.yml
```

### Test
```
api/v3/tst/docker-compose.yml
```

## Environment variables

### `NFH_DSN`

This environment variable is used to set the DSN (data source name) to connect to.

Example values: `readonly_researcher:correcthorsebatterystaple@tcp(mariadb_dev:3306)/needforheat`

> The composition of the connection string is as follows: `<db_user>:<db_password>@<protocol>(<db_host>:<db_port>)/<db_name>`.
>
> It is recommended to use a user and password without special characters to avoid parsing errors.

### `NFH_BASE_URL`

This environment variable is used to set the base URL used by the Swagger UI docs.

Example values: `https://api.energiegedrag.nl/v3` or `https://api.tst.energiegedrag.nl/v3`

### `NFH_TIMEZONE`

Time zone whose offset from UTC the API sends to devices with each upload, for the time on their screen (API 2.7.0 or later). Default: `Europe/Amsterdam`.

## Admin routes

The API serves every route that needs an admin token (creating apps, campaigns, accounts, device types and data sources, and `/admin` for the [Cockpit](../cockpit/README.md)) on a separate port, 8082 (API 2.7.0 or later). That port is not published on the host: only the Cockpit stack reaches it, over the Docker network `web`. The public port 8080 has none of these routes, and the Traefik rule also excludes `/v3/admin`. For scripts with an admin token, use `https://cockpit.<domain>/api/...` (from WireGuard or the LAN).
