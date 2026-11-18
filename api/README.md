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

### `ADMIN_BIND`

The address on the host where the admin routes (`/admin`, for the [Cockpit](https://github.com/energiegedrag-nl/energiegedrag-cockpit)) are published, on port 18081. Default: `127.0.0.1`. See [Admin access](#admin-access).

## Admin access

The API serves two things on two ports (API 2.7.0 or later):

| Port in the container | What | Who | Published |
| --- | --- | --- | --- |
| 8080 | public API (`/v3/...`) | app, devices | through Traefik on `api.<domain>/v3` |
| 8081 | admin routes (`/admin/...`) | the Cockpit on a researcher's laptop | **only** on `ADMIN_BIND:18081`, a private address |

The public route also excludes `/v3/admin` in Traefik, but the routes are not on port 8080 at all. An admin token is still needed for every admin request; the private address is the second layer. Keep admin tokens personal and short-lived (`admin create -n <name> -e <yyyy-mm-dd>`, e.g. three months) and revoke them with `admin delete`.

Choose how the Cockpit reaches port 18081:

### WireGuard (recommended for a VPS and for remote access)

The server gets a WireGuard interface; each researcher's laptop (and, if wanted, phone) is a peer. Only the admin port is reachable through it.

1. On the server: `apt install wireguard`, create keys (`wg genkey | tee server.key | wg pubkey > server.pub`) and `/etc/wireguard/wg0.conf`:
    ```ini
    [Interface]
    Address = 10.8.0.1/24
    ListenPort = 51820
    PrivateKey = <contents of server.key>

    [Peer]
    # laptop of <name>
    PublicKey = <public key of the laptop>
    AllowedIPs = 10.8.0.2/32
    ```
2. Open UDP port 51820 in the firewall of the server (and of the hosting provider), then `systemctl enable --now wg-quick@wg0`.
3. Docker can only publish on 10.8.0.1 when `wg0` exists: start WireGuard before Docker with `systemctl edit docker` and
    ```ini
    [Unit]
    After=wg-quick@wg0.service
    Wants=wg-quick@wg0.service
    ```
4. Set `ADMIN_BIND=10.8.0.1` for the API stack and redeploy it.
5. On the laptop (WireGuard app), a tunnel that only routes the server's address (split tunnel), so it can run next to other tunnels as long as the address ranges do not overlap:
    ```ini
    [Interface]
    Address = 10.8.0.2/32
    PrivateKey = <private key of the laptop>

    [Peer]
    PublicKey = <contents of server.pub>
    Endpoint = <public IP or name of the server>:51820
    AllowedIPs = 10.8.0.1/32
    PersistentKeepalive = 25
    ```
6. In the Cockpit: `API_UPSTREAM=http://10.8.0.1:18081`.

Use a /24 that is not in use on your other networks (home, office, other tunnels).

### SSH tunnel (no extra service)

Keep `ADMIN_BIND=127.0.0.1` (the default). Give each researcher a user on the server that can forward only this port, without a shell:

```shell
adduser --disabled-password --gecos "" cockpit-<name>
mkdir -p /home/cockpit-<name>/.ssh
echo 'restrict,port-forwarding,permitopen="127.0.0.1:18081" <public SSH key of the researcher>' > /home/cockpit-<name>/.ssh/authorized_keys
chown -R cockpit-<name>: /home/cockpit-<name>/.ssh && chmod 700 /home/cockpit-<name>/.ssh && chmod 600 /home/cockpit-<name>/.ssh/authorized_keys
```

On the laptop, keep this running while the Cockpit is in use:

```shell
ssh -N -L 18081:127.0.0.1:18081 cockpit-<name>@<server>
```

In the Cockpit (Docker Desktop): `API_UPSTREAM=http://host.docker.internal:18081`.

### LAN (self-hosting)

On a server at home or in the office, set `ADMIN_BIND` to the server's LAN address (e.g. `192.168.1.10`); laptops on that network use `API_UPSTREAM=http://192.168.1.10:18081`. From outside, use WireGuard into that network. The public API still needs a port forward of 443 to the server, since devices upload from the participants' homes.

Never set `ADMIN_BIND` to `0.0.0.0` or a public address on a server that is reachable from the internet: Docker publishes ports past host firewalls such as ufw.
