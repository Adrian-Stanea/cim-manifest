# cim-manifest

Demo project for [CIM](https://github.com/analogdevicesinc/cim) that keeps its
manifests next to its code, described by a root `.cim.yml`, instead of under
`targets/<name>/sdk.yml`. Used to exercise project manifests over `https://`.

```bash
U=https://github.com/Adrian-Stanea/cim-manifest.git
cim list-targets -s $U
cim list-targets -s $U -t hello          # versions of a target
cim init -t hello -s $U
```

## What is on `main`

```text
.cim.yml                       members: "examples/*", "examples/hello", "tools/*"
                               targets: docs -> extras/docs/sdk.yml, root -> sdk.yml
sdk.yml                        manifest at the repository root, exposed by the `root` entry
examples/hello/                sdk.yml, os/python deps, src/, patches/*.patch, motd.txt
examples/hello-plus/           extends: hello (+ overlay modify, own os deps)
examples/hello-max/            extends: hello-plus (3-level chain, overlay remove)
examples/hello-pinned/         extends: hello@hello-1.0 (base read from a tag)
examples/sandbox/              plain target
examples/notes/                no sdk.yml: not a target
tools/lint/                    plain target
extras/docs/                   only reachable through the explicit `docs` entry
experimental/sdk.yml           not indexed: only reachable with `-M experimental/sdk.yml`
```

## Versions

| Ref | Kind | What it shows |
| --- | --- | --- |
| `hello-1.0` | tag | early layout: one git, no composed targets, `motd.txt` says v1.0 |
| `hello-2.0` | tag | composed targets, `root` entry, two gits, `motd.txt` says v2.0 |
| `hello-dev` | branch | branches are listed as versions too |
| `legacy-1.0` | tag on `demo-legacy` | no `.cim.yml`: classic `targets/legacy/sdk.yml` |

## Scenario branches

| Branch | Index | Expected |
| --- | --- | --- |
| `demo-legacy` | none | classic `targets/*` lookup keeps working |
| `err-single-manifest` | `manifest:` + `name:` | "unknown field: manifest" (single-manifest form is not supported) |
| `err-root-member` | `members: ["."]` | "matches the repository root" |
| `err-broken-index` | unknown key (`membres:`) | clear error, no silent fallback |
| `err-dup-names` | two directories named `x` | "defined more than once" |
| `err-ghost-target` | entry pointing to a missing file | "does not exist" |
| `err-bad-path` | entry containing `..` | "relative to the source repository" |

Try them with `cim init -t <name> -s $U -v <branch> -w /tmp/ws --no-mirror --yes`.
