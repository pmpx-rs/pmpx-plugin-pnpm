# pmpx-plugin-pnpm

> English see [README.md](README.md)

[pmpx](https://crates.io/crates/pmpx) 的 **pnpm** 后端。它只做一件事：把 pmpx 的动词映射成
pnpm 命令 —— 不读文件、不看环境变量、不联网。

```console
$ pmpx install           # Node 项目里 → pnpm install
$ pmpx install lodash    #              → pnpm add lodash
$ pmpx run build --foo   #              → pnpm run build --foo
```

## 映射表

| pmpx | pnpm |
| ---- | ---- |
| `install` | `pnpm install` |
| `install <pkg>...` | `pnpm add <pkg>...` |
| `remove <pkg>...` | `pnpm remove <pkg>...` |
| `run <script> [arg]...` | `pnpm run <script> [arg]...` |
| `build [arg]...` | `pnpm build [arg]...` |
| `test [arg]...` | `pnpm test [arg]...` |
| `update` | `pnpm update` |
| `update <pkg>...` | `pnpm update <pkg>...` |
| `exec <cmd> [arg]...` | `pnpm exec <cmd> [arg]...` |

**上面一条都不插 `--`** —— 这跟 npm 和 cargo 正好相反。pnpm 会把参数原样交给脚本，所以插进去
的 `--` 也会被一起交过去：

```console
$ pnpm run probe --foo
GOT:--foo                 # 不插 -- 时，像选项的参数能正常到达脚本

$ pnpm run probe -- --foo
GOT:--,--foo              # 插了 -- 反而把 -- 本身当参数传给了脚本
```

所以 `run` / `build` / `test` 的参数直接过；手写进去的 `--` 也是原样转发，不会被去掉、也不会被
补成两个。

**`run` 的第一个参数就是脚本名。** pmpx 交给插件的参数是 `[target, ...rest]` 拍平后的一个列表，
而 pnpm 的 `run` 正好要脚本名在这个位置 —— 所以整个列表原样跟在 `run` 后面，target 不需要任何
特殊处理。

**`exec` 是支持的** —— 这点和 cargo 后端不同：pnpm 有真正的 `exec` 子命令，用来跑
`node_modules/.bin` 里的可执行文件。

**`install` 无参时是 `pnpm install`**，装锁文件里已经确定的东西；带参数时是 `pnpm add`。

## 安装

```console
$ pmpx plugin add pnpm
```

每次发版还会为常见 target（Linux x64、Windows x64、两种 macOS 架构）上传 prebuilt 产物。
`crate-plugin-kit` 会从同一个 tag 的 release 下载，因此安装通常是一秒而不是一次编译；
没有产物的 target 会回落到从源码编译 —— 只是慢，不是不能用。

## 检测

依据随这个 crate 一起发布的 `pmpx-plugin.toml`：

| 文件 | 权重 | 能证明什么 |
| ---- | ---- | ---------- |
| `pnpm-lock.yaml` | 强（100） | 这个项目确实被 pnpm 解析过 |
| `pnpm-workspace.yaml` | 强（100） | 这棵树是刻意的 pnpm workspace |
| `package.json` | 弱（10） | 只证明属于这个生态，不证明用了哪个工具 |

**两档之间的差距才是重点。** 任何一个 Node 包管理器都能匹配上 `package.json`，所以只有它一个
文件时只有 10 分，在同一棵树里可能输给别的生态 —— 这正是 `.pmpx.toml` 里那些固化项存在的理由。

## 环境要求

Rust **1.82+**，这是契约 crate 的地板。这里的映射不需要更新的版本：它只是构造一个
`CommandSpec` 然后返回。

## 仓库

<https://github.com/pmpx-rs/pmpx-plugin-pnpm>

## 许可

MIT —— 见 [LICENSE](LICENSE)。
