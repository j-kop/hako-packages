# hako-packages

The package registry for [hako](https://github.com/j-kop/hako-source). Each package is described by one manifest, `packages/<name>.toml`.

## Adding a package

1. In your package's own repo, publish a GitHub Release with prebuilt assets for each target you support:
   `x86_64-linux`, `aarch64-linux`, `x86_64-macos`, `aarch64-macos`.
   Supported formats: `.tar.gz`/`.tgz`, `.tar.xz`, `.tar`, `.zip`, or a raw binary.
2. Compute the checksum of each asset: `sha256sum hello-x86_64-linux.tar.gz`.
3. Add `packages/<name>.toml` (see `packages/ripgrep.toml`) and open a pull request.

Manifest rules:
- `name` must match the file name and contain only `[a-z0-9_-]`.
- Version keys must be valid semver (`1.2.3`, not `v1.2.3`).
- `bin` lists executable paths relative to the archive root. A single top-level wrapper directory such as `hello-1.0.0/` is stripped automatically.

## License

[MIT](LICENSE)
