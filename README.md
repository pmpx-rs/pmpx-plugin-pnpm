# pmpx-plugin-pnpm

> 中文版见 [README_CN.md](README_CN.md)

The **pnpm** backend for [pmpx](https://crates.io/crates/pmpx). It maps pmpx's verbs onto pnpm
commands, and does nothing else: no file reads, no environment, no network.

```console
$ pmpx install           # in a Node project → pnpm install
$ pmpx install lodash    #                     → pnpm add lodash
$ pmpx run build --foo   #                     → pnpm run build --foo
```

## The mapping

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

**Not one of these inserts a `--`**, which is the opposite of what npm and cargo do. pnpm hands
arguments to the script exactly as it received them, so an inserted `--` would be handed over
too:

```console
$ pnpm run probe --foo
GOT:--foo                 # without the --, the option-shaped argument reaches the script

$ pnpm run probe -- --foo
GOT:--,--foo              # with it, the -- itself arrives as an argument
```

`run`, `build` and `test` therefore pass their arguments straight through, and a `--` written by
hand is forwarded as-is rather than stripped or doubled.

**`run`'s first argument is the script name.** pmpx gives the plugin `[target, ...rest]`
flattened into a single argument list, and pnpm's `run` wants the script name in exactly that
position -- so the list goes after `run` unchanged, with no special case for the target.

**`exec` is supported**, unlike in the cargo backend: pnpm has a real `exec` subcommand, which
runs a binary out of `node_modules/.bin`.

**`install` with no arguments is `pnpm install`**, which installs what the lockfile already says.
With arguments it is `pnpm add`.

## Install

```console
$ pmpx plugin add pnpm
```

## Detection

From `pmpx-plugin.toml`, which travels with this crate:

| File | Weight | What it proves |
| ---- | ------ | -------------- |
| `pnpm-lock.yaml` | strong (100) | the project was actually resolved by pnpm |
| `pnpm-workspace.yaml` | strong (100) | the tree is a pnpm workspace on purpose |
| `package.json` | weak (10) | the ecosystem, not the tool |

The gap between the two tiers is the point. Every Node package manager matches a
`package.json`, so a tree that has nothing else scores 10 and can lose to another ecosystem in
the same tree -- that is what the pins in `.pmpx.toml` are for.

## Requirements

Rust **1.82+**, which is the contract crate's floor. The mapping here needs nothing newer: it
builds one `CommandSpec` and returns it.

## Repository

<https://github.com/pmpx-rs/pmpx-plugin-pnpm>

## License

MIT — see [LICENSE](LICENSE).
