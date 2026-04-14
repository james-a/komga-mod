# Komga fork (personal)

This tree is based on [gotson/komga](https://github.com/gotson/komga) with a small **Mylar** change so imported series titles include the **publication year** when `metadata.year` is present in `series.json`. That helps ComicRack **CBL** matching when multiple runs share the same base name.

**Remotes:** If you cloned the official repo, point `origin` at **your** GitHub fork (do not push personal patches to `gotson/komga`):

`git remote rename origin upstream` then `git remote add origin https://github.com/<you>/komga.git`

## Code changes

- `komga/.../mylar/MylarSeriesProvider.kt` — title / titleSort include Mylar `year` in the form `Name (year)` (same as upstream expectation that `year` exists in `series.json`).
- `komga/.../mylar/dto/MylarMetadata.kt` — unchanged from upstream aside from your branch history; `year` remains a required field in the DTO (invalid/missing `year` in JSON still fails deserialization tests).

After upgrading from upstream, re-run tests touching Mylar if those files conflict.

## Local development

- JDK **21+**, Node **18+** (see `.nvmrc`).
- Run unit tests: `./gradlew :komga:test --tests "org.gotson.komga.infrastructure.metadata.mylar.MylarSeriesProviderTest"`
- Full backend run (see `DEVELOPING.md`): `./gradlew :komga:bootRun` with Spring profiles as documented.

## Docker image via CI (recommended)

1. Push this repo to **GitHub** (fork or private mirror).
2. Open **Actions** → **Docker image (fork)** → **Run workflow**.
3. Pull on your NAS, e.g.  
   `docker pull ghcr.io/<your-github-username>/komga:<version>-custom`  
   (see `gradle.properties` for `<version>`; the workflow also tags `latest`).

Use the image like the official Komga Docker image; configuration paths are unchanged.

### GitHub Container Registry

Packages from `GITHUB_TOKEN` are tied to the repo. Make the package **public** or add credentials on the NAS if you need a private registry.

## Refresh metadata in Komga

After deploying the new image, use your library’s **refresh metadata** (or equivalent) so existing series pick up the new titles from Mylar sidecars.

## Rebasing on upstream releases

```bash
git remote add upstream https://github.com/gotson/komga.git   # once
git fetch upstream
git checkout main   # or your branch
git rebase upstream/master   # resolve conflicts in the files above if needed
./gradlew :komga:test --tests "org.gotson.komga.infrastructure.metadata.mylar.MylarSeriesProviderTest"
git push --force-with-lease   # if you rebased a published branch
```

Then bump `README-FORK.md` / tag / CI run as you prefer.
