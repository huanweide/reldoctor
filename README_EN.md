**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥18-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/reldoctor)](LICENSE)
<!-- /badges -->

# reldoctor

A zero-dependency release-hygiene CLI. It checks whether `package.json` version, the latest `git tag`, and `CHANGELOG.md` stay consistent, reports a 0–100 health score, and drops straight into CI as a release gate.

> In one line: you changed the code but forgot the tag, the version lagged, the CHANGELOG went unupdated — these extremely common "release incidents" are caught by reldoctor with a single command.

Zero-dependency single file (Node built-ins + system git). Works out of the box — no network, never executes your code.

## Who should use it / Who shouldn't

- **Use it if** you release manually or semi-automatically and want to block "forgot the tag / version drifted" before a release; or you maintain a multi-package monorepo and want a pre-release triangle-consistency check.
- **Skip it if** you fully delegate releases to semantic-release / release-please and trust their consistency (the gate adds little), or you run a library with no CHANGELOG flow and no intention to start one.

## Why it exists

- `standard-version` / `release-please` / `semantic-release` *automate* your release; they don't *audit* whether your release is healthy.
- `changesets` leans toward process collaboration.
- reldoctor does exactly one thing: a static check-up of whether the "version ↔ tag ↔ CHANGELOG triangle" has come apart, with a health score + CI gate. Wire it into a pre-release hook or CI to catch inconsistency *before* you ship.

## Install / Use

```bash
# Run directly (no install, Node 18+)
node reldoctor.js

# Or link globally
npm link   # then just: reldoctor

# Target a directory
reldoctor --root ./packages/core
```

## Detection dimensions

| Rule id | Severity | Meaning |
| --- | --- | --- |
| `version-tag-mismatch` | High | `package.json` version doesn't match the latest git tag |
| `untagged-commits` | High | Commits exist after the latest tag (code changed, not released) |
| `version-not-semver` | Medium | version isn't valid semver (should be `x.y.z`) |
| `changelog-missing-version` | Medium | CHANGELOG has no section for the current version |
| `never-released` | Low | A version exists but the repo has never cut any semver tag |
| `tag-prefix-inconsistent` | Low | tag naming mixes `v` prefix and no prefix (e.g. `v1.0.0` vs `1.0.0`) |
| `no-changelog` | Low | Missing CHANGELOG.md |
| `no-unreleased-section` | Low | CHANGELOG has no `Unreleased` section |

## Health score

Out of 100, deducted by severity: High −20 / Medium −12 / Low −6, floored at 0.

## CI gate

```bash
# Any High issue fails (exit 1)
reldoctor --fail-on-high

# Fine-grained control
reldoctor --max-high 0 --max-medium 2 --max-issues 5 --min-score 80

# Pipeline parsing (JSON output)
reldoctor --json
```

Exit codes: `0` pass, `1` gate failed, `2` argument/path error.

## Related tools

reldoctor is one of the four git-layer governance tools of the "code-health family":

- **repodoctor** — git repo engineering hygiene (dangerous files / large files / LICENSE / noise-commit ratio)
- **docdoctor** — Markdown doc health (dead links / heading structure / image alt / bare URLs)
- **commitdoctor** — git commit-message discipline (empty subject / too long / secret leak / WIP)
- **reldoctor** — release consistency (version ↔ tag ↔ CHANGELOG)

The eight source-layer tools (devdoctor / testlite / debtlens / a11ydoctor / awaitscan / cycscan / secscan / dupscan) cover SonarQube-style static smell dimensions.

## Design discipline

- Zero dependency: only Node built-ins `fs` / `path` / `child_process`.
- Safe: read-only `fs` and system `git`; git calls go through `execFileSync` + `cwd`, never shell string concatenation.
- Robust: non-git repo / missing package.json / bad JSON / git failure all degrade to "no data" instead of crashing.
- Gate guardrails: every threshold is validated with `Number.isFinite`; non-finite values exit 2 immediately.

## Roadmap

Rule set is stable. Future work may add multi-tag-strategy support (e.g. monorepo independent versions). No breaking changes planned.

## Contributing

Issues and pull requests are welcome. Keep it single-file and zero-dependency.

## ⭐ Star

If reldoctor ever caught a "version / tag / CHANGELOG drift" release incident for you, a Star ⭐ helps others find it: https://github.com/huanweide/reldoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
