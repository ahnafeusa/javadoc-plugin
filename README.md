# javadoc — a Claude Code plugin

Generates and updates **JavaDoc** comments that follow the consolidated conventions of:

- [Oracle — *How to Write Doc Comments for the Javadoc Tool*](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Atlassian — *Confluence Server Javadoc Standards*](https://developer.atlassian.com/server/confluence/javadoc-standards/)
- Modern Java (records, sealed types, `{@snippet}`, inline `{@return}` / `{@summary}`, `///` Markdown comments)

It documents the **full public/protected API surface** with Oracle's rigor (first-sentence
summaries, `@param`/`@return`/`@throws`, null behavior, side-effects, thrown conditions), while
declining to pad trivial pure-paraphrase accessors and skipping private internals — a *balanced
professional* application of the two standards.

## What's in here

```
javadoc-plugin/
├── .claude-plugin/
│   ├── plugin.json          # plugin manifest
│   └── marketplace.json     # marketplace catalog (this repo is its own marketplace)
└── skills/
    └── javadoc/
        ├── SKILL.md         # the procedure Claude follows
        └── patterns.md      # full convention catalog (tags, conflict resolutions, examples)
```

## Install

This repository doubles as its own marketplace, so installing is two steps in Claude Code:

```text
/plugin marketplace add ahnafeusa/javadoc-plugin
/plugin install javadoc@javadoc-tools
```

To try it before publishing, point the marketplace at a local clone instead:

```text
/plugin marketplace add C:\path\to\javadoc-plugin
/plugin install javadoc@javadoc-tools
```

## Usage

Invoke the skill on whatever scope you want documented:

```text
/javadoc src/main/java/com/example/MoneyUtil.java
/javadoc the public API in com.example.http
/javadoc the methods I changed on this branch
```

It accepts a single file, a class, a package, the whole tree, or the current branch diff.

## License

MIT
