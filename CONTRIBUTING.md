# Contributing to hako-packages

Note: this is a **testing registry**; see the notice in the [README](README.md). Pull requests are welcome, but packages may be removed at any time.

## Adding or updating a package

1. Make sure the package's release meets the requirements in [How to: make a release that works with hako](README.md#how-to-make-a-release-that-works-with-hako).
2. Add or edit `packages/<name>.toml`, following [Manifest format](README.md#manifest-format). When adding a version, keep the older versions in the file.
3. Test it locally, as described in [How to: test a package before pushing](README.md#how-to-test-a-package-before-pushing).
4. Open a pull request against `main`. Direct pushes to `main` are blocked.

Only add packages from their **official** releases, never re-uploaded copies. Checksums must come from the actual files, and preferably match checksums the project publishes itself.

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE).
