# Azure OIDC Workload Identity セットアップツール

任意のGitHubリポジトリに対して、OpenID Connect (OIDC) 認証を使用した **Azure Deployment Workflow with Workload Identities** を簡単に設定するためのツールです。

## 🚀 このツールの機能

このツールは、GitHub ActionsでAzure ADアプリケーション登録とフェデレーテッド資格情報を設定する複雑なプロセスを自動化し、Azureリソースへの安全でパスワードレスな認証を可能にします。

### 主な特徴

- ✅ **ワンコマンドセットアップ** - Azure ADの設定プロセス全体を自動化
- ✅ **セキュアな認証** - サービスプリンシパルのシークレットではなくOpenID Connect (OIDC) を使用
- ✅ **柔軟なスコープ** - サブスクリプションレベルとリソースグループレベルの権限をサポート
- ✅ **自動検出** - gitリモートからGitHubリポジトリ情報を自動検出
- ✅ **包括的な検証** - エラーハンドリングと検証ステップを含む

## 📋 前提条件

このツールを使用する前に、以下を確認してください：

- [Azure CLI](https://docs.microsoft.com/ja-jp/cli/azure/install-azure-cli) がインストールされ、設定されている
- JSON処理のための [jq](https://stedolan.github.io/jq/) がインストールされている
- ADアプリケーションの作成とロールの割り当てに適切なAzure権限がある
- リモートオリジンが設定されたGitHubリポジトリがある

## 🛠️ クイックスタート

### 1. このリポジトリをクローン

```bash
git clone https://github.com/pocpp/login-with-openid-connect-oidc.git
```

### 2. 対象リポジトリに移動

Azure認証を設定したいGitHubリポジトリに移動します：

```bash
cd /path/to/your/target/repository
```

### 3. セットアップスクリプトを実行

#### 基本的な使用方法（自動検出値を使用）：
```bash
/path/to/login-with-openid-connect-oidc/script/deploy
```

#### カスタムパラメータを使用した高度な使用方法：
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  --resource-group my-resource-group \
  --display-name my-azure-app \
  --github-username myusername \
  --github-repo myrepository \
  --branch main
```

#### 全ブランチからのアクセスを許可：
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  --all \
  --display-name my-azure-app \
  --github-username myusername \
  --github-repo myrepository
```

#### 短縮形オプション：
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  -g my-rg -d my-app -u myuser -r myrepo -b main
```

#### 全ブランチ対応の短縮形：
```bash
/path/to/login-with-openid-connect-oidc/script/deploy \
  -a -g my-rg -d my-app -u myuser -r myrepo
```

### 4. GitHubシークレットの設定

スクリプト実行後、以下のシークレットをGitHubリポジトリに追加してください：

1. GitHubリポジトリ → Settings → Secrets and variables → Actions に移動
2. 以下のリポジトリシークレットを追加：
   - `AZURE_CLIENT_ID`: （スクリプト出力で提供）
   - `AZURE_TENANT_ID`: （スクリプト出力で提供）
   - `AZURE_SUBSCRIPTION_ID`: （スクリプト出力で提供）

### 5. GitHub Actionsワークフローで使用

提供されたワークフローファイルをリポジトリにコピー：

```bash
mkdir -p .github/workflows
cp /path/to/login-with-openid-connect-oidc/.github/workflows/workflow.yml .github/workflows/
```

または、Azureログインアクションを使用して独自のワークフローを作成：

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

      - name: Azure CLIコマンドの実行
        uses: azure/cli@v2
        with:
          azcliversion: latest
          inlineScript: |
            az account show
            # ここにAzure CLIコマンドを追加
```

## 📖 コマンドラインオプション

| オプション | 短縮形 | 説明 | デフォルト |
|-----------|--------|------|-----------|
| `--help` | `-h` | ヘルプメッセージを表示 | - |
| `--display-name` | `-d` | Azure ADアプリケーション表示名 | `$USER-timestamp` |
| `--github-username` | `-u` | GitHubユーザー名 | gitリモートから自動検出 |
| `--github-repo` | `-r` | GitHubリポジトリ名 | gitリモートから自動検出 |
| `--branch` | `-b` | Gitブランチ名（`--all`指定時は無視される） | `main` |
| `--all` | `-a` | ワイルドカードパターンを使用して全ブランチからのアクセスを許可 | 特定ブランチのみ |
| `--resource-group` | `-g` | スコープ権限用のAzureリソースグループ | サブスクリプションスコープ（デフォルト） |

## 🔧 スクリプトの動作

1. **Azureログイン**: Azure CLIで認証
2. **Azure ADアプリケーション作成**: 新しいアプリケーション登録を作成
3. **フェデレーテッド資格情報の設定**: GitHubとのOIDC信頼関係を設定
4. **サービスプリンシパル作成**: アプリケーション用のサービスプリンシパルを作成
5. **権限の割り当て**: 共同作成者ロールを割り当て（サブスクリプションまたはリソースグループスコープ）
6. **出力生成**: 必要なGitHubシークレットを提供

## 📁 生成されるファイル

スクリプトは現在のディレクトリに以下のファイルを作成します：

- `app_creation_output.json`: Azure ADアプリケーションの詳細
- `policy.json`: フェデレーテッド資格情報ポリシー設定

## 🔒 セキュリティの利点

OIDC Workload Identityの使用により、以下のセキュリティ上の利点があります：

- **長期間有効なシークレットなし**: Azureサービスプリンシパルのパスワードを保存する必要がない
- **短期間有効なトークン**: 自動的に期限切れになる一時的なトークンを使用
- **条件付きアクセス**: 特定のGitHubリポジトリ（および設定に基づく特定ブランチまたは全ブランチ）でのみ有効なトークン
- **監査証跡**: すべての認証イベントがAzure ADにログ記録される

## 📚 参考資料

- [Azure Login Action with OIDC](https://github.com/Azure/login#login-with-openid-connect-oidc-recommended)
- [Microsoft Learn: ワークロード ID を使用して Azure デプロイ ワークフローを認証する](https://learn.microsoft.com/ja-jp/training/modules/authenticate-azure-deployment-workflow-workload-identities)
- [GitHub Actions: AzureでのOpenID Connectの設定](https://docs.github.com/ja/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)

## 🤝 コントリビューション

コントリビューションを歓迎します！お気軽にPull Requestを送信してください。

## 📄 ライセンス

このプロジェクトはMITライセンスの下でライセンスされています - 詳細は [LICENSE](LICENSE) ファイルを参照してください。
