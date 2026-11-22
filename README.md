# Alpacon CP Action

[![GitHub marketplace](https://img.shields.io/badge/marketplace-alpacon--cp--action-blue?logo=github)](https://github.com/marketplace/actions/alpacon-cp-action)

Copy files between GitHub Actions and your servers without handing your workflow an SSH key.

An SSH key in a CI secret is usually a full shell on your server, not just a way to copy files. With this action, the runner signs in to [Alpacon](https://alpacon.io) with a service token that can transfer files only to and from the paths you list, on the servers you list. Alpacon is a server access platform: its agent on each server connects out to Alpacon, so no inbound port has to be open for the runner.

- [Official documentation](https://docs.alpacax.com/reference/actions/cp/)

> Upgrading from `@v1`: change `alpacax/alpacon-cp-action@v1` to `alpacax/alpacon-cp-action@v2` and rename the `api-token` input to `service-token`. v2 accepts only a service token.

## How access is controlled

- **Only the paths you list**—The token's access rules name the servers and file paths it may use. Anything else is refused. Keep paths narrow: a `*` in a path rule spans subdirectories, and `*` alone allows every path.
- **No interactive access**—A service token cannot open a terminal session, a tunnel, or the file editor, and it never runs as `root`. Files are written as the server account its application owns.
- **No inbound port**—The Alpacon agent on each server connects out. Nothing on the server listens for the runner, so the runner's changing IP addresses don't matter.
- **Approved once, then unattended**—A token that can transfer files waits for approval in the Alpacon web console and authenticates nowhere until it is approved. Runs after that don't wait for anyone.
- **Bounded**—Every token has an expiry date and a rate limit, and you can disable it at any time. You can also limit it to hours and days of the week, and, on runners with fixed addresses such as self-hosted runners, to IP ranges.
- **On the record**—Every file transfer is recorded in the workspace audit log (Audit → Events → File transfers).

## Authentication

Set `service-token` to an Alpacon service token (its key starts with `alpst-`). A service token belongs to an IAM Application (a machine identity), not to a person, so it keeps working when people leave. A service token does not need a Work Session; endpoint permissions still apply.

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
| `username` | Server account for file ownership. A service token transfers files only as its application's account, never as `root` | No | |

## Important notes

- **Download `target-path`**: Must be a directory path (e.g., `./`, `./data/`). The file is saved with its original filename.
- **Upload `target-path`**: Can be either a specific file path or a directory path.
- **Multiple sources**: Only supported in upload mode (one path per line).
- **Download mode**: Supports only a single source path.

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `alpacon: command not found` | CLI not installed | Add `alpacax/alpacon-setup-action@v1` before this action |
| `Login failed` | Wrong workspace URL or token; the token is expired, disabled or still waiting for approval; or the runner is outside the token's allowed IP ranges or hours | Check both secrets, and the token's status and restrictions in the web console |
| `Permission denied uploading to …` (or downloading) | The server, the path, or the direction isn't in the token's access rules | Add a narrow rule for the server and path the workflow uses |
| `No such file or directory` | Wrong source or target path | Check paths are correct for the selected `mode` |
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
