# Dockyard templates

Template files for [Dockyard](https://github.com/dockyardapp/dockyard), the Docker panel. The panel
pulls this repository live and serves what it finds here, so a template added in this repository
appears in every install on the next refresh. No release, no rebuild, no restart.

24 templates: 17 that ship with the panel, and 7 extras.

## Adding a template

Add a `*.json` file to `templates/`. Name it after the template's `slug`. That is the whole
process.

```json
{
  "schemaVersion": 1,
  "slug": "my-service",
  "name": "My Service",
  "category": "other",
  "icon": "template",
  "description": "One line, shown on the card.",
  "image": "vendor/image",
  "tag": "1.2.3",
  "ports": [{ "container": 8080, "label": "Web UI", "defaultHost": 8080 }],
  "env": [
    {
      "key": "ADMIN_PASSWORD",
      "label": "Admin password",
      "required": true,
      "secret": true,
      "description": "Required. Set before deploying."
    }
  ],
  "volumes": [{ "container": "/data", "label": "Data", "named": true }],
  "restartPolicy": "unless-stopped"
}
```

Rules worth knowing:

- The `slug` is the identity. Two files claiming one slug is an error, and the panel reports it
  rather than picking one.
- `image` and `tag` must be real and published. Pin a tag; do not use `latest`.
- A file whose name starts with `.` or `_` is ignored by the panel, which is how you park a
  template without deleting it.
- Unknown fields are rejected, not ignored, so a typo in `restartPolicy` is reported instead of
  quietly doing nothing.
- `secret: true` masks the value in the deploy preview and stores `***` instead of the value, so a
  password never lands in the panel's database in the clear.

### Field reference

| Field | Required | Notes |
| --- | --- | --- |
| `schemaVersion` | yes | `1` |
| `slug` | yes | lowercase kebab-case, unique across the catalog |
| `name` | yes | shown on the card |
| `category` | yes | `database`, `web`, `monitoring`, `storage`, `devtools`, `messaging`, `other` |
| `icon` | yes | the name of a glyph in the panel's own icon set, shown when the panel has no brand mark for the slug. Not a pictograph: the card shows the product's real logo wherever one exists |
| `description` | yes | one line |
| `image` | yes | the upstream image, without the tag |
| `tag` | yes | a real published tag |
| `ports[]` | no | `{ container, label?, defaultHost? }` |
| `env[]` | no | `{ key, label?, default?, required?, secret?, description? }` |
| `volumes[]` | no | `{ container, label?, named? }` |
| `restartPolicy` | yes | `no`, `always`, `unless-stopped`, `on-failure` |
| `healthcheck` | no | `{ test: string[], intervalSec, timeoutSec, retries }` |
| `command` / `entrypoint` | no | string arrays, passed through to Docker |
| `notes` | no | shown under the deploy form |
| `docsUrl` | no | linked from the deploy form |

## What is in here

These 24 templates **are** the panel's catalog. Dockyard compiles none of them in, so an install
offers exactly what this repository held the last time it pulled. Adding a template here adds it to
every Dockyard in the world on the next refresh: no release, no rebuild, no restart.

| Slug | Name | Category | Image |
| --- | --- | --- | --- |
| `adminer` | Adminer | devtools | `adminer:4` |
| `caddy` | Caddy | web | `caddy:2.11.7` |
| `filebrowser` | File Browser | storage | `filebrowser/filebrowser:v2.63.23` |
| `gitea` | Gitea | devtools | `gitea/gitea:1.27.3` |
| `grafana` | Grafana | monitoring | `grafana/grafana:11.4.0` |
| `httpd` | Apache httpd | web | `httpd:2.4-alpine` |
| `mariadb` | MariaDB | database | `mariadb:11.4` |
| `memcached` | Memcached | database | `memcached:1.6.45` |
| `minio` | MinIO | storage | `minio/minio:RELEASE.2025-09-07T16-13-09Z` |
| `mongodb` | MongoDB | database | `mongo:7.0` |
| `mysql` | MySQL | database | `mysql:8.4` |
| `n8n` | n8n | other | `n8nio/n8n:1.123.84` |
| `nginx` | nginx | web | `nginx:1.27-alpine` |
| `node-app` | Node.js app | devtools | `node:22-alpine` |
| `postgres` | PostgreSQL | database | `postgres:16-alpine` |
| `prometheus` | Prometheus | monitoring | `prom/prometheus:v3.13.4` |
| `python-app` | Python app | devtools | `python:3.12-slim` |
| `rabbitmq` | RabbitMQ | messaging | `rabbitmq:4-management` |
| `redis` | Redis | database | `redis:7.4-alpine` |
| `syncthing` | Syncthing | storage | `syncthing/syncthing:2.1.6` |
| `uptime-kuma` | Uptime Kuma | monitoring | `louislam/uptime-kuma:1.23.16` |
| `vaultwarden` | Vaultwarden | storage | `vaultwarden/server:1.37.4` |
| `whoami` | whoami | devtools | `traefik/whoami:v1.11.0` |
| `wordpress` | WordPress | web | `wordpress:6.7-apache` |

## Precedence

A panel resolves a template from the highest source that offers it:

1. a template someone edited in the panel
2. a `*.json` file on that install's own host
3. this repository

This repository is the lowest of the three, so a pull can never undo local work: an install can
override anything here with a local file, and a template edited in the panel always wins. Correcting
a template that ships to everyone means changing it here.

See `deploy/template-examples/` in the Dockyard repository for the local-file form of the same
thing.

## Editing these files

There is no generator any more: the panel used to compile a catalog into the image and this
repository was produced from it, which meant two copies of every template that could drift apart.
The panel now pulls from here instead, so these files are the only copy. Edit them directly.
