---
title: 在 Windows 上构建 Zed
description: "在 Zed 开发中构建 Zed for Windows 的指南。"
---

# 在 Windows 上构建 Zed

> 以下命令可在任何 shell 中执行。

## 仓库

克隆 [Zed 仓库](https://github.com/zed-industries/zed)。

## 依赖项

- 安装 [rustup](https://www.rust-lang.org/tools/install)

- 安装 [Visual Studio](https://visualstudio.microsoft.com/downloads/)，并选择可选组件 `MSVC v*** - VS YYYY C++ x64/x86 build tools` 和 `MSVC v*** - VS YYYY C++ x64/x86 Spectre-mitigated libs (latest)`（`v***` 是你的 VS 版本，`YYYY` 是发布年份。根据需要调整架构）。
- 或者，如果你喜欢更轻量的安装，只需安装 [Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)（加上面的库）和 "Desktop development with C++" 工作负载。
  这个配置不会被 rustup 自动获取。编译前，在开始菜单或 Windows Terminal 中启动开发者 shell（cmd/PowerShell）来初始化环境变量。
- 为你的系统安装 Windows 11 或 10 SDK，并确保至少安装了 `Windows 10 SDK version 2104 (10.0.20348.0)`。你可以从 [Windows SDK Archive](https://developer.microsoft.com/windows/downloads/windows-sdk/) 下载。
- 安装 [CMake](https://cmake.org/download)（[一个依赖项](https://docs.rs/wasmtime-c-api-impl/latest/wasmtime_c_api/)需要它）。或者你可以通过 Visual Studio Installer 安装它，然后手动将 `bin` 目录添加到你的 `PATH`，例如：`C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin`。

如果无法编译 Zed，请确保 Visual Studio 安装包含以下组件：

```json
{
  "version": "1.0",
  "components": [
    "Microsoft.VisualStudio.Component.CoreEditor",
    "Microsoft.VisualStudio.Workload.CoreEditor",
    "Microsoft.VisualStudio.Component.VC.Tools.x86.x64",
    "Microsoft.VisualStudio.ComponentGroup.WebToolsExtensions.CMake",
    "Microsoft.VisualStudio.Component.VC.CMake.Project",
    "Microsoft.VisualStudio.Component.Windows11SDK.26100",
    "Microsoft.VisualStudio.Component.VC.Runtimes.x86.x64.Spectre"
  ],
  "extensions": []
}
```

如果只使用 Build Tools，请确保安装了以下组件：

```json
{
  "version": "1.0",
  "components": [
    "Microsoft.VisualStudio.Component.Roslyn.Compiler",
    "Microsoft.Component.MSBuild",
    "Microsoft.VisualStudio.Component.CoreBuildTools",
    "Microsoft.VisualStudio.Workload.MSBuildTools",
    "Microsoft.VisualStudio.Component.Windows10SDK",
    "Microsoft.VisualStudio.Component.VC.CoreBuildTools",
    "Microsoft.VisualStudio.Component.VC.Tools.x86.x64",
    "Microsoft.VisualStudio.Component.VC.Redist.14.Latest",
    "Microsoft.VisualStudio.Component.Windows11SDK.26100",
    "Microsoft.VisualStudio.Component.VC.CMake.Project",
    "Microsoft.VisualStudio.Component.TextTemplating",
    "Microsoft.VisualStudio.Component.VC.CoreIde",
    "Microsoft.VisualStudio.ComponentGroup.NativeDesktop.Core",
    "Microsoft.VisualStudio.Workload.VCTools",
    "Microsoft.VisualStudio.Component.VC.Runtimes.x86.x64.Spectre"
  ],
  "extensions": []
}
```

你可以按以下方式导出此组件列表：

- 打开 Visual Studio Installer
- 点击 `已安装` 选项卡中的 `更多`
- 点击 `导出配置`

### 注意事项

更新 `data` 目录中的 `pg_hba.conf`，将 `host` 方法的认证方式从 `scram-sha-256` 改为 `trust`。否则连接会失败，提示 `password authentication failed`。该文件通常位于 `C:\Program Files\PostgreSQL\17\data\pg_hba.conf`。修改后应如下所示：

```conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
```

如果你使用的是非拉丁语 Windows 区域设置，请在 `postgresql.conf`（在 `data` 目录中）设置 `lc_messages` 参数为 `English_United States.1252`（或你系统上其他可用的 UTF-8 兼容编码）。否则数据库可能会崩溃。文件应如下所示：

```conf
# lc_messages = 'Chinese (Simplified)_China.936' # locale for system error message strings
lc_messages = 'English_United States.1252'
```

完成后，重启 `postgresql` 服务。按 `Win`+`R` 打开运行对话框，输入 `services.msc`，然后选择 **确定**。在服务管理器中，找到 `postgresql-x64-XX`，右键点击并选择 **重启**。

## 从源码构建

安装依赖项后，你可以使用 [Cargo](https://doc.rust-lang.org/cargo/) 构建 Zed。

调试构建：

```sh
cargo run
```

发布构建：

```sh
cargo run --release
```

运行测试：

```sh
cargo test --workspace
```

> **注意：** 视觉回归测试目前仅限 macOS，需要屏幕录制权限。详情请参阅[在 macOS 上构建 Zed](./macos.md#视觉回归测试)。

## 从 msys2 安装

Zed 不支持非官方的 MSYS2 Zed 包（基于 Mingw-w64）。请将 [mingw-w64-zed](https://packages.msys2.org/base/mingw-w64-zed) 的任何问题报告到 [msys2/MINGW-packages/issues](https://github.com/msys2/MINGW-packages/issues?q=is%3Aissue+is%3Aopen+zed)。

请先参阅 [MSYS2 文档](https://www.msys2.org/docs/ides-editors/#zed)。

## 故障排除

### 设置 `RUSTFLAGS` 环境变量会破坏构建

如果你设置了 `RUSTFLAGS` 环境变量，它会覆盖 `.cargo/config.toml` 中构建 Zed 所需的 `rustflags` 设置。

由于这些设置会随时间变化，生成的构建错误可能会有所不同，从链接器失败到其他难以诊断的错误。

如果你需要额外的 Rust 标志，请在 `.cargo/config.toml` 中使用以下方法之一：

在 build 部分添加你的标志

```toml
[build]
rustflags = ["-C", "symbol-mangling-version=v0", "--cfg", "tokio_unstable"]
```

在 windows 目标部分添加你的标志

```toml
[target.'cfg(target_os = "windows")']
rustflags = [
    "--cfg",
    "windows_slim_errors",
    "-C",
    "target-feature=+crt-static",
]
```

或者，在 Zed 仓库的父目录中创建新的 `.cargo/config.toml`（见下文）。这在 CI 中很有用，因为你无需编辑仓库原始的 `.cargo/config.toml`。

```
上层目录
├── .cargo          // <-- 创建此文件夹
│   └── config.toml // <-- 创建此文件
└── zed
    ├── .cargo
    │   └── config.toml
    └── crates
        ├── assistant
        └── ...
```

在新的（如上）`.cargo/config.toml` 中，如果我们想将 `--cfg gles` 添加到 rustflags，它应该如下所示

```toml
[target.'cfg(all())']
rustflags = ["--cfg", "gles"]
```

### Cargo 错误提示依赖项正在使用不稳定功能

尝试 `cargo clean` 和 `cargo build`。

### `STATUS_ACCESS_VIOLATION`

如果你使用 "rust-lld.exe" 链接器，可能会出现此错误。考虑尝试不同的链接器。

如果你使用全局配置，考虑将 Zed 仓库移到一个嵌套目录，并在父目录中添加一个带有自定义链接器配置的 `.cargo/config.toml`。

有关更多信息，请参阅此问题 [#12041](https://github.com/zed-industries/zed/issues/12041)

### 选择了无效的 RC 路径

有时，根据你笔记本上应用的安全规则，编译 Zed 时可能会收到以下错误：

```
error: failed to run custom build command for `zed(C:\Users\USER\src\zed\crates\zed)`

Caused by:
  process didn't exit successfully: `C:\Users\USER\src\zed\target\debug\build\zed-b24f1e9300107efc\build-script-build` (exit code: 1)
  --- stdout
  cargo:rerun-if-changed=../../.git/logs/HEAD
  cargo:rustc-env=ZED_COMMIT_SHA=25e2e9c6727ba9b77415588cfa11fd969612adb7
  cargo:rustc-link-arg=/stack:8388608
  cargo:rerun-if-changed=resources/windows/app-icon.ico
  package.metadata.winresource does not exist
  Selected RC path: 'bin\x64\rc.exe'

  --- stderr
  The system cannot find the path specified. (os error 3)
warning: build failed, waiting for other jobs to finish...
```

要解决此问题，请手动将 `ZED_RC_TOOLKIT_PATH` 环境变量设置为 RC 工具包路径。通常是：
`C:\Program Files (x86)\Windows Kits\10\bin\<SDK_version>\x64`。

有关更多信息，请参阅此 [issue](https://github.com/zed-industries/zed/issues/18393)。

### 构建失败：路径太长

构建时可能会收到以下错误：

```
error: failed to get `pet` as a dependency of package `languages v0.1.0 (D:\a\zed-windows-builds\zed-windows-builds\crates\languages)`

Caused by:
  failed to load source for dependency `pet`

Caused by:
  Unable to update https://github.com/microsoft/python-environment-tools.git?rev=ffcbf3f28c46633abd5448a52b1f396c322e0d6c#ffcbf3f2

Caused by:
  path too long: 'C:/Users/runneradmin/.cargo/git/checkouts/python-environment-tools-903993894b37a7d2/ffcbf3f/crates/pet-conda/tests/unix/conda_env_without_manager_but_found_in_history/some_other_location/conda_install/conda-meta/python-fastjsonschema-2.16.2-py310hca03da5_0.json'; class=Filesystem (30)
```

要解决此问题，请为 Git 和 Windows 启用长路径支持。

对于 git：`git config --system core.longpaths true`

对于 Windows，使用以下 PS 命令：

```powershell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

有关此操作的更多信息，请参阅 [win32 文档](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation?tabs=powershell)

（启用长路径支持后需要重启系统。）

### 图形问题

#### Zed 无法启动

Zed 目前在 Windows 上使用 Vulkan 作为图形 API。如果 Zed 无法启动，Vulkan 是常见原因。

你可以在以下位置查看 Zed 日志：
`C:\Users\YOU\AppData\Local\Zed\logs\Zed.log`

如果看到以下消息：

- `Zed failed to open a window: NoSupportedDeviceFound`
- `ERROR_INITIALIZATION_FAILED`
- `GPU Crashed`
- `ERROR_SURFACE_LOST_KHR`

Vulkan 可能无法在你的系统上正常工作。更新 GPU 驱动程序通常可以解决此问题。

如果日志中没有任何与 Vulkan 相关的内容，而且你恰好安装了 Bandicam，请尝试卸载它。Zed 目前与 Bandicam 不兼容。
