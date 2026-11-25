# HiddenXP OS on Coolify

Create a **public Git application**, build pack **Docker Compose**, using this
repository and `/docker-compose.coolify.yml`. Pin its full Git commit SHA and
disable automatic deployments. Use the Operating Systems team and the dedicated
HiddenXP AWS server. This is a source build of the fork, not upstream images.

Set the custom build command to `docker compose --parallel 1 build` and retain
Coolify's default start command. The local source-built images use
`pull_policy: never`: Coolify builds them first, then starts those images rather
than launching a second build during `compose up`. Registry images for the
database and cache retain their normal pull behavior.

The initial host is 2 vCPU, 8 GiB RAM and 500 GiB encrypted gp3. Two API workers
and one ingestion child are configured. This is a small initial lane below the
4-vCPU sizing in `docs/deploy.md`; load capacity has not been established.

## Configuration

Set runtime variables in Coolify before deploying:

| Variable | Value |
| --- | --- |
| `HIDDENXPOS_SOURCE_SHA` | The exact pinned deployment commit |
| `SECRET_KEY`, `API_KEY`, `VAULT_MASTER_KEY` | Separate random 32-byte hex values |
| `POSTGRES_PASSWORD`, `REDIS_PASSWORD` | Separate strong URL-safe passwords |
| `BOOTSTRAP_EMAIL` | Owner login email |
| `BOOTSTRAP_PASSWORD` | Strong owner password, stored in the operator's vault |
| `PUBLIC_SITE_URL` | HTTPS site origin |
| `PUBLIC_API_URL` | Separate HTTPS API origin |
| `PUBLIC_WS_URL` | The API origin with the `wss` scheme |
| `CORS_ORIGINS` | JSON array containing only the site origin |

Keep credentials in the operator secret store and Coolify, never in Git. The
bootstrap service creates the HiddenXP organization and owner after migration.
It is idempotent and does not rotate an existing owner's password on redeploy.
Without an organization provider key the starter agents remain drafts; configure
providers through the product vault after sign-in.

Configure per-service domains for `app` on port 8000 and `frontend` on port 3000.
Publish no database/cache ports. Coolify manages proxy labels and TLS. Permit SSH
only from the Coolify control plane, and HTTP/HTTPS for the site.

## Verification and recovery

1. Validate with `docker compose -f docker-compose.coolify.yml config --quiet`
   using dummy values for every required variable.
2. In the deployment receipt record the pinned SHA, image IDs, successful
   migration/bootstrap exits, and health of the remaining five services.
3. Verify HTTPS site/API responses and owner sign-in. Set the deployment sign-up
   policy to `invite_only` through the administrator settings before admitting
   other users; the platform default is open registration.
4. Confirm database and media named volumes survive a restart. A database backup
   and the corresponding vault master key are both needed for recovery.

Use the previous known-good deployment commit to roll back application images.
Do not delete named volumes or assume an image rollback reverses database
migrations. Review each migration's recovery requirements before an upgrade.
