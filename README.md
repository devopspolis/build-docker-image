<div style="display: flex; align-items: center;">
  <img src="logo.png" alt="Logo" width="50" height="50" style="margin-right: 10px;"/>
  <span style="font-size: 2.2em;">Build and publish Docker image to AWS ECR</span>
</div>

![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Build%20and%20publish%20Docker%20image%20to%20AWS%20ECR-blue?logo=github)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

<p>
This GitHub Action builds a Docker image, optionally using multiple platforms, and publishes it to Amazon Elastic Container Registry (ECR). It supports custom tags, build contexts, build arguments, and optional image signing using cosign.
</p>

See more [GitHub Actions by DevOpspolis](https://github.com/marketplace?query=devopspolis&type=actions)

---

## 📚 Table of Contents
- [📥 Features](#features)
- [📥 Inputs](#inputs)
- [📤 Outputs](#outputs)
- [📦 Usage](#usage)
- [🚦 Requirements](#requirements)
- [🧑‍⚖️ Legal](#legal)

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="features"></a>
## ✨ Features
- Multi-platform Docker builds with Buildx
- Supports custom Dockerfile path and build context
- Automatically tags and pushes multiple image tags
- Optional .npmrc injection from AWS Secrets Manager
- Optional Node.js setup via .nvmrc or package.json
---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="inputs"></a>
## 📥 Inputs

| Name                | Description                                                             | Required | Default         |
| ------------------- | ----------------------------------------------------------------------- | -------- | --------------- |
| `image_name`        | The name of the image to build (ECR repository name)                    | true     | —               |
| `tags`              | Comma-separated list of tags (e.g. `v1.2.0,prod,latest`)                | false    | `latest`        |
| `ref`               | Git branch, tag, or SHA to checkout                                     | false    | default branch  |
| `dockerfile`        | Path to Dockerfile                                                      | false    | `Dockerfile`    |
| `build_context`     | Docker build context                                                    | false    | `.`             |
| `build_args`        | Docker build arguments (comma-separated `--build-arg` options)          | false    | —               |
| `working-directory` | Build working directory                                                 | false    | `.`             |
| `npmrc_secret`      | AWS Secrets Manager secret name containing `.npmrc` content             | false    | —               |
| `platforms`         | Docker platforms for multi-arch builds (e.g. `linux/amd64,linux/arm64`) | false    | —               |
| `role`              | AWS role to assume                                                      | false    | —               |

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="outputs"></a>
## 📤 Outputs

| Name    | Description            |
| ------- | ---------------------- |
| `image` | The full ECR image URI |

---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="usage"></a>
## 📦 Usage

Example 1 - Extract and deploy artifact contents.

```yaml
name: Build and Publish Image

on:
  push:
    branches: [main]

jobs:
  build-and-publish-image:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      packages: read
    steps:
      - name: Build and publish Docker image
        uses: devopspolis/build-docker-image@main
        with:
          image_name: my-app
          tags: v1.2.0,latest
          nprmc_secret: app/my-app/.npmrc
          platforms: linux/amd64,linux/arm64
```

🔐 Notes

- The action automatically logs into Amazon ECR using aws-actions/amazon-ecr-login
- If npmrc_secret is provided, it downloads the secret from AWS Secrets Manager and saves it as ~/.npmrc for private package installs
- If .nvmrc or Node.js version is defined in package.json, it sets up Node.js automatically using actions/setup-node
---
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="requirements"></a>
## 🚦Requirements

The calling workflow must have the permissions shown below.
1. Permission to pull base images (e.g. from Docker Hub). The calling workflow should either authenticate prior to calling this action, or provide a an AWS `role` to assume
1. AWS Access Configuration
The calling workflow must authenticate to AWS with permission to push Docker images to Amazon ECR. The recommended method is to configure OIDC authentication between your GitHub repository and the AWS account, allowing the workflow to assume a role with the required permissions.

    The IAM role assumed by GitHub Actions should have permissions to
    - Pull base images (e.g. from Docker Hub)
    - Authenticate to Amazon ECR, and upload images
    - Read AWS Secrets Manager npmrc_secret (if using the npmrc_secret input to download a .npmrc file)

   In the example below the `AWS_ACCOUNT_ID` and `AWS_REGION` are retrieved from the GitHub repository environment variables, enabling the workflow to target environment specific AWS accounts.

```yaml
permissions:
  id-token: write       # Required for OIDC authentication to AWS
  contents: read        # Required to checkout code
  packages: read        # Required to download private GitHub Packages (e.g., via .npmrc)

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Set up AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/build-docker-image-role
          aws-region: ${{ vars.AWS_REGION }}
```
<!-- trunk-ignore(markdownlint/MD033) -->
<a id="legal"></a>
## 🧑‍⚖️ Legal
The MIT License (MIT)
