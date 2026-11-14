# mixumOS 0.1.7 — aarch64 + x86_64

状态：预发布。目标日期：2026-11-14。上一版本：0.1.6。

> 英文原文：[release-notes-0.1.7.md](release-notes-0.1.7.md)。

## 重点特性

- **窗口级双向剪贴板（A 路线）。** A 路线合成器（`mixcomp`）完整实现了 MAP RFC 0004 剪贴板规范：支持能力握手、窗口所有权与 ACL 校验、4 MiB 单帧上限、显式订阅机制（`display.clipboard.subscribe`）以及合成器回声抑制。容器内 Wayland/X11 窗口的复制内容可实时流式推送至被授权客户端，客户端也可以直接向目标窗口注入剪贴板选区。
- **权限代理加固与失败闭环安全保障。** 通过 `mixum-sudo` 发起的特权请求会解析为 root 所属的规范绝对路径，校验 SHA-256 哈希并在执行时使用固定化副本。Shell、脚本封装、相对路径以及非 root 可写二进制在审批前即被拒绝。代理严格采用显式开启策略（`--enable-privilege`），在审计日志中记录审批者身份，实施速率与待审批队列限制，且在启动时若无法解析 `MIXUM_TASK_USER` 则直接拒绝启动（Fail Closed）。
- **特权执行的现代 Coreutils 兼容性。** 固定化可执行副本按原有程序名创建在独立临时子目录中（`<pin>/<ULID>/<程序名>`），彻底解决了 Ubuntu 镜像上自带的 `require-utility` 安全校验拦截问题。
- **快照性能跃升：Manifest 复用与 syncfs 批量化。** 未修改文件直接对比父快照的 `dev/ino/size/mtime` 并复用已有的 Chunk 清单，完全跳过重新读取与哈希计算（300 个小文件无变更快照从 11.2 ms 缩短至 1.3 ms，50 MiB 资产从 27.5 ms 缩短至 0.2 ms）。在 Linux 上通过将分散的逐块 fsync 合并为一次性文件系统级 `syncfs`，10,000 个小文件的冷打包耗时从 21.7 s 骤降至 0.78 s（加速 27.6 倍），恢复耗时从 22.0 s 骤降至 0.50 s（加速 43.6 倍）。
- **Agent 开箱一键安装修复。** `mixum-ctl agent install` 适配了 npm 12 默认禁用生命周期脚本的限制，为 `claude-code` 及 `gemini-cli` 依赖（`keytar`、`node-pty`）配置了脚本放行规则；使用 `--version` 进行严格的有效性验证；并在状态探测时优先前置 Agent 二进制路径。同时，Landlock 沙箱明确放行了 `$HOME/.opencode` 目录的写权限。

## 快照引擎

- 打包器在文件元数据（`mtime`、`dev`、`inode`、`size`）未变动时直接复用父快照 Chunk 列表，将无修改文件的处理转化为仅遍历 stat 的极速路径。
- CAS 存储写入时采用延迟落盘策略，替代以往逐块调用 `sync_all` 的高开销做法，并在 Linux 操作收尾时统一执行 `syncfs`。
- 补充了快照基准测试与单元测试，全面覆盖 Manifest 复用逻辑与数据完整性校验。

## 显示与剪贴板

- 实现 `mixcomp/src/clipboard.c`，处理 Wayland 数据选区并在容器 surface 与 MAP 客户端之间转发。
- `mixum-agentd` 增补 `display.clipboard.subscribe`、`display.clipboard.set` 与 `display.clipboard.changed` 协议处理。
- 窗口 ACL 检查保障客户端仅能获取其获授权窗口的剪贴板数据。
- 引入 4 MiB 载荷保护、JSON 控制字符转义溢出防护以及剪贴板回声抑制队列。
- Web 控制台界面支持在连接窗口后自动完成剪贴板订阅。

## 权限与隔离

- Sudo 审批代理对可执行文件进行绝对路径解析与 root 归属校验，计算并记录目标 SHA-256 摘要。
- 固定化副本隔离在只读专用子目录中并保留原程序名，避免因修改 argv[0] 引发安全告警。
- 拦截 shell 脚本（`sh`、`bash`、`zsh`）、包装脚本及组/全局可写文件。
- 守护进程在无法将 `MIXUM_TASK_USER` 映射至有效本地 UID/GID 时拒绝启动。
- 沙箱为 `$HOME/.opencode` 添加 Landlock 写入白名单。

## 验证结论

- Rust 工作区全部测试通过（`cargo test --workspace`）。
- A 路线双向剪贴板通过 `tests/integration/clipboard-e2e.mjs` 测试。
- 权限代理审批、拒绝和审计流程通过 `tests/integration/privilege-e2e.mjs` 测试。
- 快照 Manifest 复用与批量 fsync 性能在 `rust:latest` 容器基准中完成实测验证。

## 贡献者

- **yubing0721** ([@yubing0721](https://github.com/yubing0721)) — A 路线剪贴板、权限代理加固、Manifest 复用、批量 fsync 优化、Agent 安装修复
- **AshReo** ([@ashreo](https://github.com/ashreo)) — 代码审查、架构对齐、发版统筹
