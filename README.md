# Ateliers Sleet PoC

This repository contains a small end-to-end proof of concept for publishing a NuGet v3 static feed to Cloudflare R2 with Sleet.

See [docs/plan.md](docs/plan.md) for scope and phases, and [docs/operations.md](docs/operations.md) for the R2 setup and manual run procedure.

The GitHub Actions workflow is deliberately started with `workflow_dispatch`. It packages the sample class library, publishes it to R2, restores a sample consumer anonymously, deletes the package version with Sleet, then verifies that a restore using fresh package and HTTP cache directories fails because the package is no longer listed.

The workflow has not been run against R2 yet. Configure a protected `r2-poc` GitHub Environment and add the R2 credentials and public feed URL as Environment secrets before running it.
