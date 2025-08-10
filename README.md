# Claude Code with S3 Backup Action

A GitHub Action that runs [Claude Code](https://claude.ai/code) and automatically backs up your Claude projects to S3 or S3-compatible storage (like MinIO).

## Features

- ✅ Run Claude Code Action in your workflows
- 📦 Automatically compress Claude projects
- ☁️ Upload backups to S3 or S3-compatible storage
- 🔗 Support for custom S3 endpoints (MinIO, etc.)
- 📊 Include GitHub metadata with uploads

## Usage

Add this action to your GitHub workflow:

```yaml
- name: Run Claude Code with S3 Backup
  uses: takumi3488/mycca@main
  with:
    claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    s3-bucket: 'your-backup-bucket'
    # Optional: for MinIO or other S3-compatible storage
    s3-endpoint-url: 'https://minio.example.com'
    s3-force-path-style: 'true'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `claude-code-oauth-token` | Claude Code OAuth token | ✅ | - |
| `aws-access-key-id` | AWS Access Key ID | ✅ | - |
| `aws-secret-access-key` | AWS Secret Access Key | ✅ | - |
| `aws-region` | AWS Region (or region for S3-compatible storage) | ❌ | `us-east-1` |
| `s3-endpoint-url` | S3 endpoint URL (for MinIO or other S3-compatible storage) | ❌ | - |
| `s3-force-path-style` | Force path-style addressing (required for MinIO) | ❌ | `false` |
| `s3-bucket` | S3 Bucket name | ✅ | - |
| `additional-permissions` | Additional permissions for Claude Code | ❌ | `actions: read` |

## Outputs

| Output | Description |
|--------|-------------|
| `s3-location` | S3 location of the uploaded backup |
| `archive-name` | Name of the created archive file |

## Setup

### 1. Required Secrets

Add these secrets to your repository:

- `CLAUDE_CODE_OAUTH_TOKEN` - Your Claude Code OAuth token
- `AWS_ACCESS_KEY_ID` - AWS Access Key ID
- `AWS_SECRET_ACCESS_KEY` - AWS Secret Access Key

### 2. S3 Bucket

Create an S3 bucket (or MinIO bucket) where backups will be stored.

### 3. AWS Permissions

Ensure your AWS credentials have the following permissions:

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

## Examples

### Basic Usage with AWS S3

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

### Usage with MinIO

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

## How It Works

1. **Run Claude Code**: The action first runs the Claude Code Action
2. **Find Projects**: Looks for Claude projects in `$HOME/.claude/projects`
3. **Create Archive**: Compresses the projects directory into a tar.gz file
4. **Upload to S3**: Uploads the archive to your specified S3 bucket with metadata
5. **Cleanup**: Removes the temporary archive file

## Archive Naming

Archives are named using the pattern:
```
claude_projects_{REPO_NAME}_{RUN_ID}_{TIMESTAMP}.tar.gz
```

Example: `claude_projects_takumi3488_mycca_1234567890_20241210_143022.tar.gz`

## Metadata

Each uploaded archive includes GitHub metadata:
- `github-repo`: Repository name
- `github-run-id`: GitHub Actions run ID
- `github-actor`: User who triggered the workflow
- `github-event`: Event type that triggered the workflow

## License

This project is open source and available under the MIT License.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.