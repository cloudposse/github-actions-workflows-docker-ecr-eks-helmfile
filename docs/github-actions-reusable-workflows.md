<!-- markdownlint-disable -->
## Workflows

| Name | Description |
|------|-------------|
| [Features branch (Pull request) workflow ](#features-branch-pull-request-workflow) | Build, test Docker image, deploy it to EKS `preview` environment depends of PR labels   |
| [Hotfix branch (Pull request into release/\* branches) workflow ](#hotfix-branch-pull-request-into-release-branches-workflow) | Build, test Docker image, deploy it to EKS `hotfix` environment depends of PR labels   |
| [Hotfix workflow enable](#hotfix-workflow-enable) | For each new release create `release/{version}` branch that is key puzzle for `hotfix` workflow. |
| [Hotfix release workflow](#hotfix-release-workflow) | Build, test Docker image, deploy it to EKS production environment and reintegrate `hotfix` into the main branch   |
| [Main branch workflow](#main-branch-workflow) | Build, test Docker image, deploy it to EKS `dev` environment and draft new release   |
| [Release workflow ](#release-workflow) | Promote existing Docker image to release version, deploy it to EKS `staging` and then `production` environments.   |




## Features branch (Pull request) workflow 

Build, test Docker image, deploy it to EKS `preview` environment depends of PR labels  

### Usage 

Create in your repo  __`.github/workflows/feature.yaml`__

```yaml
  name: Feature Branch
  on:
    pull_request:
      branches: [ 'master' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]
  
  permissions:
    pull-requests: write
    deployments: write
    id-token: write
    contents: read
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/feature-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref  }}
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| labels | Pull Request labels | string | ${{ toJSON(github.event.pull\_request.labels.\*.name) }} | false |
| open | Pull Request open/close state. Set true if opened | boolean | ${{ github.event.pull\_request.state == 'open' }} | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | ${{ github.event.pull\_request.head.ref }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Hotfix branch (Pull request into release/* branches) workflow 

Build, test Docker image, deploy it to EKS `hotfix` environment depends of PR labels  

### Usage 

Create in your repo  __`.github/workflows/hotfix-branch.yaml`__

```yaml
  name: Hotfix Branch
  on:
    pull_request:
      branches: [ 'release/**' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]
  
  permissions:
    pull-requests: write
    deployments: write
    id-token: write
    contents: read
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        open: ${{ github.event.pull_request.state == 'open' }}
        labels: ${{ toJSON(github.event.pull_request.labels.*.name) }}
        ref: ${{ github.event.pull_request.head.ref  }}
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| labels | Pull Request labels | string | ${{ toJSON(github.event.pull\_request.labels.\*.name) }} | false |
| open | Pull Request open/close state. Set true if opened | boolean | ${{ github.event.pull\_request.state == 'open' }} | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | ${{ github.event.pull\_request.head.ref }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Hotfix workflow enable

For each new release create `release/{version}` branch that is key puzzle for `hotfix` workflow.

### Usage 

Create in your repo  __`.github/workflows/hotfix-enabled.yaml`__

```yaml
  name: Release branch
  on:
    release:
      types: [published]

  permissions:
    id-token: write
    contents: write

  jobs:
    hotfix:
      name: release / branch
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-mixin.yml@main
      with:
        version: ${{ github.event.release.tag_name }}
```

or add `hotfix` job to existing __`.github/workflows/release.yaml`__

```
  jobs:
    hotfix:
      name: release / branch
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-mixin.yml@main
      with:
        version: ${{ github.event.release.tag_name }}  
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| version | Release version tag | string | N/A | true |








## Hotfix release workflow

Build, test Docker image, deploy it to EKS production environment and reintegrate `hotfix` into the main branch  

### Usage 

Create in your repo  __`.github/workflows/hotfix-release.yaml`__

```yaml
  name: Hotfix Release
  on:
    push:
      branches: [ 'release/**' ]
  
  permissions:
    contents: write
    id-token: write
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/hotfix-release.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| default\_branch | Default branch for this repo | string | main | true |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Main branch workflow

Build, test Docker image, deploy it to EKS `dev` environment and draft new release  

### Usage 

Create in your repo  __`.github/workflows/main.yaml`__

```yaml
  name: Main Branch
  on:
    push:
      branches: [ master ]
  
  permissions:
    contents: write
    id-token: write
  
  jobs:
    do:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/main-branch.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Release workflow 

Promote existing Docker image to release version, deploy it to EKS `staging` and then `production` environments.  

### Usage 

Create in your repo  __`.github/workflows/release.yaml`__

```yaml
  name: Release
  on:
    release:
      types: [published]
  
  permissions:
    id-token: write
    contents: write
  
  jobs:
    perform:
      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/release.yml@main
      with:
        organization: "${{ github.event.repository.owner.login }}"
        repository: "${{ github.event.repository.name }}"
        version: ${{ github.event.release.tag_name }}
      secrets:
        github-private-actions-pat: "${{ secrets.PUBLIC_REPO_ACCESS_TOKEN }}"
        registry: "${{ secrets.ECR_REGISTRY }}"
        secret-outputs-passphrase: "${{ secrets.GHA_SECRET_OUTPUT_PASSPHRASE }}"
        ecr-region: "${{ secrets.ECR_REGION }}"
        ecr-iam-role: "${{ secrets.ECR_IAM_ROLE }}"
```



### Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| organization | Repository owner organization (ex. acme for repo acme/example) | string | ${{ github.event.repository.owner.login }} | false |
| repository | Repository name (ex. example for repo acme/example) | string | ${{ github.event.repository.name }} | false |
| version | Release version tag | string | ${{ github.event.release.tag\_name }} | false |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |





<!-- markdownlint-restore -->
