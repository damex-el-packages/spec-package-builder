# spec package builder

## Description

Reusable GitHub workflows for building, signing and publishing RPM packages from spec files.

Package repositories call `pipeline.yml`.
Pipeline lints spec files.
Pipeline builds packages for `Red Hat Enterprise Linux 9` and `Red Hat Enterprise Linux 10` on `x86_64` and `aarch64`.
Pipeline signs packages with repository own gpg key.
Pipeline publishes signed yum repository to `https://yum-repositories.damex.org/<repository>`.

GitHub repository name is yum repository name.

[Follow here if you want to build packages](#usage).

[Follow here if you want to use prebuilt packages](#using-prebuilt-packages).

## Usage

### Package repository layout

```
<repository>
+-- .github/workflows/pipeline.yml
+-- SPECS/<package>.spec
+-- SPECS/el9/<package>.spec -> ../<package>.spec
+-- SPECS/el10/<package>.spec -> ../<package>.spec
+-- <repository>-<expiry>.asc
+-- <repository>-<expiry>.gpg
```

Symlink in `SPECS/el9` or `SPECS/el10` enables package build for that distribution.

Spec file without symlink is not built and can be pulled into other spec files with `%include`.

### Calling pipeline

```yaml
---
name: pipeline
on:  # yamllint disable-line rule:truthy
  push:
    branches:
      - "**"
  schedule:
    - cron: 0 0 * * 0

jobs:
  pipeline:
    uses: damex-el-packages/spec-package-builder/.github/workflows/pipeline.yml@production
    with:
      gpg_key_id: 33261F4E1EF30BC3
      gpg_public_key: incus-2036-09-25.asc
      gpg_public_keyring: incus-2036-09-25.gpg
      publish: ${{ github.ref == 'refs/heads/production' }}
    secrets: inherit
```

Push to any branch lints and builds.
Push to `production` also signs and publishes.
Weekly schedule runs on default branch, publishes when default branch is `production`.

### Inputs

| Input                           | Required | Description                                                          |
|---------------------------------|----------|----------------------------------------------------------------------|
| gpg_key_id                      | yes      | Signing key id, must match `GPG_PRIVATE_KEY`                         |
| gpg_public_key                  | yes      | Armored public key file in repository root                           |
| gpg_public_keyring              | yes      | Binary public key file in repository root                            |
| build_with_published_repository | no       | Resolve build dependencies from own published yum repository. Repository must be published before first such build. |
| publish                         | no       | Sign packages and publish yum repository                             |

### Secrets

| Secret                       | Scope        | Description                              |
|------------------------------|--------------|------------------------------------------|
| GPG_PRIVATE_KEY              | repository   | Base64 of exported gpg secret key        |
| GPG_PASSPHRASE               | repository   | Passphrase of gpg secret key             |
| YUM_REPOSITORIES_ACCESS_KEY  | organization | S3 access key                            |
| YUM_REPOSITORIES_SECRET_KEY  | organization | S3 secret key                            |
| YUM_REPOSITORIES_S3_ENDPOINT | organization | S3 endpoint host without scheme          |
| YUM_REPOSITORIES_BUCKET_NAME | organization | S3 bucket name                           |

## Using prebuilt packages

Each repository describes how to add its yum repository and lists its packages.

| Repository       | Source                                                                          |
|------------------|---------------------------------------------------------------------------------|
| damex-incus      | [damex-el-packages/incus](https://github.com/damex-el-packages/incus)           |
| damex-kubernetes | [damex-el-packages/kubernetes](https://github.com/damex-el-packages/kubernetes) |
| damex-prometheus | [damex-el-packages/prometheus](https://github.com/damex-el-packages/prometheus) |
| damex-zfs        | [damex-el-packages/zfs](https://github.com/damex-el-packages/zfs)               |
