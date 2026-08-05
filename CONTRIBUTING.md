# Contribution Guidelines

Thanks for helping. This list aims to be useful for people making a real selection decision, so entries carry more information than a link and a tagline.

## What belongs here

A project belongs on this list if it helps **run untrusted or AI-generated code under a defined isolation boundary**. That includes the runtimes that provide the boundary, the orchestrators that manage it, and the products built on top.

A project does **not** belong here just because it is agent-related. Agent frameworks, prompt tooling, and MCP servers are out of scope unless they ship a sandbox — and if they do, they go under "Agent Frameworks with Built-in Sandboxes", clearly marked.

## Entry format

One line, always this shape — a sentence, then isolation, then how you run it, then who backs it, then stars:

```markdown
- [Name](url) - One sentence, under 20 words. ![isolation](badge) ![how you run it](badge) ![backer](badge) ![Stars](badge)
```

The badges are the comparison. A reader should be able to scan one column and see the difference between two projects without reading either sentence.

### Badge colours have fixed meanings

**Colour is reserved for isolation strength.** Never use a colour from this table for anything else, and never introduce a new one.

| Isolation | Colour    | Badge text            |
| --------- | --------- | --------------------- |
| Weakest   | `d1242f`  | `shared kernel`       |
| Middle    | `bf8700`  | `intercepted kernel`  |
| Strongest | `1a7f37`  | `separate kernel`     |
| Pluggable | `0969da`  | `any runtime`         |
| Strongest, hides from the host | `8250df` | `separate + TEE` |
| Not a sandbox at all | `6e7781` | `not a sandbox`, `bundles a sandbox`, `evaluation` |

The other two badges are deliberately neutral greys so they never compete with the isolation signal:

- **How you run it** — `8b949e`. Say *where* it actually runs: `native CRDs`, `Helm`, `your server`, `your laptop`, `no self-host`, `BYOC on enterprise`. Avoid vague words like "local" — a reader cannot tell whether that means "not Kubernetes" or "runs on your laptop".
- **Who backs it** — `4a5568`. A company, a foundation, or `Community`. `Unknown` is acceptable; blank is not.

Runtimes in the Isolation Runtimes section skip the isolation badge, since the subsection already states the tier. They use a distinguishing fact instead — `125ms boot`, `default on GKE`, `Kata backend`.

### Writing the badge URL

```
https://img.shields.io/badge/<TEXT>-<COLOUR>?style=flat-square
```

Escaping catches people out: spaces are `%20`, a literal `+` is `%2B`, and a literal hyphen must be **doubled** (`no self--host` renders as "no self-host"). Open the URL in a browser before opening the pull request.

### Star badges

Required for anything with a public repository:

```markdown
![Stars](https://img.shields.io/github/stars/OWNER/REPO?style=flat-square&label=%20)
```

Never hardcode a star count — they go stale within days. Entries with no repository end on the backer badge instead.

### The sentence

One claim, not a summary: what does this do that its neighbours do not? Marketing copy gets rewritten. Anything longer belongs in [COMPARISON.md](COMPARISON.md), where the tables have room for it.

## Quality bar

- The project should be usable today, not an announcement or a README with no code behind it.
- Prefer primary sources. Vendor benchmark claims are fine when labelled as claims.
- Closed-source commercial platforms are welcome in the "Managed Platforms" section. Use `—` in the stars column.

## Before opening a pull request

- One project per pull request, unless you are fixing something across the board.
- Keep the table alphabetical only where the section already is — several sections are ordered by relevance on purpose.
- Run [awesome-lint](https://github.com/sindresorhus/awesome-lint) locally: `npx awesome-lint`
- Check your links resolve.

## Removing entries

Removals are as welcome as additions. Open an issue if a project is abandoned, has been absorbed into another, or turned out to be misrepresented here. Include evidence — a last-commit date, an archive notice, a source link that contradicts the entry.

## Corrections

If an isolation model is wrong, that is a bug worth filing. Confusing an intercepted kernel with a separate one is exactly the kind of mistake this list exists to prevent.
