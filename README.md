# Arithmico Project

This repository contains the source code for the Arithmico Engine and the following web applications

- Arithmico Calc
- Arithmico Docs
- Arithmico Blog

# Deployments

## Production Deployments

- [Arithmico Calc](https://arithmico.com)
- [Arithmico Docs](https://docs.arithmico.com)
- [Arithmico Blog](https://blog.arithmico.com)

## Development Deployments

- [Arithmico Calc](https://calc.dev.arithmico.com)
- [Arithmico Docs](https://docs.dev.arithmico.com)
- [Arithmico Blog](https://blog.dev.arithmico.com/)

# Build Instructions

1. Go to the project folder and run

```
npm install
```

2. Build the Arithmico Engine by running. **Important** You must define the features with `export ARITHMICO_FEATURES=$(cat ./packages/libraries/engine/features.json)` 

```
npm run build:engine
```
or use for all features
```
npm run build:full-features
```


3. Build one a web application by running one of the following commands

- Build Arithmico Calc

```
npm run build:calc
```

- Build Arithmico Docs

```
npm run build:docs
```

- Build Arithmico Blog

```
npm run build:blog
```

- Build Arithmico Config

```
npm run build:config
```

4. Serve the files under `<project-root>/packages/<app-name>/dist/*` on a web server
5. For local testing you can use `npm run start:calc` or create the offline-version with `chuccdhtttehugjbeuuvnrdehfugfhch
6. `

## Build the offline version locally

The offline frontend can be built locally from the repository root after installing the dependencies:

```
npm ci
npm run build:offline -w packages/applications/calculator
```

This creates the static offline frontend in `packages/applications/calculator/dist/`. To create the native Tauri installer instead, Rust and the platform-specific Tauri build dependencies must also be installed. Run:

```
VITE_OFFLINE_MODE=true NODE_ENV=production npm run tauri build -w packages/applications/calculator
```

The generated installer is placed below `packages/applications/calculator/src-tauri/target/release/bundle/`. Depending on the operating system, the configured bundle targets are `.deb` on Linux, `.dmg` on macOS, and `.exe` on Windows. The exact native dependencies are platform-specific; the GitHub Actions workflow documents the additional Ubuntu packages and Rust targets used by CI in `.github/workflows/build-offline-version.yml`.

# Development and Testing

The repository is an npm workspace. Run commands from the project root unless a command explicitly uses `-w` to select a workspace.

## Install dependencies

For a reproducible installation using the versions from `package-lock.json`, run:

```
npm ci
```

## Run tests

Run all workspace test targets with:

```
npx --no nx run-many -t test
```

Run tests for a single workspace with:

```
npm run test -w packages/libraries/engine
npm run test -w packages/services/api
```

The API also provides end-to-end and coverage commands:

```
npm run test:e2e -w packages/services/api
npm run test:cov -w packages/services/api
```

The frontend workspaces currently have placeholder test scripts that exit successfully without running tests. The current automated test coverage is primarily in the Engine and API workspaces.

## Run linting

```
npx --no nx run-many -t lint
```

The current lint scripts use `--fix`, so they can modify files automatically. Review those changes before committing them.

# Release Process

Releases are started by pushing a Git tag matching `v<MAJOR>.<MINOR>.<PATCH>`, for example `v2.27.0`. The tag must point to the commit that should be released.

1. Merge the changes into `main` through a pull request.
2. Install dependencies and run the tests, linting, and relevant builds locally.
3. Check the next version. `tools/auto-version.sh` determines the bump from commit messages since the last version commit:
   - `breaking:` creates a major version bump.
   - `feat:` without `breaking:` creates a minor version bump.
   - Other changes create a patch version bump.
4. Run `bash tools/auto-version.sh`, or set the version explicitly with `npm version 2.27.0`.
5. Push the version commit and the generated tag. Replace the example with the actual version:

```
git push origin main
git push origin v2.27.0
```

The tag starts `.github/workflows/build-tag.yml`. It runs all tests first. If they pass, it builds and publishes the API Docker image to Amazon ECR and builds the Blog, Documentation, Backoffice, and Calculator frontends and uploads them to S3. The workflow requires the AWS and API secrets configured in GitHub.

# CI/CD Pipeline

## Pull requests

`.github/workflows/test-pull-request.yml` runs for pull requests targeting `main` or `dev`:

- The `test` job installs dependencies with `npm ci` and runs `npx --no nx run-many -t test`.
- The `check` job installs dependencies and runs `npx --no nx run-many -t lint`.

## Tagged releases

`.github/workflows/build-tag.yml` runs for matching version tags or by manual dispatch:

1. `test-all` runs the tests for all packages.
2. `get-version-tag` extracts the Git tag for the build jobs.
3. `build-api` builds and pushes the API Docker image to Amazon ECR.
4. The four frontend jobs use `.github/workflows/actions/build-frontend/action.yml` to install dependencies, build the Engine with all features, build the selected application, and sync its `dist/` directory to S3.

## Offline version

`.github/workflows/build-offline-version.yml` is started manually from GitHub Actions. `commitHash` and `artifactPath` are required inputs. The workflow builds the Tauri calculator for Linux, macOS, and Windows in parallel, then uploads `.deb`, `.dmg`, or `.exe` artifacts to S3. Optional engine feature and webhook inputs control the build and status notifications. The current workflow requires `commitHash` in the dispatch form but does not use it in the checkout step; select the desired workflow ref before starting the build.
