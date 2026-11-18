# Cockpit

The [Energiegedrag Cockpit](https://github.com/energiegedrag-nl/energiegedrag-cockpit) is the web portal in which researchers set up NeedForHeat campaigns, invite participants, flash and register devices, and see whether data keeps coming in. The stack has two containers:

- `cockpit-<env>`: the Cockpit (nginx). It proxies `/api/` to the API container of the same environment over the Docker network `web`, so the admin routes (`/admin`) never pass the public proxy;
- `cockpit-mirror-<env>`: the firmware mirror for the flash station. It checks every release (SHA-256, flash offsets, no secure boot or flash encryption) and writes it to a volume that the Cockpit serves under `/firmware/`.

Follow the steps in the [deploying section of the main README](../README.md#deploying) to create the stack on Portainer, using the compose path and environment variables below.

## Before you deploy

1. Back up the database of API v3 and check that you can restore it. The API migrates the schema at start-up.
2. Deploy or update [API v3](../api/README.md) (release 2.6.0 or later: it has the `/admin` routes). Its public route excludes `/v3/admin`.
3. Create an admin token for the Cockpit, with an end date:
    ```shell
    docker exec -it needforheat-api-v3-prd needforheat-server-api admin create -n <name> -e <yyyy-mm-dd>
    ```
4. The images are on GHCR (`ghcr.io/energiegedrag-nl/energiegedrag-cockpit`, `ghcr.io/energiegedrag-nl/energiegedrag-cockpit-mirror`). If the packages are private, add a GHCR registry with a token (scope `read:packages`) in Portainer.
5. Add a DNS record for `cockpit.energiegedrag.nl` (production) or `cockpit.tst.energiegedrag.nl` (test).

## Compose path

The compose path for this stack is:

### Production
```
cockpit/prd/docker-compose.yml
```

### Test
```
cockpit/tst/docker-compose.yml
```

## Environment variables

### `IP_Whitelist`

The addresses (or CIDR ranges) from which the Cockpit may be opened, e.g. the office network or the VPN. Everyone else gets 403. The Cockpit holds admin tokens: do not leave this open.

Example values: `127.0.0.1/32, 192.168.1.7`

> Read more about it in the [Traefik documentation](https://doc.traefik.io/traefik/middlewares/http/ipwhitelist/).

## Using it

Open the Cockpit and paste the admin token. It is kept in the browser tab only (`sessionStorage`). The flash station needs Chrome or Edge (Web Serial). Flash with a full erase when a device is reused, so no measurements or settings of an earlier participant stay on it.
