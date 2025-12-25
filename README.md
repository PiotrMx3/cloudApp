## CI/CD Pipeline (GitHub Actions)

This repository uses a GitHub Actions workflow to build, package, and deploy the application automatically on every push to the `main` branch that changes anything under `app/**` (it can also be triggered manually).

### Workflow overview

The pipeline consists of three jobs executed in sequence:

1. **build-test**

   - Checks out the repository.
   - Sets up Node.js (v20).
   - Installs dependencies using `npm ci`.
   - Runs a simple `echo "Build Successful"` step to confirm the job completed successfully.

2. **build-push-docker** (depends on `build-test`)

   - Checks out the repository again.
   - Sets up Docker Buildx.
   - Logs in to GitHub Container Registry (GHCR) using `GITHUB_TOKEN`.
   - Builds a Docker image from the `./app` directory (Dockerfile in `./app`).
   - Pushes the image to GHCR with two tags:
     - `latest`
     - the current commit SHA (`${{ github.sha }}`)

3. **ssh-remote** (depends on `build-push-docker`)
   - Connects to the remote server via SSH using `appleboy/ssh-action`.
   - Executes the deployment script `scripts/deploy.sh` on the server.

### Deployment script (remote)

The `scripts/deploy.sh` script performs the deployment steps on the server:

- Navigates to the project directory (`~/Docker/slot_machine`).
- Pulls the newest images (`docker compose pull`).
- Recreates containers to apply the update (`docker compose up -d --force-recreate`).
- Cleans up unused/dangling images (`docker image prune -f`).

This ensures the server always runs the latest container image from GHCR after a successful build.
