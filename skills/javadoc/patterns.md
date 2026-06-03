# JavaDoc pattern catalog

Companion reference for the **`javadoc`** skill. The skill's `SKILL.md` is the procedure; this file is the full set of conventions behind it, consolidated from three sources:

- **Oracle — "How to Write Doc Comments for the Javadoc Tool"** (the classic spec)
- **Atlassian — Confluence Server Javadoc Standards** (opinionated, practical)
- **Modern Java** — `{@snippet}`, inline `{@return}`, `{@summary}`, records, sealed types, `///` Markdown comments

The two classic sources occasionally disagree; the **Conflict resolutions** section below states how this skill reconciles them, under a *balanced professional* reading: document the full public/protected API (Atlassian's baseline) with Oracle's rigor, while declining to emit noise on trivial pure-paraphrase accessors.

Each pattern is tagged `[core]` / `[recommended]` / `[niche]` so you can tell a near-universal rule from an edge case.

---

## Conflict resolutions (the reasoning behind Step 3)

These are the points where Oracle and Atlassian, or classic and modern practice, diverge. Internalize them.

1. **How much public API to document.** Atlassian: document *every* class, constant, and public method, with `@param`/`@return` even when trivial. Oracle: comprehensive, slightly gentler. **Resolution (balanced):** document the **entire public/protected surface** — that is the standard and the point of the skill. Apply full rigor (null/side-effect/exception coverage) to it. The single concession: a trivial accessor whose comment could only restate the signature, with no contract to add, may carry a one-line summary or be skipped rather than padded with `@param`/`@return` noise. Never let this concession shrink into "document only what earns it" — the public API is documented by default.

2. **`@param`/`@return` for every parameter/return.** **Resolution:** on every documented method, include `@param` for each parameter and `@return` for each non-void return — and make each carry real information (units, ranges, null/empty contract, element type, special-case returns). Prefer inline `{@return …}` when the whole description *is* the return value. Drop a tag to silence noise only on the trivial-accessor case in #1.

3. **Package docs.** Oracle: rich package overviews. Modern: `package-info.java`. Atlassian: don't bother. **Resolution:** add a `package-info.java` overview for any coherent package that presents a public API or has design intent a caller needs — name the key entry-point types and any conventions. Skip incidental groupings. Always use `package-info.java`, never legacy `package.html`.

4. **`@author` / `@version`.** Oracle documents them; Atlassian bans both; modern practice keeps authorship/history in version control. **Resolution:** **omit by default** — they rot and `@author` isn't even emitted into the generated spec. If a project's own house style mandates them, follow that, but don't add them on your own initiative. `@since` is the one versioning tag worth using.

5. **`@since` on every member.** **Resolution:** use it on a **published/shared surface**: one on a new class/interface, and on a member only when it arrived in a later release than its type. Don't sprinkle it on internal code, and never invent an unverifiable version.

6. **Example code in doc comments.** Modern tooling supports `{@snippet}`; Oracle says keep examples out of specs. **Resolution:** examples are valuable on shared utilities a caller must learn to use, but keep them sparing. When you include one, use `{@snippet}` (compilable, validated, highlighted) — never the rotting `<pre>{@code}</pre>`. Large/reused examples go in external snippet-files.

---

## First sentence & summary

**First sentence is a self-standing summary** `[core]`
Write it as a concise, complete summary that reads correctly out of context — Javadoc extracts it verbatim into summary tables and the index. It must not lean on later sentences.
`/** Returns an image that can then be painted on the screen. */`

**Don't open with "This method…" / "This class…"** `[core]`
Methods describe an operation → start with a **verb phrase** ("Gets the label of this button."). Classes/interfaces/fields describe a thing → start with a **noun phrase** ("A button label."). Don't squander the extracted summary on filler.
✅ `Gets the label of this button.`  ❌ `This method gets the label…`

**Watch the embedded-period gotcha; fix with `{@summary}`** `[recommended]`
The summary ends at the first period-followed-by-whitespace (or first block tag), so "e.g."/"Prof." truncates it. On JDK 10+ wrap the intended summary in `{@summary …}`; on older toolchains insert `<!-- -->` after the period. Better: avoid Latin abbreviations entirely ("for example", not "e.g.").
`/** {@summary Parses an ISO-8601 instant such as 2026-01-01T00:00:00Z.} The offset is optional … */`

---

## Structure & formatting

**Doc-comment structure: description, blank line, then block tags** `[core]`
`/** … */` (single trailing asterisk). Description first, blank comment line, then block tags. The **first `@` block tag ends the description** — you cannot resume prose after tags begin. Separate paragraphs with `<p>` (or use `///` Markdown on new code).

**Offset code identifiers; format prose as HTML (or Markdown)** `[recommended]`
Wrap keywords/names mentioned in prose in `<code>…</code>` (Java keywords, type/method/field/argument names, `null`/`true`). Lists via `<ul>`/`<li>`. On JDK 23+ new code, `///` Markdown comments are cleaner — backticks for code, `-`/`1.` for lists, `[Element]` reference links instead of `{@link}` — but you can't mix `///` with `/**` on one declaration.
`May be <code>null</code>` — or in Markdown: `` may be `null` ``

---

## Style & voice

**Use "this" for the current instance; phrases over full sentences are fine** `[recommended]`
Refer to an object of the current class as "this", not "the" ("the toolkit for this component"). Sentence fragments are acceptable for brevity, especially in the summary and `@param`. Use straight ASCII quotes, not smart quotes (smart quotes can corrupt rendered text).

---

## What to document

**Go beyond the API name — never just restate the signature** `[core]`
A doc must reward the reader with something not obvious from the name/signature: behavior, edge cases, null/empty handling, context. If all you can write paraphrases the name and there's no contract to add, reduce to a single clean summary line (and rename if the name itself is unclear).
✅ `Registers text shown in a tool tip when the cursor lingers over the component.`  ❌ `Sets the tool tip text.`

**Document shared utility/helper public API as a contract** `[core]`
On public/protected members of shared utility/helper/API classes, write a real contract: when to reach for it, what it guarantees, edge-case behavior (null inputs, fallbacks, empty results), cross-references to companions. These are the members other code imports — their doc *is* the contract.

**Document business rules, invariants, and non-obvious gotchas** `[core]`
Capture the WHY a reader can't infer: business rules/invariants not visible in code, subtle gotchas (JDBC type mappings, framework lifecycle quirks, async races, browser quirks), non-obvious technical choices that stop the next person "fixing" it back.

**Make side-effects explicit** `[core]`
Spell out state mutation, I/O, caching, event firing. If a method changes anything beyond returning a value, the doc must say so — side-effects can't be inferred from a signature.

**Always document null behavior for arguments and returns** `[core]`
State what happens when an argument is null ("must not be null" suffices if behavior is undefined), and whether the method can return null — and if not, what it returns when there's no value (empty collection, sentinel, default).
`@param key the cache key; must not be null`

**Document the element type of returned collections** `[recommended]`
State what objects a returned collection holds when the signature (raw/wildcard/unclear) doesn't make it obvious.

**Write an implementation-independent contract, not mechanics** `[recommended]`
Document behavior a caller can rely on — boundary conditions, parameter ranges, thread-safety, corner cases — not mechanics (synchronized/native) and not bugs. Isolate genuinely platform-specific behavior in its own paragraph with a clear lead-in.

**If you document a behavior, back it with a test** `[recommended]`
Treat a doc comment as an enforceable contract: anything you assert (null handling, thrown conditions, return guarantees, invariants) should have a test proving it, so comment and code can't silently diverge.

---

## What to omit

**Skip private/internal members; don't pad trivial accessors** `[core]`
Don't Javadoc private or package-private members (a genuine gotcha there is an inline `//` comment). Don't emit `@param`/`@return` noise on a trivial getter/setter that has no contract — a single summary line, or nothing, is correct there. No historical notes ("changed from X"), no restating framework/stdlib defaults. This is the *only* place the skill withholds documentation on the public surface; everywhere else, document.

---

## Block tags

Canonical order — omit any that don't apply:

```
@param  →  @return  →  @throws  →  @see  →  @since  →  @serial  →  @deprecated
```

`@author` and `@version` are omitted by default (see Conflict resolution #4). Repeated-tag ordering: `@param` in argument-declaration order (matches the signature); `@throws` alphabetical by exception simple name; `@see` nearest-to-farthest (`#member`, then `Class`, `Class#member`, `package.Class`).

**`@param`: name not type, lowercase phrase, no dash, no `<code>`** `[core]`
Follow `@param` with the parameter NAME then a lowercase phrase (first noun implies the type). No leading dash (Javadoc inserts it); no `<code>` around the name (Javadoc adds it, and since 1.4 the name must contain no HTML). A pure phrase takes no capital and no trailing period; promote to a full sentence only when you need more than a phrase. Order to match the declaration.
`@param ch the character to be tested`

**`@return`: required for documented non-void methods, specific not redundant** `[core]`
Make it specific; cover special cases (out-of-range, null/empty, element type). Omit for `void` and constructors.
`@return <code>true</code> if completely loaded; <code>false</code> otherwise.`

**Prefer inline `{@return …}` for single-clause return docs** `[recommended]` (JDK 16+)
When the entire description is just what the method returns, use `{@return …}` at the very start instead of summary-sentence-plus-`@return` (eliminates the classic duplication). Composes as `{@return {@inheritDoc}}`.
`/** {@return the number of cached entries} */`

**`@throws`: checked + catchable unchecked, phrased as an "if" clause** `[core]`
Use `@throws` (preferred over `@exception`) for every checked exception and any unchecked one a caller might reasonably catch; describe the **circumstances**, starting with "if". Skip `NullPointerException`/`Error` and implementation-bound exceptions (e.g. `ArrayIndexOutOfBoundsException` for an array-backed method).
`@throws IOException if an input or output exception occurs`

**`@deprecated` must state when and link the replacement** `[core]`
First sentence: when it was deprecated + what to use instead via `{@link}` (or "No replacement"). Pair the `@deprecated` tag (text) with the `@Deprecated` annotation (compiler warning).
`@deprecated since 2.3, use {@link Cache#getOrLoad}`

**`@author` / `@version`: omit by default** `[recommended]`
Authorship and version history live in version control, where they don't rot (`@author` isn't even emitted into the generated spec). Add them only if a project's house style explicitly requires it.

**`@since` only where it carries weight, minimally placed** `[recommended]`
Use on a published/shared surface: one on a new class/interface, and on a member only when it arrived in a later release than its type. Never invent an unverifiable version. `@since 1.2`

**`@see` only for genuinely interesting collaborators** `[recommended]`
For classes/methods that interact in non-obvious ways or are companion helpers worth navigating to — not obvious neighbors.

---

## Inline tags

**Use `{@link}` to hyperlink API names — economically** `[recommended]`
Turn an API name into a hyperlink only when the reader might actually click for more, and only on the first occurrence. Don't link well-known JDK types (`String`) for an experienced audience. Links call attention to themselves in both rendered output and raw source.
`an absolute {@link URL}`

**Refer to a method in prose: parentheses for a specific overload, none for the general method** `[recommended]`
Specific overload → parentheses with arg types: "the `add(int, Object)` method". The method in general → no parentheses, add "method": "the `add` method". Empty `()` falsely implies the no-arg form.

**Let javadoc inherit for pure overrides; `{@inheritDoc}` for partial reuse** `[recommended]`
If an override adds no behavior, omit the comment entirely so Javadoc copies the inherited text (and adds the Overrides/Specified-by link). To reuse most of a parent contract and add override-specific notes, use `{@inheritDoc}` (optionally on a specific `@param`/`@return`/`@throws`). JDK 22+ allows `{@inheritDoc Supertype}` to disambiguate diamond inheritance. **Adding any comment/tag suppresses the automatic full-text copy.**
`/** {@inheritDoc} In addition, persists the change to the audit log. */`

---

## Records, sealed types, modern features

**Document record components with one `@param` per component on the record header only** `[core]`
Put one `@param <component> …` per component in the record's **class** doc comment — this documents the field, its generated accessor, and the canonical-constructor parameter at once. Do **not** repeat `@param` on the canonical/compact constructor and do **not** hand-write docs on generated accessors (Javadoc and IDEs flag the duplication). A compact constructor's comment, if any, describes only added validation/normalization.
```java
/**
 * A point in two-dimensional space.
 *
 * @param x the horizontal coordinate
 * @param y the vertical coordinate
 */
public record Point(int x, int y) { }
```

**Name permitted subtypes in a sealed type's class comment** `[recommended]`
There is no `{@sealed}`/`{@permits}` tag. In prose, state it's sealed and `{@link}` (or Markdown-link) each permitted subtype, explaining why the set is closed and what each case means.
`Sealed; the only implementations are {@link Circle}, {@link Square}, and {@link Triangle}.`

**Use `{@snippet}` (not `<pre>{@code}`) for the example that earns its place** `[recommended]` (JEP 413, JDK 18+)
When a shared utility needs an example to be usable, use inline `{@snippet : … }` or an external snippet-file (`{@snippet file="X.java" region="r"}`) — `{@snippet}` is compilable/validatable and highlighted; `<pre>{@code}` is never compiled and silently rots. Snippet bodies can't contain `*/` and need balanced braces.
`/** {@snippet : var sum = IntStream.rangeClosed(1, 10).sum(); // 55 } */`

---

## Package & module docs

**Add `package-info.java` overviews for coherent API/feature packages** `[recommended]`
Summary-first sentence, no leading heading, key entry-point types `{@link}`ed, conventions noted — for any package that presents a public API or carries design intent a caller needs. Skip incidental groupings. Prefer `package-info.java` over legacy `package.html`. JPMS `module-info` doc comments apply only to modular (non-classpath) builds.

---

## Worked examples — good vs noise

**A shared util method that earns a full contract:**
```java
/**
 * Returns the value for {@code key}, loading and caching it on a miss.
 *
 * <p>Thread-safe: concurrent calls for the same absent key block until the
 * first computation completes, so the loader runs at most once per key.
 *
 * @param key    the cache key; must not be null
 * @param loader supplies the value when {@code key} is absent; must not be null
 * @return the cached or freshly loaded value, never null
 * @throws ExecutionException if the loader throws while computing the value
 * @see #invalidate(Object)
 */
public V getOrLoad(K key, Callable<V> loader) throws ExecutionException { … }
```
Why it earns it: public, on a shared util, with a null contract on both params and the return, a thread-safety guarantee, a thrown condition, and a companion cross-ref — none inferable from the signature.

**A trivial accessor — one clean line, no tag noise:**
```java
// ✅ Acceptable (single summary, no padding):
/** Returns the display name of this user. */
public String getDisplayName() { return displayName; }

// ❌ Noise — delete the tags or the whole comment:
/** Gets the display name. @return the display name */
public String getDisplayName() { return displayName; }
```

**A method whose contract changed — update only the affected tag:**
```java
// Before: returned null on miss. Change made it return Optional.
// Update the @return; leave the rest of the existing doc untouched.
 * @return the matching user, or {@link Optional#empty()} if none matches
```
