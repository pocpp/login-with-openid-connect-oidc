# Azure OIDC Workload Identity Setup Tool

[日本語](./README_ja.md)

A simple tool to easily configure **Azure Deployment Workflow with Workload Identities** for any GitHub repository using OpenID Connect (OIDC) authentication.

## 🚀 What This Tool Does

This tool automates the complex process of setting up Azure AD application registration and federated credentials for GitHub Actions, enabling secure, passwordless authentication to Azure resources.

### Key Features

- ✅ **One-command setup** - Automates the entire Azure AD configuration process
- ✅ **Secure authentication** - Uses OpenID Connect (OIDC) instead of service principal secrets
- ✅ **Flexible scope** - Supports both subscription and resource group level permissions
- ✅ **Auto-detection** - Automatically detects GitHub repository information from git remote
- ✅ **Comprehensive validation** - Includes error handling and verification steps

## 📋 Prerequisites

Before using this tool, ensure you have:

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) installed and configured
- [jq](https://stedolan.github.io/jq/) installed for JSON processing
- Appropriate Azure permissions to create AD applications and assign roles
- A GitHub repository with remote origin configured

## 🛠️ Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/pocpp/login-with-openid-connect-oidc.git
```

### 2. Navigate to your target repository

Move to the GitHub repository where you want to set up Azure authentication:

```bash
cd /path/to/your/target/repository
```

### 3. Run the setup script

#### Basic usage (with auto-detected values):
```bash
/path/to/login-with-openid-connect-oidc/script/deploy
```

#### Advanced usage with custom parameters:
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  --resource-group my-resource-group \
  --display-name my-azure-app \
  --github-username myusername \
  --github-repo myrepository \
  --branch main
```

#### Allow access from all branches:
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  --all \
  --display-name my-azure-app \
  --github-username myusername \
  --github-repo myrepository
```

#### Short form options:
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  -g my-rg -d my-app -u myuser -r myrepo -b main
```

#### Short form with all branches:
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  -a -g my-rg -d my-app -u myuser -r myrepo
```

### 4. Configure GitHub Secrets

After running the script, add the following secrets to your GitHub repository:

1. Go to your GitHub repository → Settings → Secrets and variables → Actions
2. Add these repository secrets:
   - `AZURE_CLIENT_ID`: (provided by the script output)
   - `AZURE_TENANT_ID`: (provided by the script output)
   - `AZURE_SUBSCRIPTION_ID`: (provided by the script output)

### 5. Use in your GitHub Actions workflow

Copy the provided workflow file to your repository:

```bash
mkdir -p .github/workflows
cp /path/to/login-with-openid-connect-oidc/.github/workflows/workflow.yml .github/workflows/
```

Or create your own workflow using the Azure login action:

```yaml
name: Azure Deployment
on: [push]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Azure login
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Run Azure CLI commands
        uses: azure/cli@v2
        with:
          azcliversion: latest
          inlineScript: |
            az account show
            # Add your Azure CLI commands here
```

## 📖 Command Line Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--help` | `-h` | Show help message | - |
| `--display-name` | `-d` | Azure AD application display name | `$USER-timestamp` |
| `--github-username` | `-u` | GitHub username | Auto-detected from git remote |
| `--github-repo` | `-r` | GitHub repository name | Auto-detected from git remote |
| `--branch` | `-b` | Git branch name (ignored when `--all` is specified) | `main` |
| `--all` | `-a` | Allow access from all branches using wildcard pattern | Specific branch only |
| `--resource-group` | `-g` | Azure resource group for scoped permissions | Subscription scope (default) |

## 🔧 What the Script Does

1. **Azure Login**: Authenticates with Azure CLI
2. **Create Azure AD Application**: Creates a new application registration
3. **Configure Federated Credentials**: Sets up OIDC trust relationship with GitHub
4. **Create Service Principal**: Creates a service principal for the application
5. **Assign Permissions**: Assigns Contributor role (subscription or resource group scope)
6. **Generate Output**: Provides the required GitHub secrets

## 📁 Generated Files

The script creates the following files in your current directory:

- `app_creation_output.json`: Azure AD application details
- `policy.json`: Federated credential policy configuration

## 🔒 Security Benefits

Using OIDC Workload Identity provides several security advantages:

- **No long-lived secrets**: Eliminates the need to store Azure service principal passwords
- **Short-lived tokens**: Uses temporary tokens that expire automatically
- **Conditional access**: Tokens are only valid for specific GitHub repository (and specific branch or all branches based on configuration)
- **Audit trail**: All authentication events are logged in Azure AD

## 📚 References

- [Azure Login Action with OIDC](https://github.com/Azure/login#login-with-openid-connect-oidc-recommended)
- [Microsoft Learn: Authenticate Azure deployment workflow with workload identities](https://learn.microsoft.com/en-us/training/modules/authenticate-azure-deployment-workflow-workload-identities)
- [GitHub Actions: Configuring OpenID Connect in Azure](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
