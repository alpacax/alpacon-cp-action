# Changelog

## v1.3.0

- Add multi-source file upload support (multiple paths, one per line)
- Guard against empty source paths
- Enforce single-source restriction in download mode

## v1.2.0

- Add `set -e` error handling for login and execution steps
- Quote login inputs to prevent word splitting

## v1.1.0

- Add `username` and `groupname` inputs for file ownership control
- Add permissions configuration to test workflow

## v1.0.0

- Initial release
- Upload and download files via `alpacon cp`
- Recursive directory copy support
