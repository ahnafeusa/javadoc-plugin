---
name: javadoc
description: Generate or update JavaDoc comments following Oracle's "How to Write Doc Comments for the Javadoc Tool" and Atlassian's Confluence Javadoc standards, with modern JDK (records, sealed types, {@snippet}, inline {@return}). Use when asked to document Java code — a class, method, interface, enum, record, package, file, or set of changes — or to add/fix Javadoc to meet a documentation standard. Documents the full public/protected API thoroughly; skips private internals and pure signature-paraphrases.
allowed-tools: Read Edit Glob Grep Bash(git diff:*) Bash(git fetch:*)
---

# javadoc

## What this is

A workflow for writing or updating JavaDoc comments on Java code so that the **public and protected API is documented thoroughly and consistently**, following the consolidated conventions of:

- **Oracle — "How to Write Doc Comments for the Javadoc Tool"** (the classic spec)
- **Atlassian — Confluence Server Javadoc Standards** (opinionated, practical)
- **Modern Java** — records, sealed types, `{@snippet}`, inline `{@return}` / `{@summary}`, `///` Markdown comments

The full convention catalog — every tag's phrasing rules, the source-conflict resolutions, and worked examples — lives in the companion file **`patterns.md`** next to this one. This file is the procedure; reach into `patterns.md` for the detail behind any rule.

## When to use this skill

Use it when the user wants Java documentation produced to a standard, e.g.:

- "document this class / these methods / this package"
- "add Javadoc to `MoneyUtil`"
- "write Javadoc for the public API in `com.example.http`"
- "update the Javadoc on the methods I changed"
- "/javadoc" on a file, package, or the current diff

Do **NOT** use it for:

- Non-Java documentation (README, design docs, OpenAPI).
- Inline `// why` comments on private implementation logic — that's ordinary commenting, not Javadoc.
- Rewriting prose-style guides; this produces API doc comments only.

## Philosophy: balanced professional

This skill documents the **entire public/protected API surface** — that is Atlassian's baseline and the point of the skill. It applies Oracle's rigor (first-sentence summary, `@param`/`@return`/`@throws`, null behavior, side-effects, thrown conditions) to that surface.

It is *balanced*, not dogmatic, in two ways:

1. **Trivial pure-paraphrase accessors may be left with a one-line summary or skipped** when a doc comment could only restate the signature and the member carries no contract (no null/range rule, no side-effect, no validation). Prefer a real one-sentence contract; never emit `/** Gets the name. @return the name */` noise. But do not leave the broader public API undocumented.
2. **Private and package-private members are not Javadoc'd** by default — they are implementation. If one carries a non-obvious gotcha, that belongs in an inline `// ...` comment, not a doc comment.

The dividing line from a minimalist style: here the **default for public/protected API is to document it**. You skip only the trivial paraphrase, not the API at large.

## Prime directives — read before doing anything

1. **Cover the public/protected surface.** Within the resolved target, every public/protected type and member is a candidate, and the default is to document it well.
2. **Contract, not paraphrase.** A doc must tell the reader something the signature and names don't. If the only available sentence restates the signature and there's no contract to add, write a single clean summary line or skip — never noise.
3. **Don't touch private internals or unrelated code.** No Javadoc on private members; no reflowing or "improving" code you aren't documenting.
4. **Don't invent facts.** Read the member body before documenting null behavior, thrown conditions, or side-effects. Never assert a contract you can't see in the code.

---

## Step 1 — Resolve the target

Determine exactly what to document. Infer from the user's phrasing; ask only if genuinely ambiguous.

| Target | How to enumerate it |
|--------|---------------------|
| **A class / file** | The named `.java` file(s). Document the type and its public/protected members. |
| **A package / directory** | Every `.java` file under that directory (non-recursive unless the user says "and subpackages"). |
| **A whole module / source tree** | All `.java` under the source root. **Large** — confirm with the user first and consider doing it package-by-package so the change is reviewable. |
| **A set of changes (diff mode)** | `git fetch origin` then `git diff origin/main...HEAD` for branch work, or `git diff` / `git diff --staged` for uncommitted. Document the public/protected members the diff **added or changed**. The three-dot form diffs against the merge-base, excluding drift on the base branch. |

Produce a concrete **work-list**: the set of types and members to be documented. Everything downstream operates on this list.

## Step 2 — Enumerate members and their current doc state

For each type in the work-list, list its public/protected members (and the type itself). Read the source — don't work from a diff or signature alone. For each, note the current doc state:

- **none** — no doc comment.
- **present and accurate** — leave it unless the standard is violated.
- **present but wrong / stale** — the code no longer matches the comment; fix it.
- **present but noise** — paraphrases the signature; tighten to a real contract or reduce to a clean summary.

Classify each member: type, constructor, method, constant/field, record component, enum constant.

## Step 3 — What to document (the balanced rule)

**DOCUMENT (default for the public/protected surface):**

- Every **public/protected type** (class, interface, enum, record) — a class-level summary describing what it is and, for shared utilities/services, when to reach for it and what it guarantees.
- Every **public/protected method and constructor** — first-sentence summary plus the applicable block tags (`@param`, `@return`, `@throws`), covering null behavior, side-effects, ranges, and thrown conditions per Oracle/Atlassian rigor.
- **Public constants/fields** — a noun-phrase summary; document units, ranges, and meaning where not obvious.
- **Record components** — one `@param <component>` per component on the record header (see `patterns.md`).

**MAY SKIP or reduce to a single summary line:**

- A trivial getter/setter or pure-paraphrase accessor with no contract to add (no null/range rule, no side-effect, no validation). Prefer one clean summary sentence; skip only if even that would be pure restatement.
- A pure override that adds no behavior beyond the inherited contract — let Javadoc inherit (omit the comment); see `{@inheritDoc}` in `patterns.md`.

**SKIP (no Javadoc):**

- `private` / package-private members. (A genuine gotcha there → inline `//` comment, not Javadoc.)
- Historical notes ("changed from X", "previously did Y") and restatements of framework/stdlib defaults — never.

When in doubt on a *public* member, **document it** (that's the standard). When in doubt on a *private* member, leave it.

## Step 4 — Write the doc (craft rules)

Full detail and examples in `patterns.md`; the high-frequency rules:

**First sentence** — a self-standing summary; Javadoc extracts it verbatim into tables and the index.
- Methods: **verb phrase** ("Returns the active sessions for this user."). Never "This method…".
- Types/fields: **noun phrase** ("A thread-safe LRU cache.").
- Go beyond the name. Watch the **embedded-period gotcha**: the summary ends at the first `. ` (period+space). Avoid "e.g."/"i.e." (write "for example"); if unavoidable, wrap the summary in `{@summary …}` (JDK 10+).

**Block-tag order** (omit any that don't apply):

```
@param   →   @return   →   @throws   →   @see   →   @since   →   @deprecated
```

- **`@param`** — parameter NAME then a lowercase phrase (no leading dash, no `<code>` around the name; Javadoc adds both). Order to match the signature. Carry real info: units, ranges, null/empty contract, collection element type. `@param key the cache key; must not be null`
- **`@return`** — for every documented non-void method; be specific, cover special cases (null/empty/sentinel, out-of-range, collection element type). Omit for `void` and constructors. Prefer inline **`{@return …}`** when the whole description *is* the return value: `/** {@return the number of cached entries} */`.
- **`@throws`** (preferred over `@exception`) — every checked exception and any unchecked one a caller might reasonably catch; phrase as an `if`-clause: `@throws IOException if an I/O error occurs`. Skip `NullPointerException`/`Error` and implementation-bound exceptions.
- **`@deprecated`** — state *when* and the replacement via `{@link}` (or "No replacement"); pair with the `@Deprecated` annotation.
- **`@see`** — for genuinely interesting collaborators/companion helpers, not obvious neighbors.
- **`@author` / `@version`** — Oracle permits them; Atlassian and modern practice drop them (authorship/history live in version control). **Default: omit.** `@since` is the one versioning tag worth using on a published surface — never invent a version.

**Special shapes:**
- **Records** — exactly one `@param <component>` per component **on the record header's class comment only**. Never repeat `@param` on the canonical/compact constructor or hand-write docs on generated accessors.
- **Sealed types** — no `{@sealed}` tag; in prose state it's sealed and `{@link}` each permitted subtype with its meaning.
- **Overrides** — pure overrides: omit and let Javadoc inherit. Partial reuse: `{@inheritDoc}` plus the override-specific note (any comment suppresses the automatic full-text copy).
- **Identifiers in prose** — wrap in `<code>…</code>` (or backticks in a `///` Markdown comment on new code).
- **Examples** — sparingly; when a shared util needs one, use `{@snippet}`, never `<pre>{@code}</pre>`.

## Step 5 — Update vs leave existing docs

- **Member with an accurate, standard-compliant doc:** leave it.
- **Member with a stale/wrong doc:** fix the affected part (changed param, different null/return behavior, new thrown exception, changed side-effect). Don't rewrite the whole comment if only one tag is now wrong.
- **Member with a noise doc** (pure paraphrase): tighten to a real contract, or reduce to one clean summary line per Step 3.
- **Member with no doc on the public/protected surface:** add one (Step 3 default).

## Step 6 — Verify

- Re-read every comment you wrote against the first-sentence rule and the Step 3 balance — delete or tighten any that ended up as pure paraphrase.
- Optionally lint with the standard Javadoc doclint (catches malformed tags, bad references, missing `@param`/`@return`):
  ```
  javadoc -Xdoclint:all -d /tmp/jd <files>
  ```
  or rely on the project's own check (Checkstyle's `JavadocMethod`/`SummaryJavadoc`, an IDE's Javadoc inspection). Surface warnings rather than silencing them.
- Final pass: review the diff to confirm you changed **only** doc comments (and the members you intended), not surrounding code, formatting, or private internals.

## Quick checklist

- [ ] Target resolved to a concrete work-list of types/members (Step 1).
- [ ] Each member's current doc state assessed from the source, not the signature (Step 2).
- [ ] Full public/protected surface documented; trivial paraphrases reduced/skipped; private skipped (Step 3).
- [ ] Docs follow first-sentence + tag-order + phrasing rules (Step 4).
- [ ] Existing docs: accurate left alone, stale fixed, noise tightened (Step 5).
- [ ] Verified: doclint clean (or warnings surfaced), diff touches only doc comments (Step 6).

## Reference

`patterns.md` (same directory) — the full consolidated pattern catalog: per-tag phrasing notes, the six source-conflict resolutions, records/sealed/modern-feature rules, and worked good-vs-bad examples. Consult it whenever a situation isn't covered by the condensed rules above.
