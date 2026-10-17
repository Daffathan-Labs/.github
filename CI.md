# CI/CD Daffathan Labs

Every repo calls the reusable workflows here at `@v1`. Change them in one place, then move the `v1` tag.

| Workflow | Contents |
|---|---|
| `guard.yml` | gitleaks over the incoming commit range |
| `ci-node.yml` | npm ci → lint → typecheck → test → (e2e) → build |
| `ci-python.yml` | pip install → ruff (errors only) → pytest |
| `ci-flutter.yml` | flutter analyze → flutter test |
| `security.yml` | Trivy fs (dependencies) + Trivy config (Dockerfile) + Semgrep. `full: true` = whole repo plus an SBOM |
| `release-docker.yml` | build → Trivy image (before push) → push Docker Hub → SSH deploy → GitHub Release |

## When each part runs (repo `ci.yml`)

| Event | What runs |
|---|---|
| PR opened/updated | `guard` only (~1 min) |
| PR given the `ci` label | guard + ci + security |
| push to `main`/`dev` | guard + ci + security, **no deploy** |
| push tag `vX.Y.Z` | everything + build image + deploy + Release |
| Run workflow (manual) | everything + deploy (the `deploy` input can be turned off) |
| Mondays 03:00 WIB | full security scan + SBOM |
| docs/markdown-only change | nothing |

Deploy: `git tag v2.0.37 && git push origin v2.0.37`.

## Deliberate exceptions
- A vulnerable dependency or misconfiguration that is accepted → `.trivyignore` (one ID per line, with a `#` comment explaining why).
- A Semgrep false positive → `# nosemgrep: <rule-id>` on that line, or `.semgrepignore` for a folder.

## Secrets (repo or org)
`DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, `SERVER_HOST`, `SERVER_USER`, `SERVER_KEY`,
`SERVER_KNOWN_HOSTS` (the output of `ssh-keyscan <host>`; the SSH host key is always checked).
The caller passes them one by one under `secrets:` (not `secrets: inherit`). Secret build-args go through `BUILD_ARGS` (KEY=VALUE lines).
