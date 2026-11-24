# ArmSoM Linux SDK Manifests

English | [简体中文](README.zh-CN.md)

Manifest files for downloading and managing the ArmSoM Linux SDK with [Repo](https://gerrit.googlesource.com/git-repo/). Each release selects a coordinated set of source revisions for board configuration, the Linux kernel, U-Boot, root filesystems, and development tools.

**Start here:** [SDK build guide](https://docs.armsom.org/build-system-image/armsom-sdk) · [Product documentation](https://docs.armsom.org/) · [Community forum](https://forum.armsom.org/)

Cloning this repository alone downloads the manifests. Use the steps below to obtain the SDK source tree and its top-level `build.sh` script.

## 1. Prepare a Linux host

Use an **x86_64 Ubuntu 22.04** build host. The SDK guide also lists Ubuntu 20.04. Allow more than **80 GB of free disk space**, with additional capacity for downloaded root filesystems and multiple builds. These commands are for a Linux shell.

Install the source-download tools:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git git-lfs python3
git lfs install
```

Install the Repo launcher:

```bash
mkdir -p "$HOME/.local/bin"
curl -fL https://storage.googleapis.com/git-repo-downloads/repo \
  -o "$HOME/.local/bin/repo"
chmod 0755 "$HOME/.local/bin/repo"
export PATH="$HOME/.local/bin:$PATH"
repo version
```

Keep `$HOME/.local/bin` in your shell's `PATH` for future sessions. See the [upstream Repo installation instructions](https://gerrit.googlesource.com/git-repo/+/HEAD/README.md) for other installation options.

If Git identity is not already configured, set your own details:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Before compiling, also install the **build dependencies** listed in the [SDK build guide](https://docs.armsom.org/build-system-image/armsom-sdk). The short package list above only prepares source download.

## 2. Download the SDK

Create a dedicated SDK workspace and synchronize the default release:

```bash
mkdir -p "$HOME/armsom-sdk"
cd "$HOME/armsom-sdk"

repo init -u https://github.com/ArmSoM/manifests.git \
  -b linux -m armsom_linux_generic.xml
repo sync -c -j4

git -C ubuntu24.04 lfs pull
```

The `ubuntu24.04` project uses **Git LFS**. Its large files must be downloaded successfully before building configurations that use them; a small LFS pointer file is not the actual archive or image.

The `linux` argument selects the manifest repository branch. The manifest's `revision` attributes select the individual SDK project commits.

## 3. Select a board and build

Run these commands from the **SDK workspace**, after synchronization and build-dependency installation have completed:

```bash
./build.sh chip
./build.sh
```

The first command opens the chip and board-configuration selection flow. Choose the configuration matching your board, carrier board, and operating system. The second command builds the selected firmware; generated images are collected in `rockdev/`.

For example, the Sige7 Debian XFCE configuration can be selected directly:

```bash
./build.sh armsom-sige7_rk3588_debian_xfce_linux6.1_defconfig
./build.sh
```

The current default release includes board configuration families for:

| SoC | Configuration families |
| --- | --- |
| RK3528 | Sige1 |
| RK3576 | Sige5, CM5, AIM5 |
| RK3588 | Sige7, AIM7, LM7 |
| RK3588S | CM7s, Sige7s |

These are configuration entries in the pinned [`device_rockchip` source](https://github.com/ArmSoM/device_rockchip/tree/57870e769cab3737707c607937a41da5f5a55447/.chips), not a claim that every board supports every root filesystem or kernel combination. Inspect the configurations available in your checkout:

```bash
find device/rockchip/.chips -type f -name '*defconfig' | sort
./build.sh help
```

For component builds and image packaging, follow the [SDK build guide](https://docs.armsom.org/build-system-image/armsom-sdk). Use the [flashing guide](https://docs.armsom.org/getting-start/flash-img) and the documentation for your board when installing the resulting image.

## 4. Choose a release

[`armsom_linux_generic.xml`](armsom_linux_generic.xml) is a symbolic link to the default release manifest. It currently points to **2026-11-12**. Its target may change as new releases are published.

| Release | Manifest passed to `repo init -m` |
| --- | --- |
| 2026-11-12 — current default | [`armsom_linux/armsom_linux_generic_release_20261112.xml`](armsom_linux/armsom_linux_generic_release_20261112.xml) |
| 2026-06-24 | [`armsom_linux/armsom_linux_generic_release_20260624.xml`](armsom_linux/armsom_linux_generic_release_20260624.xml) |
| 2026-03-12 | [`armsom_linux/armsom_linux_generic_release_20260312.xml`](armsom_linux/armsom_linux_generic_release_20260312.xml) |
| 2025-12-16 | [`armsom_linux/armsom_linux_generic_release_20251216.xml`](armsom_linux/armsom_linux_generic_release_20251216.xml) |
| 2025-11-25 | [`armsom_linux/armsom_linux_generic_release_20251125.xml`](armsom_linux/armsom_linux_generic_release_20251125.xml) |

To select a dated release explicitly, use a separate workspace:

```bash
mkdir -p "$HOME/armsom-sdk-20261112"
cd "$HOME/armsom-sdk-20261112"

repo init -u https://github.com/ArmSoM/manifests.git \
  -b linux -m armsom_linux/armsom_linux_generic_release_20261112.xml
repo sync -c -j4
git -C ubuntu24.04 lfs pull
```

After a successful synchronization, record the checked-out Git revisions:

```bash
repo manifest -r -o armsom-sdk-lock.xml
```

Keep this file with your build configuration and release records. It records project commit IDs; it does not include uncommitted edits, downloaded package versions, or the build-host environment.

## 5. SDK source layout

The current default manifest synchronizes 12 projects. Key paths in the resulting SDK workspace are:

| Path | Contents |
| --- | --- |
| `device/rockchip/` | Board configurations and build scripts |
| `kernel-6.1/`, `kernel-5.10/` | Linux kernel source trees; the selected board configuration determines which is used |
| `u-boot/`, `rkbin/` | Bootloader sources and Rockchip binary components |
| `debian11/`, `debian12/` | Debian root-filesystem build resources |
| `ubuntu22.04/`, `ubuntu24.04/` | Ubuntu root-filesystem build resources |
| `prebuilts/gcc/linux-x86/aarch64/` | x86_64-hosted AArch64 cross-compilation toolchain |
| `armsom-initrd/`, `tools/` | Initrd resources and development tools |

The manifest creates `build.sh`, `Makefile`, and `rkflash.sh` links at the SDK root from files in `device/rockchip/`. Repository URLs are configured in [`remote.xml`](remote.xml); the release manifest records each project's revision and checkout path.

## 6. Update an existing workspace

Inspect your local work first:

```bash
cd "$HOME/armsom-sdk"
repo status
```

Commit or stash local changes in the affected project repositories before updating. Then run:

```bash
repo sync -c -j4
git -C ubuntu24.04 lfs pull
```

Repo also updates the manifest checkout during synchronization. Workspaces initialized with the default alias follow its selected release; workspaces initialized with a dated manifest keep using that manifest file. Project checkouts follow its recorded revisions, rather than automatically tracking each project's latest branch tip. See [Repo's manifest documentation](https://gerrit.googlesource.com/git-repo/+/HEAD/docs/manifest-format.md).

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| `repo: command not found` | Confirm the launcher is executable and `$HOME/.local/bin` is in the current shell's `PATH`. |
| Synchronization fails or times out | Check the first failing project and network access. Retry from the same workspace with `repo sync -c -j1 --fail-fast` to isolate the failure. |
| `build.sh` is missing | Confirm you ran `repo sync` successfully and are in the SDK workspace, not a standalone clone of this manifests repository. |
| Ubuntu 24.04 archives are missing or contain pointer text | Run `git -C ubuntu24.04 lfs pull` and resolve any LFS download or quota errors before building. |
| A board configuration cannot be found | Check the selected release and the files under `device/rockchip/.chips/`; configuration names must match exactly. |
| Local changes prevent synchronization | Save the affected work and resolve the reported conflict. Do not use force-checkout or reset commands just to bypass the error. |

## Support and contributions

- **Manifest issues:** [open an issue](https://github.com/ArmSoM/manifests/issues) with the manifest filename, host OS, board configuration, command, and relevant error output.
- **Board usage and SDK help:** visit the [ArmSoM forum](https://forum.armsom.org/) and [documentation](https://docs.armsom.org/).
- **Documentation changes:** keep the English and Chinese instructions, release list, and command examples consistent.

SDK components retain the licenses supplied by their individual repositories. Review those licenses and any binary-component terms before redistributing an SDK or firmware image.
