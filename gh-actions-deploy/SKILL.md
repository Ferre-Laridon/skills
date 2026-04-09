---
name: gh-actions-deploy
description: >
  Creates GitHub Actions CI/CD workflows for building Docker images and pushing them to a Harbor registry,
  with optional Balena fleet deployment. Use this skill whenever the user wants to create, update, or troubleshoot
  a GitHub Actions workflow that involves Docker builds, Harbor registry, or Balena deployment — even if they
  just say "set up a pipeline", "add CI/CD", "help me deploy to harbor", "create a workflow for my services",
  or "I need an action to push images". Trigger even when the description is rough — the skill will gather
  what it needs through targeted questions.
---

# GitHub Actions: Harbor & Balena Deployment Workflows

You are helping the user create or update a `.github/workflows/*.yml` file that builds Docker images and pushes
them to a **Harbor** container registry, optionally followed by a **Balena** fleet push.

Follow the interview → design → generate loop below. The goal is a complete, copy-pasteable workflow YAML.

---

## Step 1: Interview the User

Ask these questions (adapt based on what the user already provided). Group related ones to avoid a long quiz:

### Project basics
- What is the **workflow name** / what does this project do?

### Versioning & environment tiers
- How is the **image tag** determined?
  - Option A: **Git tag-based** — workflow triggers on `push: tags: ["v*"]`, git tag becomes the image tag
  - Option B: **Manual dispatch** — user picks an environment (`dev`/`prod`) as input
- Does the workflow need **environment tiers** based on the semver tag format?
  - e.g., `v1.2.3` → production, `v1.2.3-rc.1` → staging, `v1.2.3-alpha.1` → acceptance, anything else → development
  - Only suggest this if the project has multiple deployment environments tied to release maturity

### Services to build
- What **services/components** need a Docker image built?
- For each service, ask:
  - Docker **context directory** (e.g., `./backend`, `./apps/collector_docker`)
  - **Dockerfile path** (e.g., `./backend/Dockerfile`, or same as context)
  - **Runner**: `ubuntu-latest` (default, AMD64) or `vintecc-ubuntu-arm64` (ARM builds)
  - **Extra platforms**: any multi-platform builds? (`linux/arm64,linux/amd64`)
  - **Build args** needed? (e.g., `NPM_TOKEN`, `FONTAWESOME_TOKEN`)

### Registry & image paths
- **Harbor registry URL** (commonly `harbor.mgmt.vintecc.cloud`)
- **Project/namespace path** in Harbor (e.g., `vyncke`, `bekaert`, `capture-cloud`)
- **Image tag strategy** for each service:
  - Single tag: `$REGISTRY/$PROJECT/$SERVICE:$TAG`
  - Environment-scoped: `$REGISTRY/$PROJECT/$ENV/$SERVICE:$TAG` (e.g., `bekaert/dev/frontend`)
  - Multiple tags: versioned + `latest` + env name
- **`IMAGE` env var pattern** — define both `REGISTRY` and a full `IMAGE` path at the top level for clarity:
  ```yaml
  env:
    REGISTRY: ${{ vars.HARBOR_MGMT_VINTECC_URL }}
    IMAGE: ${{ vars.HARBOR_MGMT_VINTECC_URL }}/capture-cloud/data-sink
  ```

### Secrets & credentials
- Which Harbor **secret names** are used? Common patterns:
  - `HARBOR_ROBOT_<PROJECT>_USERNAME` / `PASSWORD`
  - `HARBOR_MGMT_VINTECC_ROBOT_USER` / `HARBOR_MGMT_VINTECC_ROBOT_PWD`
  - Or as **vars** (not secrets): `HARBOR_MGMT_VINTECC_URL` for the registry URL

### Balena deployment (only ask if relevant)
- Does the workflow need to deploy to **Balena**?
- If yes:
  - Which **Balena fleet/app** does each tag tier map to? (e.g., `vintecc/myproject_development`)
  - Does it use a `docker-compose.base.yml` template with `__PLACEHOLDER__` substitutions?
  - Which environment variables are injected into the compose file? (e.g., `__APP_TAG__`, `__SERVER_URL__`)
  - **Balena token secret name** (e.g., `VINTECC_BALENA_TOKEN`)

### Post-build steps (only ask if relevant)
- Should the workflow do anything after a successful build? (e.g., notify a Slack channel, create a GitHub release)

---

## Step 2: Design Choices to Present

Before generating, summarize what you'll build and confirm:
- Trigger strategy (tag push, dispatch, or both)
- Job graph (set-env → parallel builds → [deploy])
- Environment tier mapping (if tag-based)
- Any assumptions you made

---

## Step 3: Generate the Workflow YAML

Use the patterns below as your guide. Generate the full YAML, then explain any non-obvious choices.

---

## Workflow Patterns Reference

### Trigger patterns

**Tag-based with dispatch override:**
```yaml
on:
  workflow_dispatch:
    inputs:
      tag:
        description: "Tag (e.g. v1.2.3, v1.2.3-rc.1, or develop)"
        required: true
        default: "develop"
  push:
    tags:
      - "v*"
```

**Manual dispatch with environment selection:**
```yaml
on:
  workflow_dispatch:
    inputs:
      image_prefix:
        description: "Select Environment"
        required: true
        default: "dev"
        type: choice
        options:
          - dev
          - prod
```

---

### Environment / tier detection (tag-based)

Used in a `set-env` job when the trigger is tag-based. Maps semver tags to environments:

```yaml
set-env:
  runs-on: ubuntu-latest
  outputs:
    tag: ${{ steps.set-env.outputs.tag }}
    environment: ${{ steps.set-env.outputs.environment }}
  steps:
    - name: Determine environment
      id: set-env
      run: |
        if [[ "${{ github.event_name }}" == "workflow_dispatch" ]]; then
          TAG="${{ github.event.inputs.tag }}"
        else
          TAG="${GITHUB_REF_NAME}"
        fi

        if [[ "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
          ENVIRONMENT="myproject/fleet_production"
        elif [[ "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+-rc\.[0-9]+$ ]]; then
          ENVIRONMENT="myproject/fleet_staging"
        elif [[ "$TAG" =~ ^v[0-9]+\.[0-9]+\.[0-9]+-alpha\.[0-9]+$ ]]; then
          ENVIRONMENT="myproject/fleet_acceptance"
        else
          ENVIRONMENT="myproject/fleet_development"
        fi

        echo "tag=$TAG" >> $GITHUB_OUTPUT
        echo "environment=$ENVIRONMENT" >> $GITHUB_OUTPUT
```

---

### Standard Docker build job

The most common pattern. Repeat this for each service, adjusting context/file/tags:

```yaml
build-my-service:
  runs-on: ubuntu-latest        # or vintecc-ubuntu-arm64 for ARM
  needs: set-env                # or: needs: setup (for version-bump workflows)
  steps:
    - uses: actions/checkout@v4

    - name: Login to Harbor registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ secrets.HARBOR_ROBOT_MYPROJECT_USERNAME }}
        password: ${{ secrets.HARBOR_ROBOT_MYPROJECT_PASSWORD }}

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Build and push
      uses: docker/build-push-action@v6
      with:
        context: ./my-service
        file: ./my-service/Dockerfile
        push: true
        tags: ${{ env.REGISTRY }}/myproject/my-service:${{ needs.set-env.outputs.tag }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

**With build args:**
```yaml
        build-args: |
          NPM_TOKEN=${{ secrets.NPM_TOKEN }}
          FONTAWESOME_TOKEN=${{ secrets.FONTAWESOME_TOKEN }}
```

**With multiple tags (version + latest + env):**
```yaml
        tags: |
          ${{ vars.HARBOR_URL }}/project/dev/service:v${{ needs.setup.outputs.new_version }}
          ${{ vars.HARBOR_URL }}/project/dev/service:latest
          ${{ vars.HARBOR_URL }}/project/dev/service:dev
```

**Multi-platform build:**
```yaml
        platforms: linux/arm64,linux/amd64
```

---

### Balena deployment job

Comes after all build jobs. Uses `balena push` with the pre-built Harbor images as the source:

```yaml
deploy-to-balena:
  runs-on: ubuntu-latest
  needs: [set-env, build-service-a, build-service-b]
  env:
    ENVIRONMENT: ${{ needs.set-env.outputs.environment }}
  steps:
    - uses: actions/checkout@v4

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: "20"

    - name: Create registry_secrets.json
      run: |
        echo '{
          "${{ env.REGISTRY }}": {
            "username": "${{ secrets.HARBOR_ROBOT_MYPROJECT_USERNAME }}",
            "password": "${{ secrets.HARBOR_ROBOT_MYPROJECT_PASSWORD }}"
          }
        }' > registry_secrets.json

    - name: Install Balena CLI
      run: |
        npm install --global balena-cli
        balena version

    - name: Log in to Balena
      run: balena login --token ${{ secrets.VINTECC_BALENA_TOKEN }}

    - name: Push to Balena
      run: |
        set -euo pipefail
        TAG="${{ needs.set-env.outputs.tag }}"

        # Substitute placeholders in compose template
        cp docker-compose.base.yml docker-compose.yml
        perl -0777 -pi -e "s|__APP_TAG__|$TAG|g" docker-compose.yml

        balena push ${{ env.ENVIRONMENT }} \
          --registry-secrets registry_secrets.json \
          --debug
```

---

## Common Secrets Reference

| Secret name | What it's for |
|---|---|
| `HARBOR_ROBOT_<PROJECT>_USERNAME` | Harbor robot account login |
| `HARBOR_ROBOT_<PROJECT>_PASSWORD` | Harbor robot account password |
| `HARBOR_MGMT_VINTECC_ROBOT_USER` | Alternative naming convention |
| `HARBOR_MGMT_VINTECC_ROBOT_PWD` | Alternative naming convention |
| `VINTECC_BALENA_TOKEN` | Balena authentication token |
| `NPM_TOKEN` | Private npm registry access |
| `FONTAWESOME_TOKEN` | FontAwesome Pro access |

| Variable name | What it's for |
|---|---|
| `HARBOR_MGMT_VINTECC_URL` | Harbor registry URL (as a repo var, not secret) |

---

## Tips for Good Workflow Design

- **GHA cache** (`type=gha`) is almost always worth including for Docker builds — it dramatically speeds up repeated builds by caching layers.
- **ARM builds** (`vintecc-ubuntu-arm64`) must run on the matching runner. Don't cross-compile with QEMU unless you specifically need multi-platform images — native builds are much faster.
- **Parallel jobs** are better than sequential for independent services — all build jobs should have the same `needs: [set-env]` or `needs: [setup]` dependency, not depend on each other.
- For **Balena**, the `registry_secrets.json` tells Balena where to pull the already-built images from, so it doesn't rebuild them — this is the expected pattern when images live in Harbor.
- If a job builds **two similar variants** (e.g., `Dockerfile-Cloud` vs `Dockerfile-Edge`), make them separate jobs so they run in parallel.
