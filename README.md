# Dockyard templates

Template files for [Dockyard](https://github.com/EliasL-git/dockyard), the Docker panel. The panel
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
  "icon": "📦",
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
| `icon` | yes | used when the panel has no brand mark for the slug |
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

These 17 match the catalog the panel ships. Editing one here changes what every
install offers, which is the point, but it also means the panel's own copy is no longer what you
see: a change here is a deliberate override, not a patch.

| Slug | Name | Category | Image |
| --- | --- | --- | --- |
| `postgres` | PostgreSQL | database | `postgres:16-alpine` |
| `mysql` | MySQL | database | `mysql:8.4` |
| `redis` | Redis | database | `redis:7.4-alpine` |
| `mongodb` | MongoDB | database | `mongo:7.0` |
| `adminer` | Adminer | devtools | `adminer:4` |
| `nginx` | nginx | web | `nginx:1.27-alpine` |
| `httpd` | Apache httpd | web | `httpd:2.4-alpine` |
| `wordpress` | WordPress | web | `wordpress:6.7-apache` |
| `node-app` | Node.js app | devtools | `node:22-alpine` |
| `python-app` | Python app | devtools | `python:3.12-slim` |
| `uptime-kuma` | Uptime Kuma | monitoring | `louislam/uptime-kuma:1.23.16` |
| `grafana` | Grafana | monitoring | `grafana/grafana:11.4.0` |
| `prometheus` | Prometheus | monitoring | `prom/prometheus:v3.13.4` |
| `minio` | MinIO | storage | `minio/minio:RELEASE.2025-09-07T16-13-09Z` |
| `n8n` | n8n | other | `n8nio/n8n:1.123.84` |
| `rabbitmq` | RabbitMQ | messaging | `rabbitmq:4-management` |
| `whoami` | whoami | devtools | `traefik/whoami:v1.11.0` |

Extras, which the panel does not ship on its own:

| Slug | Name | Category | Image |
| --- | --- | --- | --- |
| `gitea` | Gitea | devtools | `gitea/gitea:1.27.3` |
| `syncthing` | Syncthing | storage | `syncthing/syncthing:2.1.6` |
| `filebrowser` | File Browser | storage | `filebrowser/filebrowser:v2.63.23` |
| `mariadb` | MariaDB | database | `mariadb:11.4` |
| `caddy` | Caddy | web | `caddy:2.11.7` |
| `memcached` | Memcached | database | `memcached:1.6.45` |
| `vaultwarden` | Vaultwarden | storage | `vaultwarden/server:1.37.4` |

## Precedence

A panel resolves a template from the highest source that offers it:

1. a template someone edited in the panel
2. a `*.json` file on that install's own host
3. this repository
4. the catalog compiled into the panel

So an install can override anything here with a local file, and pulling can never undo that. See
`deploy/template-examples/` in the Dockyard repository for the local-file form of the same thing.

## Generated, mostly

The files here are produced from the panel's own catalog by
`scripts/gen-templates-repo.mjs`, so a built-in template and its file cannot drift apart. Extras
are copied from `deploy/template-examples/`. Editing a file here by hand is fine, but the next
run of the generator will overwrite it: make the change in the panel's catalog instead if it is a
template the panel ships.
