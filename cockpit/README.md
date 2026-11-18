# Cockpit

The [Energiegedrag Cockpit](https://github.com/energiegedrag-nl/energiegedrag-cockpit) is the web portal in which researchers set up NeedForHeat campaigns, invite participants, flash and register devices, and see whether data keeps coming in. It runs on the NeedForHeat server, next to the API. The stack has two containers:

- `cockpit-<env>`: the Cockpit (nginx). It proxies `/api/` to the **admin port 8082** of the API container of the same environment, over the Docker network `web`. That port is not published on the host and has every route that needs an admin token; the public API (port 8080, `api.<domain>/v3`) has none of them;
- `cockpit-mirror-<env>`: the firmware mirror for the flash station. It checks every release (SHA-256, flash offsets, no secure boot or flash encryption) and writes it to a volume that the Cockpit serves under `/firmware/`.

The local vault for names and e-mail addresses of participants (`kluis`) is not part of this stack.

Follow the steps in the [deploying section of the main README](../README.md#deploying) to create the stack on Portainer, using the compose path and environment variables below.

## Security

Whoever reaches the Cockpit with a valid admin token can create accounts and invitations, see the state of every campaign and register devices. Access is therefore guarded in layers:

1. **Network:** Traefik admits only the addresses in `IP_Whitelist`: the WireGuard subnet of the server (recommended, also for a server at home) and, when self-hosting, the LAN. Everyone else gets 403. Requests are rate limited.
2. **Transport:** HTTPS with a Let's Encrypt certificate, also inside WireGuard. A secure context is needed anyway for Web Serial (flash station).
3. **Identity:** a **personal** admin token per researcher, with an end date (e.g. three months), kept in a password manager. The Cockpit keeps it only in the browser tab (`sessionStorage`). Revoke it with `admin delete` when someone leaves or a laptop is lost.
4. **No admin routes on the internet:** the API serves them only on port 8082, which only this stack reaches.

Later (phase 2 in the Cockpit design): personal logins with OIDC instead of tokens, and an audit log.

## Reaching the Cockpit through WireGuard

The server gets a WireGuard interface; each researcher's laptop (and, if wanted, phone) is a peer. Traefik already listens on port 443 of every interface, so the Cockpit is reachable at the server's WireGuard address.

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
    Use a /24 that is not in use on your other networks (home, office, other tunnels).
2. Open UDP port 51820 in the firewall of the server and of the hosting provider, then `systemctl enable --now wg-quick@wg0`.
3. On the laptop (WireGuard app), a tunnel that only routes the server's WireGuard address (split tunnel), so it can run next to other tunnels:
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
4. Let the laptop resolve the Cockpit's name to the WireGuard address, e.g. in its hosts file (`C:\Windows\System32\drivers\etc\hosts` or `/etc/hosts`):
    ```text
    10.8.0.1  cockpit.energiegedrag.nl
    ```
    The public DNS record points to the server's public address, so Let's Encrypt can issue the certificate (the TLS challenge is answered before the IP whitelist applies). From the internet the Cockpit only answers 403.
5. `IP_Whitelist = 10.8.0.0/24` (add the LAN range when self-hosting).

On a server at home or in the office, the same works with the LAN: add the LAN range to `IP_Whitelist` and let the LAN's DNS (or the hosts file) point `cockpit.<domain>` to the server's LAN address.

## Before you deploy

1. Back up the database of API v3 and check that you can restore it. The API migrates the schema at start-up.
2. Deploy or update [API v3](../api/README.md) (release 2.7.0 or later: admin routes on port 8082).
3. Create a personal admin token with an end date:
    ```shell
    docker exec -it needforheat-api-v3-prd needforheat-server-api admin create -n <name> -e <yyyy-mm-dd>
    ```
4. The images are on GHCR (`ghcr.io/energiegedrag-nl/energiegedrag-cockpit`, `ghcr.io/energiegedrag-nl/energiegedrag-cockpit-mirror`). If the packages are private, add a GHCR registry with a token (scope `read:packages`) in Portainer.
5. Add a DNS record for `cockpit.energiegedrag.nl` (production) or `cockpit.tst.energiegedrag.nl` (test) to the server's public address, and set up WireGuard as above.

## Compose path

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

The addresses (or CIDR ranges) from which the Cockpit may be opened: the WireGuard subnet and, when self-hosting, the LAN. Do not leave this open.

Example values: `10.8.0.0/24` or `10.8.0.0/24, 192.168.1.0/24`

> Read more about it in the [Traefik documentation](https://doc.traefik.io/traefik/middlewares/http/ipwhitelist/).

## Using it

Open `https://cockpit.energiegedrag.nl` with the WireGuard tunnel active and paste your admin token. The flash station needs Chrome or Edge (Web Serial). Flash with a full erase when a device is reused, so no measurements or settings of an earlier participant stay on it.
