# Watson's Creative Store Connector — release channel

Public release channel for the Watson's Creative WooCommerce Store Connector.
This repository exists so WordPress's native plugin updater has a stable,
unauthenticated download source.

## Why a separate public repository

The API repository is private. WordPress fetches plugin updates server-side on
an unauthenticated schedule, so it cannot read a private repository's release
assets or raw files. A small public channel solves that without exposing any
application source, credentials, or customer data.

Nothing here is secret. It contains version metadata and the installable
connector ZIP, which WordPress serves to the storefront anyway.

## Files

| File | Purpose |
| --- | --- |
| `connector-update.json` | The update manifest the connector reads |
| `RELEASES.md` | Human-readable release history |
| Releases | Versioned connector ZIPs, one per release tag |

## How the update flow works

```
1. WordPress asks plugins for updates       (WordPress core)
2. Connector fetches connector-update.json  (cached 6 hours)
3. Manifest version > installed version?
      yes -> WordPress shows "Update available"
4. Administrator clicks Update
5. WordPress validates the download host
6. WordPress downloads, unzips, replaces files
7. Connector clears its manifest cache
```

Only step 2 is custom code. Everything else is WordPress core.

## Publishing a release

1. Confirm the connector's PHP header and class constant agree:

   ```php
   * Version: 0.3.69
   const VERSION = '0.3.69';
   ```

2. Build the ZIP so the **plugin folder sits at the root** of the archive:

   ```
   watsons-creative-woocommerce-0.3.69.zip
     +-- watsons-creative-woocommerce/
         +-- watsons-creative-woocommerce.php
         +-- README.md
         +-- assets/
   ```

   A ZIP containing only files, with no enclosing folder, will not install.
   Paths must use forward slashes.

3. Record the SHA-256:

   ```
   certutil -hashfile watsons-creative-woocommerce-0.3.69.zip SHA256
   ```

4. Create a GitHub release tagged `v0.3.69` and attach the ZIP as an asset.

5. Update `connector-update.json`: `version`, `download_url`, `sha256`,
   `release_notes`, and `api_version`.

6. Confirm the manifest is reachable:

   ```
   curl -sI https://raw.githubusercontent.com/WatsonsCreativeInc/watsons-creative-connector/main/connector-update.json
   ```

## Version pairing

The connector and the API are a coupled pair. The API test suite asserts the
connector version string and hashes every connector file, so a mismatched pair
fails the release gate.

| Connector | API |
| --- | --- |
| 0.3.69 | 4.0.0-alpha5.2.82 |
| 0.3.68 | 4.0.0-alpha5.2.81 |

Deploy the API first, then release the connector.

## Scope

No API source, integration key, webhook secret, or customer data. The connector
ZIP is byte-identical to the artifact shipped in the private API repository
under `integrations/woocommerce/watsons-creative-woocommerce/`.
