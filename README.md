# humind-workflows

Shared reusable GitHub Actions workflows for the Humind platform.

## Available Workflows

### `slot-deploy-region.yml`
**Used by:** humind-backend, humind-website

Zero-downtime slot-based deployment for a single Azure App Service region.

**Flow:** Deploy to staging slot → Health check → Swap to production → Verify → Auto-rollback on failure

```yaml
uses: Castieldan/humind-workflows/.github/workflows/slot-deploy-region.yml@main
with:
  region_label: 'Europe'
  app_name: 'app-humind-api-prod-001'
  resource_group: 'rg-humind-api-prod-eu'
  staging_url: 'https://...staging.azurewebsites.net'
  production_url: 'https://...azurewebsites.net'
  artifact_name: 'node-app'
  artifact_filename: 'deploy.zip'
  health_check_path: '/health'
  env_staging: 'Production-Europe-Staging'
  env_production: 'Production-Europe'
secrets: inherit
```

**Outputs:** `deployment_status` — `staging-only`, `healthy`, `rolled-back`, or `unhealthy`

---

### `blob-deploy.yml`
**Used by:** humind-B2B, humind-merchant-embed, humind-quiz-elements

Deploy static files to Azure Blob Storage with CDN cache purge.

```yaml
uses: Castieldan/humind-workflows/.github/workflows/blob-deploy.yml@main
with:
  storage_account_name: 'sthumindappprod001'
  artifact_name: 'build-output-prod'
  front_door_resource_group: 'rg-humind-api-prod-eu'
  front_door_profile: 'afd-humind-prod'
  front_door_endpoint: 'fde-humind-b2b-prod'
  environment_name: 'production'
  # Optional features (all default to false):
  enable_cache_control: true    # Add Cache-Control headers per file type
  enable_validation: true       # Check for 0-byte files after upload
  enable_cleanup: true          # Remove stale files from previous deploys
  has_i18n: true                # Override i18n file cache headers
secrets: inherit
```

---

### `create-release.yml`
**Used by:** all Humind repositories

Create a versioned GitHub Release with deployment artifact attached.

```yaml
uses: Castieldan/humind-workflows/.github/workflows/create-release.yml@main
with:
  version: '20260223-a1b2c3d'
  artifact_name: 'node-app'        # Empty string for Docker-based deploys
  environment: 'production'         # or 'preprod' (marks as prerelease)
  docker_image: ''                  # Optional: Docker image URI for release notes
secrets: inherit
```

## Version Format

All deployments use the format `YYYYMMDD-<short-sha>` (e.g., `20260223-a1b2c3d`).

- **Date prefix:** Chronologically sortable
- **Short SHA:** Unique, traceable to exact commit
- **Tag format:** `v20260223-a1b2c3d`
