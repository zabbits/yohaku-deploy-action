# Yohaku Private Docker Image Action

This fork builds the private Yohaku standalone artifact with GitHub Actions, then packages it into a private Docker image and pushes it to GHCR.

It is based on the original `yohaku-deploy-action` workflow, but it no longer copies files to a server over SSH. The build still follows the same artifact path:

1. checkout `innei-dev/Yohaku`
2. install dependencies with `pnpm`
3. run `pnpm --filter @yohaku/web build:ci`
4. prepare the Next.js standalone output
5. zip `.next` into `release.zip`
6. build a runtime image from that artifact

The workflow does not use `Yohaku/Dockerfile` or `Yohaku/docker-compose.yml`.

## Image

Default image name:

```txt
ghcr.io/zabbits/yohaku
```

Each successful build pushes:

- `ghcr.io/zabbits/yohaku:latest`
- `ghcr.io/zabbits/yohaku:<source-short-sha>`

The workflow publishes a multi-architecture image for `linux/amd64` and `linux/arm64`.

The runtime image uses `node:lts-bookworm-slim`, installs `pnpm` and `sharp`, then starts Yohaku with `node server.js`. Runtime environment variables are read from the container environment.

## Configuration

Workflow defaults live in `.github/workflows/deploy.yml`.

| Variable | Default | Description |
| --- | --- | --- |
| `SOURCE_REPO` | `innei-dev/Yohaku` | Private source repository to build. |
| `BUILD_COMMAND` | `pnpm --filter @yohaku/web build:ci` | Yohaku build command. |
| `STANDALONE_SUBPATH` | `standalone/apps/web` | Path to the standalone app inside the zipped `.next` artifact. |
| `IMAGE_NAME` | `ghcr.io/zabbits/yohaku` | GHCR image name to push. |
| `HASH_FILE` | `build_hash` | Stores the last built source short SHA. |

## Secrets

Add these in GitHub repository settings under **Secrets and variables -> Actions**:

- `GH_PAT`: GitHub token that can read the private `SOURCE_REPO`.
- `BASE_URL`: build-time site URL.
- `NEXT_PUBLIC_API_URL`: build-time public API URL.
- `NEXT_PUBLIC_GATEWAY_URL`: build-time public gateway URL.

The GHCR push uses the workflow `GITHUB_TOKEN` with `packages: write`; no separate registry password is required for publishing from this repository.

The image is labeled with `org.opencontainers.image.source=https://github.com/zabbits/yohaku` so GitHub can associate the package with the private `zabbits/yohaku` repository.

## Run

Authenticate to GHCR with a token that can read the private package:

```sh
echo "$GHCR_TOKEN" | docker login ghcr.io -u zabbits --password-stdin
docker pull ghcr.io/zabbits/yohaku:latest
```

Run the container with your runtime environment variables:

```sh
docker run -d \
  --name yohaku \
  --restart always \
  --env-file .env \
  -p 2323:2323 \
  ghcr.io/zabbits/yohaku:latest
```

The image does not bake `.env` into the artifact. Provide server-side runtime values with `--env-file` or your container platform.

`BASE_URL`, `NEXT_PUBLIC_API_URL`, and `NEXT_PUBLIC_GATEWAY_URL` are still required during the workflow build because Next.js inlines public client variables into the generated bundle.

## Rebuild behavior

The workflow compares the current source short SHA with `build_hash`. If they match, the workflow stops before building. After a successful image push, the workflow commits the new source short SHA back to `build_hash`.
