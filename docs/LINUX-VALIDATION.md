# Linux 安装支持验证记录

日期：2026-11-10。范围：发行版安装路由、设置界面更新器、插件路径、原生包依赖与安装文档。
安装步骤见 [INSTALL.md](INSTALL.md)，仓库为 [LIghtJUNction/WowType](https://github.com/LIghtJUNction/WowType)。

## 已通过

| 检查 | 结果 | 覆盖范围 |
|---|---|---|
| `bash scripts/test-install.sh` | 34 项通过 | Ubuntu / Debian 基线、Arch 衍生版、Fedora 版本匹配、Manjaro / Nix / Atomic 指引、失败不安装、严格逐文件 SHA256 |
| `python3 scripts/test-update.py` | 10 个主测试通过，含多组子测试 | 本机包查询、对应包管理器、Arch 升级失败停止、版本匹配、路径校验、缺失 / 错误 / 重复校验条目 |
| Rust 文字核心 | 136 项通过，0 失败 | `cargo test --locked -p wowtype-core --no-default-features` |
| Bash / Python / workflow YAML | 通过 | 所有安装 / 构建脚本语法、设置脚本 AST、CI 任务依赖 |
| 原生包元数据 | 通过 | Arch / RPM 生成时记录实际 Fcitx 版本；无效版本拒绝生成 |

安装器回归使用本地资产和替代包管理器；更新器回归使用真实函数与隔离网络 / 进程接口。
这些检查没有安装宿主软件包或重启用户输入法。

## Debian 13 原生编译与加载

环境：Debian 13 x86_64、Fcitx 5.1.12、GCC 14.2、CMake 3.31.6、Rust 1.99.1。
构建依赖从 Debian 官方仓库下载，按软件包索引中的 SHA256 核对后解压到私有前缀。
验证用 CMake 配置仅把依赖头文件路径重定位到该前缀，项目源码未为测试作替换。

以下步骤实际执行通过：

1. 不含语音的插件编译和链接。
2. 默认含语音的 Rust release 核心构建，以及 C++ 插件编译和链接。
3. `DESTDIR` 暂存安装，插件落在 `usr/lib/x86_64-linux-gnu/fcitx5/libwowtype.so`。
4. 对暂存插件执行 `ldd -r`，无缺失动态库和未解析符号。
5. 通过 `ctypes.CDLL` 动态加载暂存插件。
6. 独立配置自定义安装前缀，插件路径位于该前缀的 `lib/fcitx5`。

桌面中的输入、候选框显示和真实麦克风采集未在本轮验证。

## CI 与发布状态

[CI 配置](../.github/workflows/ci.yml)包含：Ubuntu 24.04 / 26.04、Debian 13、Arch、
Fedora 43 / 44 / 45、openSUSE Tumbleweed、Nix，以及独立安装 / 更新回归。
Arch 和 Fedora 的安装检查从 `install.sh` 入口执行；标签发布需上述任务全部通过。

本轮查询该 fork 时，GitHub 返回 0 个 workflow 和 0 次运行，提交后仍未产生运行记录。
已连接的 GitHub 工具可提交代码，但没有启用 Actions 的操作；浏览器没有 GitHub 登录会话。
因此本轮未获得远端发行版矩阵结果，也未发布新的二进制 Release。
管理员需要在 [Actions](https://github.com/LIghtJUNction/WowType/actions) 启用仓库工作流，
再执行 CI；通过后推送对应版本标签可生成发布包。
在此之前，安装文档提供的本机源码 / makepkg 路径可用于构建当前代码。
