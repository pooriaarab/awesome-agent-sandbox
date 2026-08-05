# Contribution Guidelines

Thanks for helping. This list aims to be useful for people making a real selection decision, so entries carry more information than a link and a tagline.

## What belongs here

A project belongs on this list if it helps **run untrusted or AI-generated code under a defined isolation boundary**. That includes the runtimes that provide the boundary, the orchestrators that manage it, and the products built on top.

A project does **not** belong here just because it is agent-related. Agent frameworks, prompt tooling, and MCP servers are out of scope unless they ship a sandbox — and if they do, they go under "Agent Frameworks with Built-in Sandboxes", clearly marked.

## Entry format

One line, always this shape:

```markdown
- [Name](url) - One sentence, under 20 words. `backer` `isolation` `how you run it` ![Stars](badge)
```

Real example:

```markdown
- [agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) - The emerging standard: four CRDs, warm pools, stable identity. `Google + Red Hat` `any runtime` `v0.x` ![Stars](https://img.shields.io/github/stars/kubernetes-sigs/agent-sandbox?style=flat-square&label=%20)
```

The tags are the comparison. Keep them to two or three, lowercase, and reuse the vocabulary already in the file rather than inventing a synonym — a reader should be able to scan one column of tags and see the difference between two projects without reading either sentence.

1. **The sentence** says what this does that its neighbours do not. One claim, not a summary. Marketing copy gets rewritten.
2. **The isolation tag** is one of the three tiers in the README — `shared kernel`, `intercepted kernel`, or `separate kernel`. Use `any runtime` when the project delegates to a RuntimeClass, add `+ TEE` for hardware confidential computing. If you cannot determine it from documentation or source, say so in the pull request rather than guessing.
3. **The backer tag** is a company, a foundation, or `Community`. `Unknown` is acceptable; blank is not.
4. **The star badge** is required for anything with a public repository:

   ```markdown
   ![Stars](https://img.shields.io/github/stars/OWNER/REPO?style=flat-square&label=%20)
   ```

   Never hardcode a star count. They go stale within days. Entries with no repository end the last tag with a period instead.

Anything longer than one line belongs in [COMPARISON.md](COMPARISON.md), where the tables have room for it.

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
