# Claude Code S3バックアップアクション

[Claude Code](https://claude.ai/code)を実行し、ClaudeプロジェクトをS3またはS3互換ストレージ（MinIOなど）に自動的にバックアップするGitHub Actionです。

## 機能

- ✅ ワークフローでClaude Code Actionを実行
- 📦 Claudeプロジェクトを自動圧縮
- ☁️ S3またはS3互換ストレージにバックアップをアップロード
- 🔗 カスタムS3エンドポイント（MinIOなど）をサポート
- 📊 アップロード時にGitHubメタデータを含める

## 使用方法

このアクションをGitHubワークフローに追加してください：

```yaml
- name: Run Claude Code with S3 Backup
  uses: takumi3488/mycca@main
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    s3-bucket: 'your-backup-bucket'
    # オプション：MinIOや他のS3互換ストレージの場合
    s3-endpoint-url: 'https://minio.example.com'
    s3-force-path-style: 'true'
```

## 入力パラメータ

| 入力 | 説明 | 必須 | デフォルト |
|------|------|------|-----------|
| `claude-code-oauth-token` | Claude Code OAuthトークン | ✅ | - |
| `aws-access-key-id` | AWS Access Key ID | ✅ | - |
| `aws-secret-access-key` | AWS Secret Access Key | ✅ | - |
| `aws-region` | AWSリージョン（またはS3互換ストレージのリージョン） | ❌ | `us-east-1` |
| `s3-endpoint-url` | S3エンドポイントURL（MinIOや他のS3互換ストレージ用） | ❌ | - |
| `s3-force-path-style` | パススタイルアドレッシングを強制（MinIOに必要） | ❌ | `false` |
| `s3-bucket` | S3バケット名 | ✅ | - |
| `additional-permissions` | Claude Codeの追加権限 | ❌ | `actions: read` |

## 出力

| 出力 | 説明 |
|------|------|
| `s3-location` | アップロードされたバックアップのS3場所 |
| `archive-name` | 作成されたアーカイブファイル名 |

## セットアップ

### 1. 必要なシークレット

リポジトリに以下のシークレットを追加してください：

- `CLAUDE_CODE_OAUTH_TOKEN` - Claude Code OAuthトークン
- `AWS_ACCESS_KEY_ID` - AWS Access Key ID
- `AWS_SECRET_ACCESS_KEY` - AWS Secret Access Key

### 2. S3バケット

バックアップが保存されるS3バケット（またはMinIOバケット）を作成してください。

### 3. AWS権限

AWS認証情報に以下の権限があることを確認してください：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": "arn:aws:s3:::your-backup-bucket/*"
    }
  ]
}
```

## 使用例

### AWS S3での基本的な使用方法

```yaml
name: Claude Code Backup
on: [push]

jobs:
  claude-backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: takumi3488/mycca@main
        with:
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          s3-bucket: 'my-claude-backups'
```

### MinIOでの使用方法

```yaml
name: Claude Code MinIO Backup
on: [push]

jobs:
  claude-backup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: takumi3488/mycca@main
        with:
          claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          aws-access-key-id: ${{ secrets.MINIO_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.MINIO_SECRET_KEY }}
          aws-region: 'us-east-1'
          s3-endpoint-url: 'https://minio.example.com'
          s3-force-path-style: 'true'
          s3-bucket: 'claude-projects'
```

## 動作原理

1. **Claude Codeの実行**: アクションはまずClaude Code Actionを実行します
2. **プロジェクトの検索**: `$HOME/.claude/projects`でClaudeプロジェクトを検索します
3. **アーカイブの作成**: プロジェクトディレクトリをtar.gzファイルに圧縮します
4. **S3へのアップロード**: アーカイブをメタデータと共に指定されたS3バケットにアップロードします
5. **クリーンアップ**: 一時アーカイブファイルを削除します

## アーカイブの命名規則

アーカイブは以下のパターンで名前が付けられます：
```
claude_projects_{REPO_NAME}_{RUN_ID}_{TIMESTAMP}.tar.gz
```

例：`claude_projects_takumi3488_mycca_1234567890_20241210_143022.tar.gz`

## メタデータ

アップロードされた各アーカイブには、GitHubメタデータが含まれます：
- `github-repo`: リポジトリ名
- `github-run-id`: GitHub Actions実行ID
- `github-actor`: ワークフローをトリガーしたユーザー
- `github-event`: ワークフローをトリガーしたイベントタイプ

## ライセンス

このプロジェクトはオープンソースで、MITライセンスの下で利用可能です。

## 貢献

貢献を歓迎します！プルリクエストをお気軽に提出してください。