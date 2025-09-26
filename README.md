

# Alpacon CP Action

Copy files and directories between your local machine and a remote server in your AlpacaX workspace using the `alpacon cp` command in GitHub Actions.

**Official Docs:** [alpacon cp](https://docs.alpacax.com/alpacon/cli/alpacon_cp)

## Features

- Upload files or folders to a remote server
- Download files or folders from a remote server  
- Supports recursive copy, user/group options, and multiple files

## Important Notes

- **Download target-path**: Must be a directory path (e.g., `./`, `./data/`). The file will be saved with its original filename.
- **Upload target-path**: Can be either a specific file path or directory path.
- **Recursive operations**: Use `recursive: true` for directory operations.

## Usage


### Upload a file to a remote server
```yaml
- name: Upload File
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
    source: './data/file.txt'
    target-server: 'worker1'
    target-path: '/data/file.txt'
```

### Download a file from a remote server
```yaml
- name: Download File
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
    source: '/data/file.txt'
    target-server: 'worker1'
    target-path: './data/'  # Directory path - file will be saved as './data/file.txt'
    mode: download
```

### Upload a directory recursively
```yaml
- name: Upload Directory
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
    source: './data/'
    target-server: 'worker1'
    target-path: '/data/'
    recursive: true
```

### Download a directory recursively
```yaml
- name: Download Directory
  uses: alpacax/alpacon-cp-action@v1
  with:
    workspace-url: ${{ secrets.ALPACON_WORKSPACE_URL }}
    api-token: ${{ secrets.ALPACON_API_TOKEN }}
    source: '/data/'
    target-server: 'worker1'
    target-path: './data/'
    mode: download
    recursive: true
```

## Inputs

| Name           | Description                                                                 | Required |
|----------------|-----------------------------------------------------------------------------|----------|
| workspace-url  | AlpacaX workspace URL.                                                      | Yes      |
| api-token      | Alpacon API token for authentication.                                       | Yes      |
| source         | Path to upload (local) or download (remote).                                | Yes      |
| target-server  | Target server name.                                                         | Yes      |
| target-path    | Destination path (remote for upload, local for download).                   | Yes      |
| mode           | "upload" (default) or "download".                                         | No       |
| recursive      | Set to true to copy directories recursively.                                | No       |
| username       | Username for server authentication (optional).                              | No       |
| groupname      | Group name for server authentication (optional).                            | No       |

## Notes

- For upload: `source` is a local path, `target-path` is the remote destination
- For download: `source` is a remote path, `target-path` is the local destination
- To copy multiple files, provide an array to `source`
- See the [alpacon cp documentation](https://docs.alpacax.com/alpacon/cli/alpacon_cp) for advanced usage

