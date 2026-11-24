# ArmSoM Linux SDK 清单仓库

[English](README.md) | 简体中文

本仓库通过 [Repo](https://gerrit.googlesource.com/git-repo/) 管理 ArmSoM Linux SDK 的源码清单。每个发布清单指定一组配套的代码版本，涵盖板卡配置、Linux 内核、U-Boot、根文件系统和开发工具。

**相关文档：**[SDK 编译指南](https://docs.armsom.org/zh/build-system-image/armsom-sdk) · [产品文档](https://docs.armsom.org/zh/) · [社区论坛](https://forum.armsom.org/)

直接克隆本仓库只能获得清单文件。请按下面的步骤下载完整 SDK 源码，获取 SDK 根目录下的 `build.sh` 脚本。

## 1. 准备 Linux 主机

建议使用 **x86_64 Ubuntu 22.04** 编译主机，SDK 指南也列出了 Ubuntu 20.04。请预留 **80 GB 以上的可用磁盘空间**；下载根文件系统资源或构建多个版本时，还需要更多空间。以下命令均在 Linux 终端执行。

安装源码下载工具：

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git git-lfs python3
git lfs install
```

安装 Repo 启动脚本：

```bash
mkdir -p "$HOME/.local/bin"
curl -fL https://storage.googleapis.com/git-repo-downloads/repo \
  -o "$HOME/.local/bin/repo"
chmod 0755 "$HOME/.local/bin/repo"
export PATH="$HOME/.local/bin:$PATH"
repo version
```

后续打开终端时，也需要确保 `$HOME/.local/bin` 位于 `PATH` 中。其他安装方式见 [Repo 官方说明](https://gerrit.googlesource.com/git-repo/+/HEAD/README.md)。

如果尚未设置 Git 身份信息，请填写自己的姓名和邮箱：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

开始编译前，还需按 [SDK 编译指南](https://docs.armsom.org/zh/build-system-image/armsom-sdk)安装**编译依赖**。上面的简短软件包列表仅用于准备源码下载环境。

## 2. 下载 SDK

创建独立的 SDK 工作目录，同步默认发布版本：

```bash
mkdir -p "$HOME/armsom-sdk"
cd "$HOME/armsom-sdk"

repo init -u https://github.com/ArmSoM/manifests.git \
  -b linux -m armsom_linux_generic.xml
repo sync -c -j4

git -C ubuntu24.04 lfs pull
```

`ubuntu24.04` 仓库使用 **Git LFS** 管理大文件。编译依赖这些资源的配置前，必须完成大文件下载；体积很小的 LFS 指针文件并非实际压缩包或镜像。

`linux` 指定清单仓库的分支，各 SDK 子仓库使用的提交版本则由清单中的 `revision` 属性指定。

## 3. 选择板卡并编译

完成源码同步及编译依赖安装后，在 **SDK 工作目录**中执行：

```bash
./build.sh chip
./build.sh
```

第一条命令进入芯片与板卡配置选择流程，请选择与核心板、底板和目标系统匹配的配置。第二条命令编译所选固件，生成的镜像集中在 `rockdev/` 目录下。

例如，可直接选择 Sige7 的 Debian XFCE 配置：

```bash
./build.sh armsom-sige7_rk3588_debian_xfce_linux6.1_defconfig
./build.sh
```

当前默认版本包含以下板卡配置系列：

| SoC | 配置系列 |
| --- | --- |
| RK3528 | Sige1 |
| RK3576 | Sige5、CM5、AIM5 |
| RK3588 | Sige7、AIM7、LM7 |
| RK3588S | CM7s、Sige7s |

上表根据清单固定版本的 [`device_rockchip` 源码](https://github.com/ArmSoM/device_rockchip/tree/57870e769cab3737707c607937a41da5f5a55447/.chips)整理，表示存在相应配置，不代表每款板卡都支持所有根文件系统和内核组合。可查看当前工作目录中的配置及命令帮助：

```bash
find device/rockchip/.chips -type f -name '*defconfig' | sort
./build.sh help
```

分模块编译和镜像打包方法见 [SDK 编译指南](https://docs.armsom.org/zh/build-system-image/armsom-sdk)。安装生成的镜像时，请结合[系统烧录指南](https://docs.armsom.org/zh/getting-start/flash-img)与对应板卡文档操作。

## 4. 选择发布版本

[`armsom_linux_generic.xml`](armsom_linux_generic.xml) 是指向默认发布清单的符号链接，当前指向 **2026-11-12**。后续发布新版本时，该链接可能更新。

| 发布日期 | `repo init -m` 使用的清单路径 |
| --- | --- |
| 2026-11-12（当前默认） | [`armsom_linux/armsom_linux_generic_release_20261112.xml`](armsom_linux/armsom_linux_generic_release_20261112.xml) |
| 2026-06-24 | [`armsom_linux/armsom_linux_generic_release_20260624.xml`](armsom_linux/armsom_linux_generic_release_20260624.xml) |
| 2026-03-12 | [`armsom_linux/armsom_linux_generic_release_20260312.xml`](armsom_linux/armsom_linux_generic_release_20260312.xml) |
| 2025-12-16 | [`armsom_linux/armsom_linux_generic_release_20251216.xml`](armsom_linux/armsom_linux_generic_release_20251216.xml) |
| 2025-11-25 | [`armsom_linux/armsom_linux_generic_release_20251125.xml`](armsom_linux/armsom_linux_generic_release_20251125.xml) |

需要明确选择某个日期版本时，建议使用独立工作目录：

```bash
mkdir -p "$HOME/armsom-sdk-20261112"
cd "$HOME/armsom-sdk-20261112"

repo init -u https://github.com/ArmSoM/manifests.git \
  -b linux -m armsom_linux/armsom_linux_generic_release_20261112.xml
repo sync -c -j4
git -C ubuntu24.04 lfs pull
```

同步成功后，可以导出当前各仓库的 Git 提交版本：

```bash
repo manifest -r -o armsom-sdk-lock.xml
```

建议将该文件与板卡配置、发布记录一起保存。它记录的是各项目提交 ID，不包含尚未提交的修改、外部下载软件包版本或编译主机环境。

## 5. SDK 源码目录

当前默认清单同步 12 个项目，生成的 SDK 工作目录主要包括：

| 路径 | 内容 |
| --- | --- |
| `device/rockchip/` | 板卡配置和编译脚本 |
| `kernel-6.1/`、`kernel-5.10/` | Linux 内核源码，由所选板卡配置决定使用哪一套 |
| `u-boot/`、`rkbin/` | 引导程序源码及 Rockchip 二进制组件 |
| `debian11/`、`debian12/` | Debian 根文件系统构建资源 |
| `ubuntu22.04/`、`ubuntu24.04/` | Ubuntu 根文件系统构建资源 |
| `prebuilts/gcc/linux-x86/aarch64/` | 在 x86_64 主机上使用的 AArch64 交叉编译工具链 |
| `armsom-initrd/`、`tools/` | Initrd 资源及开发工具 |

清单会将 `device/rockchip/` 中的相关文件链接到 SDK 根目录，生成 `build.sh`、`Makefile` 和 `rkflash.sh` 入口。仓库下载地址在 [`remote.xml`](remote.xml) 中配置，各项目的提交版本与本地路径由发布清单指定。

## 6. 更新已有工作目录

更新前先检查本地修改：

```bash
cd "$HOME/armsom-sdk"
repo status
```

在涉及的项目仓库中提交或暂存需要保留的修改，再执行：

```bash
repo sync -c -j4
git -C ubuntu24.04 lfs pull
```

Repo 同步时也会更新清单仓库。通过默认入口初始化的工作目录跟随该入口所指向的版本；通过日期清单初始化的工作目录继续使用该清单文件。各子仓库按清单记录的提交版本同步，不会自动跟随其开发分支的最新提交。详见 [Repo 清单格式说明](https://gerrit.googlesource.com/git-repo/+/HEAD/docs/manifest-format.md)。

## 常见问题

| 问题 | 排查方法 |
| --- | --- |
| 提示 `repo: command not found` | 检查 Repo 启动脚本是否有执行权限，并确认当前终端的 `PATH` 包含 `$HOME/.local/bin`。 |
| 同步中断或超时 | 检查最先失败的项目和网络访问情况。在原工作目录执行 `repo sync -c -j1 --fail-fast`，降低并发以定位问题。 |
| 找不到 `build.sh` | 确认 `repo sync` 已成功完成，并且当前位于 SDK 工作目录，而不是单独克隆的 manifests 仓库。 |
| Ubuntu 24.04 压缩包缺失或内容是指针文本 | 执行 `git -C ubuntu24.04 lfs pull`，处理 LFS 下载或配额错误后再编译。 |
| 找不到板卡配置 | 检查所选发布版本及 `device/rockchip/.chips/` 下的文件，配置名称必须完全匹配。 |
| 本地修改导致同步失败 | 保存需要保留的工作并处理报错冲突，不要直接用强制检出或重置命令绕过错误。 |

## 支持与参与贡献

- **清单问题：**请在[本仓库提交 Issue](https://github.com/ArmSoM/manifests/issues)，附上清单文件名、主机系统、板卡配置、执行命令和相关错误日志。
- **板卡使用及 SDK 问题：**请查阅 [ArmSoM 文档](https://docs.armsom.org/zh/)或访问[社区论坛](https://forum.armsom.org/)。
- **文档修改：**请保持中英文说明、版本列表和命令示例一致。

SDK 各组件遵循其所在仓库提供的许可证。重新分发 SDK 或固件镜像前，请检查相应许可证及二进制组件的使用条款。
