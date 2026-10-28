---
description: Automatically inspect a project and create the smallest
  correct Docker Compose configuration for deployment to Dokploy. Use
  when preparing, fixing, or deploying an application to Dokploy from a
  repository.
name: dokploy-docker-compose-deploy
---

# Dokploy Docker Compose Deploy

## Goal

Deploy the project with the **minimum configuration that works**.
Inspect the repository first; do not assume its stack or add unnecessary
services, variables, networks, volumes, or infrastructure.

## Workflow

1.  **Inspect the project**
    -   Read the README, package/build manifests, Dockerfile(s),
        existing Compose files, lockfiles, and app entry points.
    -   Identify framework, build command, runtime command, required
        services, listening port, health endpoint, persistence needs,
        and environment variables.
    -   Reuse existing working configuration. Do not replace project
        architecture without a reason.
2.  **Choose the deployment shape**
    -   If the app already has a valid Dockerfile, use it.
    -   If Compose is needed, create or update `docker-compose.yml`
        using only required services.
    -   Add a database, cache, worker, or reverse-proxy container only
        when the project actually needs it and it is not provided
        externally.
    -   Prefer Dokploy's built-in domain/HTTPS routing; do not add
        Nginx/Traefik/Caddy inside the app unless explicitly required.
    -   Do not publish container ports to the host unless direct host
        access is needed. Configure the app's internal listening port
        for Dokploy routing.
    -   Bind the application to `0.0.0.0`, not `127.0.0.1`.
3.  **Create the smallest correct Compose file**
    -   Use a clear service name such as `app`.
    -   Set a valid build context and Dockerfile path.
    -   Set `restart: unless-stopped` where appropriate.
    -   Add a healthcheck only when a reliable command/endpoint exists;
        do not invent one.
    -   Add named volumes only for data that must survive container
        replacement.
    -   Use service DNS names for Compose-to-Compose connections, not
        `localhost`.
    -   Use environment variables for configuration. Never commit
        secrets or put real credentials in Compose; use Dokploy
        environment settings or an untracked `.env`.
    -   Do not hardcode container IPs, add unnecessary `container_name`,
        or define redundant networks.
    -   Do not add `version:` to modern Compose files.
    -   Pin or preserve image versions where practical; do not silently
        upgrade dependencies.
4.  **Handle project-specific cases**
    -   **Next.js / Node.js:** use the project's package manager and
        lockfile; ensure the production start command and exposed
        internal port match the app. Do not assume port `3000` if
        configuration says otherwise.
    -   **Python / API:** install dependencies from the project's actual
        manifest and use its real production entry point.
    -   **Static frontend:** use the project's existing build/output
        setup; add a web server only if required to serve the built
        assets.
    -   **Database:** preserve existing migrations and persistence
        strategy. Do not initialize a second database if the project
        uses a managed/external one.
    -   **Monorepo:** identify the correct app directory and build
        context; do not build the entire repository by default.
    -   These are examples, not permission to force a stack. Follow the
        repository evidence.
5.  **Validate before reporting**
    -   Run `docker compose config` if Docker is available.
    -   Check Dockerfile paths, build context, required environment
        variables, port consistency, persistent paths, and service
        dependencies.
    -   If possible, build the image. Do not claim a successful build or
        deployment unless the command actually succeeds.
    -   If validation cannot run, state the exact unverified item
        briefly.
6.  **Deploy to Dokploy**
    -   Explain the minimum required Dokploy settings: Compose
        file/repository path, domain and internal container port, and
        required environment variables.
    -   Use the actual project path and port discovered from the
        repository.
    -   If connected deployment tools are available and authorized, use
        them. Otherwise provide the exact concise steps/commands for the
        user.
    -   Never claim to have deployed, configured a domain, or created
        secrets unless that action was confirmed.

## Safety and change control

-   Inspect before editing; make the smallest targeted changes.
-   Preserve existing working settings and unrelated files.
-   Never delete volumes, databases, backups, or production data as a
    cleanup step.
-   Never print secrets. If credentials are missing, use clearly named
    placeholders and list only the required variable names.
-   Do not run destructive commands or migrations against production
    without explicit approval.
-   Do not add monitoring, logging stacks, reverse proxies, or extra
    services unless the project requires them.

## Response style

Be concise and action-oriented. Avoid long explanations and generic
Docker tutorials.

Report only: 1. **Detected:** stack, app path, internal port, required
services. 2. **Changed:** files created/updated. 3. **Deploy settings:**
Dokploy path/domain port/environment variables. 4. **Validation:**
passed/failed/not run, with the reason.

If one essential detail is missing, ask one direct question. Otherwise
infer from repository evidence and proceed; do not ask the user to
confirm obvious defaults.
