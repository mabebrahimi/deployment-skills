---
name: al-fanar-school-deploy
description: Use when asked to deploy or operate the Al-Fanar school stack taught in prior runs: school-dashboard, school-app, and school-api across staging or production; updating GitHub Actions secrets; checking GH Actions runs; preparing nginx vhosts; verifying HTTP health; or planning SSL handoff. Use especially when the request mentions branch-based deploys, GHCR, staging/production hosts, dashboard/app/api domains, sibling repos under Documents/Code, or ssh aliases al-fanar-staging and al-fanar-production.
---

# Al-Fanar School Deploy

This skill captures the canonical deployment workflow established in prior Codex runs for the Al-Fanar school stack. It is for future Codex agents coordinating these sibling repos:

- `/Users/aliebrahimi/Documents/Code/school-dashboard`
- `/Users/aliebrahimi/Documents/Code/school-app`
- `/Users/aliebrahimi/Documents/Code/school-api`

Use this skill when the task is to deploy, repair, verify, or document the school staging/production stack.

## Canonical scope

- `school-dashboard`: Next.js admin/dashboard frontend
- `school-app`: Next.js public/app frontend
- `school-api`: Django/ASGI backend with Postgres + Redis + Celery worker
- GitHub Actions is the primary deployment entrypoint
- GHCR is the image registry
- nginx terminates HTTP today; SSL is a separate explicit step and must not be taken without user confirmation

## Environment map

As taught in the canonical run:

- Staging host: `65.109.208.43`
- Staging SSH alias: `al-fanar-staging`
- Staging branch mapping:
  - `school-dashboard`: `dev`
  - `school-app`: `dev`
  - `school-api`: `staging`
- Staging ports:
  - dashboard `3000`
  - app `3001`
  - api `8000`
- Staging domains:
  - `al-fanar-dashboard-staging.geniusai.io`
  - `al-fanar-app-staging.geniusai.io`
  - API base URL expected by frontends: `https://al-fanar-api-staging.geniusai.io`

- Production host: `91.107.245.120`
- Production SSH alias: `al-fanar-production`
- Production branch mapping:
  - all three repos deploy from `main`
- Production ports:
  - dashboard `3000`
  - app `3001`
  - api `8000`
- Production domains:
  - `dashboard.alfanarschool.ae`
  - `app.alfanarschool.ae`
  - `api.alfanarschool.ae`
- Production frontend API base URL is currently `http://api.alfanarschool.ae` until SSL is explicitly approved and enabled

If any host, alias, branch policy, or domain differs from the current request, treat the current user message as the source of truth and update your plan before executing.

## Repo alignment

Prefer existing repo logic over inventing new deployment flows:

- Review GitHub Actions in each target repo
- Review remote deploy helpers such as `scripts/ci/deploy_compose_remote.sh`
- Review any environment or ops docs already present in the target repo

Do not duplicate existing commands or YAML patterns when a repo file already expresses them.

## Signals to use this skill

Use this skill when the user asks to:

- deploy staging or production
- update GitHub Actions secrets
- set `NEXT_PUBLIC_BASE_URL_*` or backend deploy env
- push a branch and monitor GH Actions with `gh`
- fix a failed deploy, failed `docker pull`, or bad health check
- configure nginx vhosts or prepare SSL
- bring the stack online end-to-end

## Ordered workflow

1. Identify the exact target repos, environment, domains, and branch mapping.
2. Read the target repos' deploy workflows, compose files, and remote deploy scripts before editing anything.
3. Verify current branch safety.
   - For production, confirm the deploy branch actually builds locally or in Docker before pushing.
   - For Next.js frontends, verify `NEXT_PUBLIC_BASE_URL_*` is passed as a build arg and consumed in code.
4. Verify server prerequisites on the target host.
   - Docker
   - Docker Compose plugin
   - nginx if HTTP/HTTPS routing is in scope
5. Set or update GitHub secrets with `gh secret set`.
   - Never print secret values back to the user
   - Never commit secret material
6. Trigger or allow the GitHub Actions workflow to deploy.
   - Frontends: monitor `security-scan -> build -> deploy -> notify`
   - Backend: monitor build and deploy, then inspect container health
7. Verify the deployed containers on the host.
   - `docker ps`
   - service logs
   - `curl` to localhost ports
8. Configure nginx if domain routing is requested.
   - one vhost per domain
   - proxy `dashboard -> 3000`, `app -> 3001`, `api -> 8000`
   - test with `nginx -t` before reload
9. Run smoke checks from outside the host.
   - `curl -I` against public HTTP or HTTPS endpoints
10. If SSL was not explicitly approved, stop at HTTP-only readiness and report the exact next step.

## What can be automated safely

These are normally safe to automate without extra confirmation once the target environment is clear:

- reading workflows, compose files, deploy scripts, and docs
- checking server prerequisites and installed packages
- setting GitHub secrets when the user has already provided the needed values
- creating or updating nginx HTTP vhosts
- triggering GitHub Actions deploy workflows
- watching GH Actions runs
- host-side health checks, `docker ps`, and `docker logs`
- non-destructive server installs such as `docker.io`, `docker-compose-v2`, `nginx`, and `curl`

## What requires confirmation

Require explicit confirmation before:

- enabling SSL or running Certbot
- changing production domains or branch mappings
- changing secret values you do not have from the user
- force pushing, rewriting history, or merging unrelated work into deploy branches
- introducing fallback behavior that weakens security or changes app-facing URLs
- destructive cleanup of containers, volumes, databases, or existing nginx/cert state

## Migrations and startup expectations

Backend production is expected to:

- run `collectstatic`
- run `python manage.py migrate`
- start ASGI on `0.0.0.0:8000`

Do not assume optional seed commands exist on `main`. In the canonical run, production backend startup initially failed because `seed_geography`, then `seed_access_control` and `seed_super_admin`, were unavailable on `main`. If startup commands reference missing management commands, remove only the missing commands from the production startup path and redeploy.

## Known failure patterns

Check these first:

- Frontend `NEXT_PUBLIC_BASE_URL_*` missing or wrong at build time
  - symptom: browser requests go to same-origin frontend instead of API
  - fix: update GitHub secret and rebuild image
- Frontend deploy host binding bug
  - symptom: Next container crashes with hostname resolution errors
  - fix: make deploy script write `HOSTNAME=0.0.0.0` from `APP_HOSTNAME`, not inherited runner `HOSTNAME`
- `docker pull` or `containerd` transient failure on host
  - symptom: deploy job fails during image pull with blob or rename errors
  - fix: re-run the workflow and verify current containers remain healthy
- Backend container restart loop
  - symptom: localhost health check never stabilizes
  - fix: inspect `docker logs`, then patch only the failing startup command or env mismatch
- Domain works on direct IP or Host header but not publicly
  - symptom: nginx and app are healthy, public domain still fails
  - fix: verify DNS first

## Verification checklist

A deployment is successful only when all relevant checks pass:

- GH Actions run finishes `success`
- target containers are `Up`
- localhost port checks succeed on the host
- public domain returns expected HTTP status
- frontends point to the intended API base URL for that environment
- nginx config validates with `nginx -t`

Recommended commands:

```bash
gh run list --repo <owner/repo> --workflow <workflow-file> --limit 1
gh run view <run-id> --repo <owner/repo>
gh run view <run-id> --repo <owner/repo> --log-failed
ssh <host-alias> 'docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
ssh <host-alias> 'docker logs --tail 100 <container-name>'
curl -I -sS http://<public-domain> | head -n 20
curl -I -sS https://<public-domain> | head -n 20
```

## Recovery workflow

If deploy fails:

1. Determine whether the failure is build-time, image-pull, startup, or nginx/domain.
2. Do not make broad refactors. Patch the narrow failing point.
3. Re-check container logs after each change.
4. If the fix is server-only and urgent, stabilize the host first, then push the permanent code or workflow fix.
5. Re-run the failed workflow and watch it to completion.

If nginx routing is broken:

1. Inspect active site files in `/etc/nginx/sites-available` and `/etc/nginx/sites-enabled`
2. Validate with `nginx -t`
3. Reload nginx
4. Re-test localhost upstream and public domain separately

## Approval-sensitive unknowns

Always surface these as explicit unknowns instead of guessing:

- current PAT, GHCR token, or Telegram bot token
- SSL approval status
- whether a missing production DNS record has already been requested elsewhere
- whether a production secret should be blank, inherited, or disabled
- whether missing backend seed commands should be restored to `main` or intentionally omitted

## Canonical notes from the prior run

- Production frontend deploys succeeded after ensuring `main` was deploy-safe and build-safe.
- Production backend deploy succeeded only after removing unavailable seed commands from production startup.
- HTTP-only production nginx routing was prepared intentionally before SSL.
- Public `api.alfanarschool.ae` DNS was still an explicit follow-up at the time of the canonical run, so it must be verified before claiming the stack is externally complete.
