# Unofficial TraceDraw packages

Reproducible local packaging for [TraceDraw](https://github.com/igtoth/tracedraw),
currently targeting upstream version **0.30.0**.

TraceDraw is primarily authored by **Ighor Toth**. This is an unofficial
downstream packaging project. It downloads the exact upstream release archives,
verifies their SHA-256 digests, and repackages the binaries; it does not contain
TraceDraw source code. Upstream TraceDraw is licensed under **MIT OR Apache-2.0**.

## Layout

- `flatpak/` — Flatpak manifest and exported desktop metadata
- `snap/` — strict-confinement `core24` Snapcraft project
- `releases/` — locally built distributable artifacts; ignored by Git except
  for `SHA256SUMS`

## Published local artifacts

All artifacts target **x86_64** (also called **AMD64**). The Snap filename uses
`amd64` because that is Snapcraft's architecture name; it is the same CPU
architecture as `x86_64` in the Flatpak filename.

| Artifact | Architecture | SHA-256 |
| --- | --- | --- |
| `tracedraw-0.30.0-x86_64-workdirs.flatpak` | x86_64 (AMD64) | `a96ec3eeb08cffde2b9acf87f800b78ffd34bff11f2311291d457eeccaaec7f4` |
| `tracedraw_0.30.0_amd64.snap` | x86_64 (AMD64) | `af897b278e225e9cacd97197851ee13671446b38cf3e1672436704de907bb2fb` |

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
flatpak build-bundle repo ../releases/tracedraw-0.30.0-x86_64-workdirs.flatpak io.github.igtoth.tracedraw master
```

Test a bundle by reinstalling it:

```sh
flatpak install --user --reinstall ../releases/tracedraw-0.30.0-x86_64-workdirs.flatpak
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
sudo snap install --dangerous ../releases/tracedraw_0.30.0_amd64.snap
snap run tracedraw
snap run tracedraw.cli
```

`--dangerous` accepts a locally built, unsigned artifact. It does **not** make
the installed Snap unconfined.
