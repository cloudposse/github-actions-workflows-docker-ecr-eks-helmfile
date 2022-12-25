<!-- markdownlint-disable -->



## Features branch (Pull request) workflow <br><br>Build, test Docker image, deploy it to EKS preview environment depends of PR labels  <br><br>### Usage <br><br>Create in your repo  \_\_`.github/workflows/feature.yaml`\_\_<br><br>```yaml<br>  name: Feature Branch<br>  on:<br>    pull\_request:<br>      branches: [ 'master' ]<br>      types: [opened, synchronize, reopened, closed, labeled, unlabeled]<br><br>  permissions:<br>    pull-requests: write<br>    deployments: write<br>    id-token: write<br>    contents: read<br><br>  jobs:<br>    do:<br>      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/feature-branch.yml@main<br>      with:<br>        organization: "${{ github.event.repository.owner.login }}"<br>        repository: "${{ github.event.repository.name }}"<br>        open: ${{ github.event.pull\_request.state == 'open' }}<br>        labels: ${{ toJSON(github.event.pull\_request.labels.\*.name) }}<br>        ref: ${{ github.event.pull\_request.head.ref  }}<br>      secrets:<br>        github-private-actions-pat: "${{ secrets.PUBLIC\_REPO\_ACCESS\_TOKEN }}"<br>        registry: "${{ secrets.ECR\_REGISTRY }}"<br>        secret-outputs-passphrase: "${{ secrets.GHA\_SECRET\_OUTPUT\_PASSPHRASE }}"<br>        ecr-region: "${{ secrets.ECR\_REGION }}"<br>        ecr-iam-role: "${{ secrets.ECR\_IAM\_ROLE }}"<br>```



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






## Main branch workflow<br><br>Build, test Docker image, deploy it to EKS dev environment and draft new release  <br><br>### Usage <br><br>Create in your repo  \_\_`.github/workflows/main.yaml`\_\_<br><br>```yaml<br>  name: Main Branch<br>  on:<br>    push:<br>      branches: [ master ]<br><br>  permissions:<br>    contents: write<br>    id-token: write<br><br>  jobs:<br>    do:<br>      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/main-branch.yml@main<br>      with:<br>        organization: "${{ github.event.repository.owner.login }}"<br>        repository: "${{ github.event.repository.name }}"<br>      secrets:<br>        github-private-actions-pat: "${{ secrets.PUBLIC\_REPO\_ACCESS\_TOKEN }}"<br>        registry: "${{ secrets.ECR\_REGISTRY }}"<br>        secret-outputs-passphrase: "${{ secrets.GHA\_SECRET\_OUTPUT\_PASSPHRASE }}"<br>        ecr-region: "${{ secrets.ECR\_REGION }}"<br>        ecr-iam-role: "${{ secrets.ECR\_IAM\_ROLE }}"<br>```



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






## Release workflow <br><br>Promote existing Docker image to release version, deploy it to EKS staging and then production environments.  <br><br>### Usage <br><br>Create in your repo  \_\_`.github/workflows/release.yaml`\_\_<br><br>```yaml<br>  name: Release<br>  on:<br>    release:<br>      types: [published]<br><br>  permissions:<br>    id-token: write<br>    contents: write<br><br>  jobs:<br>    perform:<br>      uses: cloudposse/github-actions-workflows-docker-ecr-eks-helmfile/.github/workflows/release.yml@main<br>      with:<br>        organization: "${{ github.event.repository.owner.login }}"<br>        repository: "${{ github.event.repository.name }}"<br>        version: ${{ github.event.release.tag\_name }}<br>      secrets:<br>        github-private-actions-pat: "${{ secrets.PUBLIC\_REPO\_ACCESS\_TOKEN }}"<br>        registry: "${{ secrets.ECR\_REGISTRY }}"<br>        secret-outputs-passphrase: "${{ secrets.GHA\_SECRET\_OUTPUT\_PASSPHRASE }}"<br>        ecr-region: "${{ secrets.ECR\_REGION }}"<br>        ecr-iam-role: "${{ secrets.ECR\_IAM\_ROLE }}"<br>```



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
