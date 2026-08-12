# Komga fork (personal)

This tree is based on [gotson/komga](https://github.com/gotson/komga) with a small **Mylar** change so imported series titles include the **publication year** from `series.json` (`Name (year)`). That helps ComicRack **CBL** matching when multiple runs share the same base name.

**Repo:** [james-a/komga-mod](https://github.com/james-a/komga-mod)  
**Remotes:** Keep `origin` on this fork. Track upstream as:

```bash
git remote add upstream https://github.com/gotson/komga.git   # once
```

Do not push personal patches to `gotson/komga`.

## Code changes

- `komga/.../mylar/MylarSeriesProvider.kt` — `title` / `titleSort` always use `Name (year)` when Mylar metadata is applied.
- `komga/.../mylar/MylarSeriesProviderTest.kt` — expectations updated for the year suffix.
- `.github/workflows/docker-fork.yml` — fork-only GHCR image build (not in upstream).
- `fork.build` — integer iterate for image tags (`{komgaVersion}-mod.{n}`).

`MylarMetadata.year` stays a required field (same as upstream).

## Local development

- JDK **21+**
- Node per UI project: `komga-webui/.nvmrc` (currently 22), `next-ui/.nvmrc` (currently 24)
- Unit test for the patch:  
  `./gradlew :komga:test --tests "org.gotson.komga.infrastructure.metadata.mylar.MylarSeriesProviderTest"`
- Full run: see `DEVELOPING.md` (`./gradlew :komga:bootRun`). Komga serves both WebUI and NextUI; a full jar needs both UIs built and copied (as in CI).

## Docker image via CI

1. Open **Actions** → **Docker image (fork)** → **Run workflow**.
2. Image tags:
   - `ghcr.io/james-a/komga:latest`
   - `ghcr.io/james-a/komga:<version>-mod.<n>`  
     Example after the 1.26.1 sync: `ghcr.io/james-a/komga:1.26.1-mod.1`
3. Pull on the NAS and point the compose/stack at that image (same config paths as official Komga).

**`fork.build`:** bump when publishing another image for the **same** Komga `version=` in `gradle.properties`. After syncing to a **new** upstream version, reset `fork.build` to `1`.

### GitHub Container Registry

Packages from `GITHUB_TOKEN` are tied to the repo. Make the package **public**, or ensure the NAS can authenticate to GHCR if it stays private.

## Metadata refresh

Only needed if existing series still have old titles without the year. New imports or libraries already titled correctly do not require a refresh. When needed: library **refresh metadata** so Mylar `series.json` is re-applied.

## Syncing upstream

```bash
git fetch upstream
git checkout master
git merge upstream/master   # or rebase if you prefer a linear history
./gradlew :komga:test --tests "org.gotson.komga.infrastructure.metadata.mylar.MylarSeriesProviderTest"
```

After a large sync, check whether `.github/workflows/release.yml` changed how UIs / `bootJar` / Docker / JReleaser work, and update `docker-fork.yml` if needed. Then reset or bump `fork.build`, run **Docker image (fork)**, and deploy.

## Optional / later

- **Chromatic**, **Dispatch events**, and **Discord announce release** are gated with `if: github.repository_owner == 'gotson'` so they no-op on this fork.
- Install [GitHub CLI](https://cli.github.com/) (`gh`) and authenticate for Actions logs, runs, and PR checks from the terminal.

  ```powershell
  gh auth login --hostname github.com --git-protocol https --web
  gh auth status
  gh run list --repo james-a/komga-mod --limit 5
  ```
