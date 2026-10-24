# Unofficial TraceDraw packages

Reproducible local packaging for [TraceDraw](https://github.com/igtoth/tracedraw),
currently targeting upstream version **0.15.0**.

This is an unofficial downstream packaging project. It downloads the exact
upstream release archives, verifies their SHA-256 digests, and repackages the
binaries; it does not contain TraceDraw source code. Upstream TraceDraw is
licensed under **MIT OR Apache-2.0**.

## Layout

- `flatpak/` — Flatpak manifest and exported desktop metadata
- `snap/` — strict-confinement `core24` Snapcraft project
- `releases/` — locally built distributable artifacts; ignored by Git except
  for `SHA256SUMS`

## Published local artifacts

| Artifact | Architecture | SHA-256 |
| --- | --- | --- |
| `tracedraw-0.15.0-x86_64-workdirs.flatpak` | x86_64 | `ae8f008c370b710fc5de5e3d542e7c550b652b6a811d8428a3c07355ebed7379` |
| `tracedraw_0.15.0_amd64.snap` | amd64 | `d78f092988a790ed892c9b1356ea0cba55c050a3d74e05365ad722a0f19ae91d` |

Verify locally with:

```sh
cd releases
sha256sum -c SHA256SUMS
```

## Flatpak

The Flatpak application ID is `io.github.igtoth.tracedraw`. It grants direct
access only to Desktop, Documents, and Downloads, because TraceDraw's own
Save As browser directly enumerates folders rather than using a document
portal. This is deliberately narrower than access to the entire home directory.

### Build and install

```sh
cd flatpak
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install --user flathub org.freedesktop.Sdk//26.08 org.freedesktop.Platform//26.08
flatpak-builder --user --install --force-clean build-dir io.github.igtoth.tracedraw.yml
flatpak run io.github.igtoth.tracedraw
```

### Create a portable bundle

```sh
cd flatpak
flatpak-builder --repo=repo --force-clean build-dir io.github.igtoth.tracedraw.yml
flatpak build-bundle repo ../releases/tracedraw-0.15.0-x86_64-workdirs.flatpak io.github.igtoth.tracedraw master
```

Test a bundle by reinstalling it:

```sh
flatpak install --user --reinstall ../releases/tracedraw-0.15.0-x86_64-workdirs.flatpak
```

## Snap

The Snap uses `strict` confinement with `core24`. Its `home` interface allows
ordinary non-hidden files in the user's home directory, including Desktop,
Documents, and Downloads. `removable-media` is declared but must be connected
manually if external drives are needed:

```sh
sudo snap connect tracedraw:removable-media
```

### Build and install

On this Ubuntu 26.04 host, `core24` must be built in the managed LXD
environment rather than with `--destructive-mode`:

```sh
cd snap
snapcraft pack --use-lxd
sudo snap install --dangerous tracedraw_0.15.0_amd64.snap
snap run tracedraw
snap run tracedraw.cli
```

`--dangerous` accepts a locally built, unsigned artifact. It does **not** make
the installed Snap unconfined.

## Preparing a GitHub release

Keep source manifests and metadata in Git. Build artifacts are intentionally
ignored so that a Git repository stays small; attach the verified `.flatpak`,
`.snap`, and generated `SHA256SUMS` to a GitHub Release. Before publishing:

1. update both manifests to the same upstream version and published checksums;
2. build each artifact from a clean build directory;
3. reinstall and smoke-test both packages;
4. regenerate `releases/SHA256SUMS`;
5. create a Git tag such as `v0.15.0-packaging.1` and attach the files to the
   corresponding GitHub Release.
