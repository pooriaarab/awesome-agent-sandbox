# Contribution Guidelines

Thanks for helping. This list aims to be useful for people making a real selection decision, so entries carry more information than a link and a tagline.

## What belongs here

A project belongs on this list if it helps **run untrusted or AI-generated code under a defined isolation boundary**. That includes the runtimes that provide the boundary, the orchestrators that manage it, and the products built on top.

A project does **not** belong here just because it is agent-related. Agent frameworks, prompt tooling, and MCP servers are out of scope unless they ship a sandbox — and if they do, they go under "Agent Frameworks with Built-in Sandboxes", clearly marked.

## Every entry needs

1. **An isolation model** — one of the three tiers in the README: *shared kernel*, *intercepted kernel*, or *separate kernel*. Add `+ TEE` for hardware confidential computing, say `WebAssembly` where there is no kernel, or say `any` if the project delegates to a RuntimeClass. If you cannot determine it from documentation or source, say so in the pull request rather than guessing.
2. **A named backer** — a company, a foundation, or "Community". "Unknown" is acceptable; blank is not.
3. **A one-line reason it is on the list.** What does it do that its neighbours in the table do not? Marketing copy gets rewritten.
4. **A live star badge**, if it has a public repository:

   ```markdown
   ![Stars](https://img.shields.io/github/stars/OWNER/REPO?style=flat-square&label=%20)
   ```

   Never hardcode a star count. They go stale within days.

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
