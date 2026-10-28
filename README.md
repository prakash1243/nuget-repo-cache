# NuGet.org cache

This repository runs a persistent, read-through [BaGetter](https://www.bagetter.com/) cache in Docker. The first request for a package is fetched from NuGet.org and stored in the Docker volume; later requests are served from the cache.

## Start the cache

Requirements: Docker with Docker Compose.

1. Create the local environment file:

   ```powershell
   Copy-Item .env.example .env
   ```

2. Replace `BAGETTER_API_KEY` in `.env` with a long random value. Restore clients do not need this key; it prevents unauthorized package uploads.
3. Start the service:

   ```powershell
   docker compose up -d
   ```

4. Confirm it is healthy:

   ```powershell
   Invoke-RestMethod http://localhost:5000/health
   ```

The feed is available at `http://localhost:5000/v3/index.json`. Data remains in the `nuget-repo-cache_nuget-cache-data` Docker volume across container restarts and upgrades.

## Point other repositories at the cache

Copy [`examples/NuGet.Config`](examples/NuGet.Config) to each consuming repository as `NuGet.Config`. Replace `CACHE_HOST` with the DNS name or IP address of the machine running this service, then run:

```powershell
dotnet restore
```

The `<clear />` entry makes the cache the only package source. BaGetter retrieves cache misses from `https://api.nuget.org/v3/index.json` automatically.

The example explicitly permits HTTP for trusted local networks. Put the service behind an HTTPS reverse proxy and change the source URL to `https://...` before exposing it outside a trusted network; remove `allowInsecureConnections` when HTTPS is enabled.

## Operations

```powershell
# Follow server logs
docker compose logs -f nuget-cache

# Stop without deleting cached packages
docker compose down

# Update to the image version pinned in compose.yaml
docker compose pull
docker compose up -d
```

Do not run `docker compose down --volumes` unless the package cache and its index should be deleted.
