# Authenticate Azure Deployment Workflow with workload-identities

https://github.com/Azure/login#login-with-openid-connect-oidc-recommended
https://learn.microsoft.com/ja-jp/training/modules/authenticate-azure-deployment-workflow-workload-identities

## Usage

1. deploy

```bash
$ script/deploy <display-name> <github-username> <github-repo> <branch-name>

# if you want set to default
$ script/deploy
# display-name = $USER-$(date +%Y%m%d%H%M%S)
# github-username = $(basename "$(dirname "$git_path")")
# github-repo = $(basename "$git_path" .git)
# branch-name = main
```

2. Set AZURE_CLIENT_ID AZURE_TENANT_ID AZURE_SUBSCRIPTION_ID in GitHub Action

3. git push to main branch