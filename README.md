# Alpacon CP Action

[![GitHub marketplace](https://img.shields.io/badge/marketplace-alpacon--cp--action-blue?logo=github)](https://github.com/marketplace/actions/alpacon-cp-action)

Copy files and directories between your CI/CD runner and remote servers in your [Alpacon](https://alpacon.io) workspace—no SSH keys, SCP, or rsync configuration needed.

- [Official documentation](https://docs.alpacax.com/reference/actions/cp/)

## Why use this action?

- **No SSH/SCP setup**—Transfer files using a service token; no deploy keys or known_hosts to manage
- **Upload & download**—Move files in both directions between runner and server
- **Multiple files**—Upload several files in a single step without scripting loops
- **Recursive copy**—Transfer entire directory trees with one flag
- **User/group control**—Set file ownership on the remote server during transfer

## How access is controlled

This action runs on the service-token path, which is separate from the work sessions that people and AI agents open:

- **Reviewed before it works**—A service token that can run commands or transfer files is approved in the Alpacon console before it authenticates anywhere.
- **Deny by default**—Its ACL names the servers, commands and file paths the pipeline may use. Everything else is refused.
- **No one waits**—Within that ACL the workflow runs without a work session or an approval step.
- **Bounded**—Optional allowed IP ranges and hours, and an expiry set by the token's rotation policy.
- **On the record**—Every command and file transfer is recorded under the token in the workspace audit log.

## Authentication

Set `service-token` to an Alpacon service token (its key starts with `alpst-`). A service token belongs to an IAM Application (a machine identity), not to a person, so it keeps working when people leave. Its access is limited by the token's scopes and its server, command, and file access rules. A service token does not need a Work Session; endpoint permissions still apply.

Store the token as a repository secret named `ALPACON_SERVICE_TOKEN`, as in the examples below. Logging in with a service token requires Alpacon CLI v1.7.4 or later.

See [Service tokens](https://docs.alpacax.com/use/iam/applications/service-tokens/) and the [GitHub Actions integration guide](https://docs.alpacax.com/integrate/github-actions/).

## Usage

### Upload build artifacts

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: npm run build

      - name: Setup Alpacon CLI
        uses: alpacax/alpacon-setup-action@v1

      - name: Upload build to server
        uses: alpacax/alpacon-cp-action@v2
        with:
          workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
          service-token: ${{ secrets.ALPACON_SERVICE_TOKEN }}
          source: './dist/'
          target-server: 'prod-server'
          target-path: '/var/www/app/'
          recursive: true
```

### Upload multiple files

```yaml
- name: Upload config files
  uses: alpacax/alpacon-cp-action@v2
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    service-token: ${{ secrets.ALPACON_SERVICE_TOKEN }}
    source: |
      ./docker-compose.yml
      ./nginx.conf
      ./.env.production
    target-server: 'prod-server'
    target-path: '/opt/myapp/'
    username: ubuntu
```

### Download files from a server

```yaml
- name: Download error logs
  uses: alpacax/alpacon-cp-action@v2
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    service-token: ${{ secrets.ALPACON_SERVICE_TOKEN }}
    source: '/var/log/app/error.log'
    target-server: 'prod-server'
    target-path: './logs/'
    mode: download
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `workspace-url` | Alpacon workspace URL | Yes | |
| `service-token` | Alpacon service token key (`alpst-`) | Yes | |
| `source` | Source path(s). Upload: local paths (supports multiple, one per line). Download: single remote path. | Yes | |
| `target-server` | Target server name | Yes | |
| `target-path` | Destination path (remote for upload, local for download) | Yes | |
| `mode` | `upload` (default) or `download` | No | `upload` |
| `recursive` | Set to `true` to copy directories recursively | No | `false` |
| `username` | Username for file ownership on the remote server | No | |
| `groupname` | Group name for file ownership (requires username) | No | |

`groupname` cannot be used with a service token: files are transferred with the service account's private group, and a `groupname` value is ignored.

## Important notes

- **Download `target-path`**: Must be a directory path (e.g., `./`, `./data/`). The file is saved with its original filename.
- **Upload `target-path`**: Can be either a specific file path or a directory path.
- **Multiple sources**: Only supported in upload mode (one path per line).
- **Download mode**: Supports only a single source path.

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `alpacon: command not found` | CLI not installed | Add `alpacax/alpacon-setup-action@v1` before this action |
| `login failed` | Invalid credentials | Verify `workspace-url` and `service-token` secrets are set correctly |
| `No such file or directory` | Wrong source or target path | Check paths are correct for the selected `mode` |
| `groupname requires username` | `groupname` set without `username` | Always set `username` when using `groupname` |

## Related actions

- [alpacon-setup-action](https://github.com/alpacax/alpacon-setup-action)—Install Alpacon CLI (required)
- [alpacon-exec-action](https://github.com/alpacax/alpacon-exec-action)—Execute shell commands on remote servers
- [alpacon-common-action](https://github.com/alpacax/alpacon-common-action)—Run any Alpacon CLI command

## Resources

- [GitHub Actions integration guide](https://docs.alpacax.com/integrate/github-actions/)
- [Alpacon CLI reference](https://docs.alpacax.com/reference/cli/)
- [alpacon cp command reference](https://docs.alpacax.com/reference/cli/cp/)

## Releasing

When creating a new release, always update the `v2` major version tag:

```bash
git tag -f v2 v2.x.0
git push origin v2 --force
```

This ensures users referencing `@v2` automatically get the latest release.
