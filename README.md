⚠️ I vibe-coded this whole thing...

# hako-packages

> **This is just a testing package registry.** It exists to try out [hako](https://github.com/j-kop/hako-source) and show how a registry works. Packages may be added, changed, or removed at any time, and nothing here is guaranteed to be maintained. If you depend on hako, set up your own registry (see below).

A hako registry is a plain git repository. It contains **no binaries**, only one small TOML manifest per package. Each manifest says where that package's prebuilt release files are and what their sha256 checksums are. hako clones the registry with `git`, picks the right file for your platform, downloads it from the package's own release, checks the checksum, and links the executables into `~/.hako/bin`.

## Packages in this registry

| Package | What it is |
|---|---|
| `hako` | hako itself, so `hako upgrade` keeps hako up to date |

## How to: set up your own registry

1. **Create a git repository** with this layout:
   ```
   hako-registry.toml
   packages/
     mytool.toml
   ```
   Any git host works (GitHub, GitLab, self-hosted), and so does a local path. hako just runs `git clone`.

2. **Add `hako-registry.toml`** with exactly this content:
   ```toml
   schema = 1
   ```
   hako refuses registries without this file, or with a schema version it doesn't understand.

3. **Add one manifest per package** in `packages/<name>.toml`. The format is described below.

4. **Commit and push**, then point hako at it:
   ```sh
   hako registry add myregistry https://github.com/you/your-registry.git
   hako search
   hako install mytool
   ```
   You can use several registries at the same time. If two registries have a package with the same name, choose one with `hako install myregistry/mytool`.

**Private registries** work too. Use an SSH URL (`git@github.com:you/your-registry.git`), and hako clones it with your SSH key. The **release files** themselves must still be downloadable without signing in (see below).

## How to: make a release that works with hako

hako installs **prebuilt binaries**; it never builds from source. A package's release (usually a GitHub Release in the package's own repo) has to meet these requirements:

### 1. One file per platform
hako knows these four platform keys:

| Key | Platform |
|---|---|
| `x86_64-linux` | Linux, Intel/AMD 64-bit |
| `aarch64-linux` | Linux, ARM 64-bit |
| `x86_64-macos` | macOS, Intel |
| `aarch64-macos` | macOS, Apple Silicon |

You don't have to support all of them. On a platform with no build, hako shows a clear error. For Linux, **static musl builds** (`*-unknown-linux-musl`) are recommended, because they run on any distro.

### 2. A supported file format
hako detects the format from the URL's file extension:

| Extension | Handled as |
|---|---|
| `.tar.gz`, `.tgz` | gzip tarball |
| `.tar.xz`, `.txz` | xz tarball |
| `.tar` | plain tarball |
| `.zip` | zip archive |
| anything else | a single raw executable (the manifest must then list exactly one `bin`) |

### 3. Archive layout
Put the executable either at the top of the archive or inside **one** top-level folder. Both of these work for `bin = ["mytool"]`:
```
mytool-x86_64-linux.tar.gz          mytool-x86_64-linux.tar.gz
└── mytool                          └── mytool-1.0.0-x86_64-linux/
                                        ├── mytool
                                        ├── README.md
                                        └── LICENSE
```
- Executables in subfolders are fine: use `bin = ["bin/mytool"]`.
- Archives containing absolute paths or `..` are rejected.
- hako sets the executable permission itself, so it doesn't matter whether the archive preserved it.

### 4. Publicly downloadable files
hako downloads release files with a plain HTTPS request, without signing in. Redirects are fine, and GitHub Release download links use them. Files in a **private** repo's release **can't** be downloaded this way.

### 5. Never change published files
The checksum in the manifest locks the exact file. If a file is replaced under the same version, installs fail with `checksum mismatch`. To fix a broken build, publish a **new version** instead.

### Example: automating it
[hako-source's release workflow](https://github.com/j-kop/hako-source/blob/main/.github/workflows/release.yml) is a working example. On a `vX.Y.Z` tag, it builds all four platforms on GitHub Actions and packages each build as `hako-<key>.tar.gz` with a single top-level folder. It also publishes a ready-to-copy `hako.toml` manifest with the checksums filled in. Copy that workflow and swap in your own binary name.

## Manifest format

`packages/<name>.toml`:

```toml
name        = "mytool"                         # must match the file name; only a-z 0-9 - _
description = "What mytool does"               # shown by `hako search` and `hako info`
repo        = "https://github.com/you/mytool"  # the package's own repo
license     = "MIT"                            # optional
bin         = ["mytool"]                       # executables to put on PATH (paths inside the archive)

[versions."1.0.0".targets.x86_64-linux]
url    = "https://github.com/you/mytool/releases/download/v1.0.0/mytool-x86_64-linux.tar.gz"
sha256 = "<64 hex characters>"

[versions."1.0.0".targets.aarch64-macos]
url    = "https://github.com/you/mytool/releases/download/v1.0.0/mytool-aarch64-macos.tar.gz"
sha256 = "<64 hex characters>"
```

Rules:
- **Version keys** must be semver without a `v`: `"1.2.3"`, not `"v1.2.3"`. The git tag in the URL can still be `v1.2.3`.
- **Which version is installed:** `hako install mytool` picks the highest stable version. Pre-releases like `"2.0.0-beta.1"` are installed only on request (`hako install mytool@2.0.0-beta.1`), or when no stable version exists.
- **Unknown fields are rejected**, so typos don't go unnoticed.
- **URLs** must start with `https://`, `http://`, or `file://`. `file://` is for local testing only.

### Getting the checksums
```sh
curl -fsSL <url> | sha256sum        # Linux
curl -fsSL <url> | shasum -a 256    # macOS
```
If the project publishes its own checksums (for example a `.sha256` file next to each release file), compare them against what you computed.

### Adding a new version
Add another `[versions."X.Y.Z".targets.<key>]` section for each platform, and **keep the old ones**. That way `hako upgrade` moves everyone to the new version, and `hako install mytool@1.0.0` still works for anyone who needs the old one.

## How to: test a package before pushing

hako can use a local clone of the registry. Commit your change, then install into a throwaway folder so your real setup isn't touched:

```sh
git commit -am "Add mytool 1.0.0"
HAKO_HOME=/tmp/hako-test HAKO_REGISTRY=$PWD hako install mytool
/tmp/hako-test/bin/mytool --version
rm -rf /tmp/hako-test
```

`HAKO_REGISTRY` replaces the `default` registry's URL, and `HAKO_HOME` keeps everything inside `/tmp/hako-test`. hako reads the committed state, so uncommitted edits aren't picked up.

## License

[MIT](LICENSE)
