# Edgeshield Hardening (fork modifications)

A fork of [C0nw0nk/Nginx-Lua-Anti-DDoS](https://github.com/C0nw0nk/Nginx-Lua-Anti-DDoS)
with bundled test-driven hardening. This file documents everything this fork
changes on top of upstream, and how the changes were verified live.

## Why this fork exists

Upstream is battle-tested (1.6k+ stars) and correct by default. This fork does
not rewrite it. It tightens three defaults that leave a server exposed when you
take the project out of the box, then ships a live test so the protection is
proven, not assumed.

## Changes

### 1. Whitelisted IPs no longer bypass flood protection

Upstream default:
```lua
localized.ip_whitelist_bypass_flood_protection = 1
```
Whitelisted IPs (loopback, internal ranges) skip flooding detection entirely.
On a single box where your own health checks and admin traffic come from
loopback, that is often exactly the IP a botnet spoofs or the one that should
be rate limited.

This fork:
```lua
localized.ip_whitelist_bypass_flood_protection = 0
```
Whitelist still skips the JS puzzle (so crawlers and internal tooling keep
working), but no IP is exempt from the flooding rate limit any more.

### 2. Aggressive flood thresholds (demo / high-sensitivity profile)

Upstream default allows 10,000 requests per 500 ms window per IP before a
24-hour ban. That is a generous ceiling meant for high-traffic production.

This fork (set in `lua/anti_ddos_challenge.lua` in the rate-limiting block):
- `10` requests per `0.5` s window (was `10000`)
- `120` s ban time (was `86400` s = 24 h)

These are tuned for the bundled demo and for servers that want to drop floods
early. Raise them back for production throughput.

## Live verification

Reproduced on Ubuntu 24.04 + OpenResty 1.31.1.1, shield bound to `0.0.0.0:8080`,
target on loopback:

```
40 requests, 0 delay, from one IP:
200 200 200 200 200 200 200 200 200 200
000 000 000 000 000 000 000 000 000 000 000 000 000 000 000

result: 10 allowed (below threshold), 15 connection-reset (000),
IP auto-banned, /health now returns 000 (connection closed)
other IP still served 200  -> single-IP ban does not take the site down
ban lifted after ~120s      -> /health back to 200
```

0 bytes returned for a banned connection = the flood never reaches the origin.

## Run it

```bash
# Ubuntu 24.04
apt-get update && apt-get install -y wget gnupg ca-certificates
wget -qO - https://openresty.org/package/pubkey.gpg | gpg --dearmor -o /usr/share/keyrings/openresty.gpg
echo "deb [signed-by=/usr/share/keyrings/openresty.gpg] http://openresty.org/package/ubuntu noble main" > /etc/apt/sources.list.d/openresty.list
apt-get update && apt-get install -y openresty

# deploy
cp lua/anti_ddos_challenge.lua /usr/local/openresty/nginx/conf/lua-anti-ddos.lua
# add the shared-memory zones + access_by_lua_file to nginx.conf (see HARDENING nginx.conf)
openresty
```

See upstream README for the full feature list, packaging (deb/rpm), and the
Redis/memcached storage options.
