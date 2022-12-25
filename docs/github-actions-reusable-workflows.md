<!-- markdownlint-disable -->



## Features branch (Pull request) workflow 

Build, test Docker image, deploy it to EKS preview environment depends of PR labels  

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
| labels | Pull Request labels | string | [] | false |
| open | Pull Request open/close state. Set true if opened | boolean | true | false |
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| ref | The fully-formed ref of the branch or tag that triggered the workflow run | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Main branch workflow

Build, test Docker image, deploy it to EKS dev environment and draft new release  

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
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |






## Release workflow 

Promote existing Docker image to release version, deploy it to EKS staging and then production environments.  

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
| organization | Repository owner organization (ex. acme for repo acme/example) | string | N/A | true |
| repository | Repository name (ex. example for repo acme/example) | string | N/A | true |
| version | Release version tag | string | N/A | true |



### Secrets

| Name | Description | Required |
|------|-------------|----------|
| ecr-iam-role | IAM Role ARN provide ECR write/read access | true |
| ecr-region | ECR AWS region | true |
| github-private-actions-pat | Github PAT allow to pull private repos | true |
| registry | ECR ARN | true |
| secret-outputs-passphrase | Passphrase to encrypt/decrypt secret outputs with gpg. For more information [read](https://github.com/cloudposse/github-action-secret-outputs) | true |





<!-- markdownlint-restore -->
