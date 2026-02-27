# Alpacon CP Action

[![GitHub marketplace](https://img.shields.io/badge/marketplace-alpacon--cp--action-blue?logo=github)](https://github.com/marketplace/actions/alpacon-cp-action)

Copy files and directories between your CI/CD runner and remote servers in your [Alpacon](https://alpacon.io) workspace — no SSH keys, SCP, or rsync configuration needed.

- [Official documentation](https://docs.alpacax.com/reference/actions/cp/)

## Why use this action?

- **No SSH/SCP setup** — Transfer files using API tokens; no deploy keys or known_hosts to manage
- **Upload & download** — Move files in both directions between runner and server
- **Multiple files** — Upload several files in a single step without scripting loops
- **Recursive copy** — Transfer entire directory trees with one flag
- **User/group control** — Set file ownership on the remote server during transfer

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
        uses: alpacax/alpacon-cp-action@v1
        with:
          workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
          api-token: ${{ secrets.ALPACON_API_TOKEN }}
          source: './dist/'
          target-server: 'prod-server'
          target-path: '/var/www/app/'
          recursive: true
```

### Upload multiple files

```yaml
- name: Upload config files
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
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
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
    source: '/var/log/app/error.log'
    target-server: 'prod-server'
    target-path: './logs/'
    mode: download
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `workspace-url` | Alpacon workspace URL | Yes | |
| `api-token` | Alpacon API token for authentication | Yes | |
| `source` | Source path(s). Upload: local paths (supports multiple, one per line). Download: single remote path. | Yes | |
| `target-server` | Target server name | Yes | |
| `target-path` | Destination path (remote for upload, local for download) | Yes | |
| `mode` | `upload` (default) or `download` | No | `upload` |
| `recursive` | Set to `true` to copy directories recursively | No | `false` |
| `username` | Username for file ownership on the remote server | No | |
| `groupname` | Group name for file ownership (requires username) | No | |

## Important notes

- **Download `target-path`**: Must be a directory path (e.g., `./`, `./data/`). The file is saved with its original filename.
- **Upload `target-path`**: Can be either a specific file path or a directory path.
- **Multiple sources**: Only supported in upload mode (one path per line).
- **Download mode**: Supports only a single source path.

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `alpacon: command not found` | CLI not installed | Add `alpacax/alpacon-setup-action@v1` before this action |
| `login failed` | Invalid credentials | Verify `workspace-url` and `api-token` secrets are set correctly |
| `No such file or directory` | Wrong source or target path | Check paths are correct for the selected `mode` |
| `groupname requires username` | `groupname` set without `username` | Always set `username` when using `groupname` |

## Related actions

- [alpacon-setup-action](https://github.com/alpacax/alpacon-setup-action) — Install Alpacon CLI (required)
- [alpacon-websh-action](https://github.com/alpacax/alpacon-websh-action) — Execute shell commands on remote servers
- [alpacon-common-action](https://github.com/alpacax/alpacon-common-action) — Run any Alpacon CLI command

## Resources

- [GitHub Actions integration guide](https://docs.alpacax.com/integrate/github-actions/)
- [Alpacon CLI reference](https://docs.alpacax.com/reference/cli/)
- [alpacon cp command reference](https://docs.alpacax.com/reference/cli/cp/)

## Releasing

When creating a new release, always update the `v1` major version tag:

```bash
git tag -f v1 v1.x.0
git push origin v1 --force
```

This ensures users referencing `@v1` automatically get the latest release.
