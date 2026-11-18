[![CircleCI](https://circleci.com/gh/ovotech/circleci-orbs.svg?style=shield&circle-token=ae0a459eabe5a6b454eab8e241a516fd1a212e8c)](https://app.circleci.com/pipelines/github/ovotech/circleci-orbs)

# CircleCI Orbs

This repo contains public circleci orbs in the ovotech namespace.

## Contributing

Orbs follow the conventions:

- **Directory** named the **same** as the orb

- A single Dockerfile in the executor directory (unless you specify publish-docker-image:false)

- The orb yaml is in a file name `orb.yml`

- Published in the ovotech namespace

- An `orb_version.txt` file may exist in an orb directory containing the version of the orb. (A new version is only published when the version changes)

- An entry in [CODEOWNERS](./.github/CODEOWNERS) indicating which team maintains the orb - they will be responsible for reviewing
  any changes to the orb.

- This project uses CircleCI path filtering. The publish job only runs for orbs with file changes.
- Any new orb needs to be added to the [path filter mapping](.circleci/config.yml) and to the top and bottom of the [Circle config](.circleci/continue_config.yml)

## Published Orbs

- [ovotech/argocd](argocd) - Interact with an ArgoCD API.
- [ovotech/aws-configure-credentials-oidc](aws-configure-credentials-oidc) - Authenticate with AWS via CircleCI OIDC.
- [ovotech/aws-get-parameters@1](aws-get-parameters) - Get parameters from AWS Parameter Store.
- [ovotech/aws-rotate-keys@2](aws-rotate-keys) - Rotate AWS access keys and update corresponding CircleCI environment variables.
- [ovotech/clair-scanner@1](clair-scanner) - Scan Docker images for vulnerabilities.
- [ovotech/rac-gcp-deply@1](rac-gcp-deploy) - Deploy Scala services to a Kubernetes cluster running in Google Cloud.
- [ovotech/set-current-environment@1](set-current-environment) - Set current environment (master branch, tag or other) to environmental variable.
- [ovotech/terraform@1](terraform) - Plan and apply Terraform modules, with terraform <=0.14
- [ovotech/terraform-v2@2](terraform-v2) - Plan and apply Terraform modules, with terraform >=0.14
- [ovotech/tools-install@1](tools-install) - Download and unpack tools archives
- [ovotech/with-git-deploy-key@1](with-git-deploy-key) - Execute Git operations on another repo selecting specific public key.
- [ovotech/clojure](clojure) - test, scan and package Clojure projects built using Leiningen.
- [ovotech/oot-eks](oot-eks) - deploy services to EKS, OOT-style.
- [ovotech/oot-deploy](oot-deploy) - deploy services via Argo, OOT-style.
- [ovotech/sbt-deploy](sbt-deploy) - Deploy services via Argo, using Kustomize.
- [ovotech/jaws-journey-deploy](jaws-journey-deploy) - Provides a standard deployment process for all jaws journey repositories.
- [ovotech/pipeline-utils](pipeline-utils) - Miscellaneous bash commands to ease CircleCI pipeline development.
- [ovotech/ipa-deploy](ipa-deploy) - IPA team deployment notification templates.
- [ovotech/helm-ci](helm-ci) - CI checks for Helm charts
- [ovotech/sast](sast) - Consumer Products Production Engineering team orb to run static security analysis.
- [ovotech/setup-scheduled-pipeline](setup-scheduled-pipeline) - Consumer Products Production Engineering team orb responsible for setting up schedules for [scheduled pipelines](https://circleci.com/docs/2.0/scheduled-pipelines/#get-started).
- [ovotech/jira](jira) - Interact with JIRA from a CI pipeline.
- [ovotech/github-actions](github-actions) - Trigger (and optionally wait for completion) GitHub Actions pipeline from CircleCI
- [ovotech/telemetry](telemetry) - Consumer Products Production Engineering team orb which enables sending Orb usage telemetry.

Other orbs in the ovotech namespace:

- [ovotech/shipit@1](https://github.com/ovotech/pe-orbs/tree/master/shipit) - Run shipit and record deployments to https://shipit.ovotech.org.uk.

## Releasing pendo/terraform-v2

The orb is published manually to the private `pendo` namespace with the CircleCI CLI. The `publish_orb` job in `.circleci/continue_config.yml` and `orb_version.txt` belong to the upstream ovotech flow and are not used.

Steps below use CircleCI CLI v1 (tested with `circleci 1.15.0`).

### Prerequisites

- CircleCI CLI v1 authenticated with a personal API token that can publish to the `pendo` namespace.
- Changes merged to `master`.

### Steps

Run from the repo root.

1. Update master:

   ```bash
   git checkout master
   git pull
   ```

2. Flatten the orb. `orb.yml` uses `include <file>` directives that `tools/include.py` expands. The script prints to stdout and does not create a file, so redirect the output:

   ```bash
   python3 tools/include.py terraform-v2/orb.yml > /tmp/terraform-v2_orb.yml
   ```

   A `SyntaxWarning: "\<" is an invalid escape sequence` on Python 3.12+ is expected and does not affect the output.

3. Validate:

   ```bash
   circleci orb validate /tmp/terraform-v2_orb.yml
   ```

4. Get the latest published version:

   ```bash
   circleci orb get pendo/terraform-v2
   ```

5. Diff against the latest version to confirm only the expected changes are included (replace `X.Y.Z` with the version from step 4):

   ```bash
   circleci orb source pendo/terraform-v2@X.Y.Z > /tmp/terraform-v2_current.yml
   diff /tmp/terraform-v2_current.yml /tmp/terraform-v2_orb.yml
   ```

6. Publish the next version. Published versions are immutable.

   ```bash
   circleci orb publish /tmp/terraform-v2_orb.yml pendo/terraform-v2@X.Y.Z
   ```

7. Confirm:

   ```bash
   circleci orb get pendo/terraform-v2
   ```

### Testing before release

To test a change from a consuming repo before releasing, publish a dev version. Dev versions can be overwritten and expire after 90 days.

```bash
circleci orb publish /tmp/terraform-v2_orb.yml pendo/terraform-v2@dev:<branch-name>
```

Reference it in the consuming repo as `pendo/terraform-v2@dev:<branch-name>`.

### After release

Bump the orb version in the consuming repos:

| Repo | File |
| --- | --- |
| pendo-infra | `.circleci/continue-config.yml` |
| pendo-infra-org | `.circleci/config.yml` |
| terraform-aws | `.circleci/config.yml` |
| terraform-azure | `.circleci/config.yml` |

### CLI v1 command changes

| Pre-v1 | v1 |
| --- | --- |
| `circleci orb info pendo/terraform-v2` | `circleci orb get pendo/terraform-v2` |
| `circleci orb publish increment <file> <orb> patch` | `circleci orb publish increment <file> <orb> --bump patch` |
| `circleci orb publish promote <orb>@dev:<label> patch` | `circleci orb publish promote <orb>@dev:<label> --bump patch` |
