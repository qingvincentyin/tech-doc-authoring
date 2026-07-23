---
name: tech-doc-authoring
description: >
  Authoring-time rules for writing and editing Markdown technical documentation —
  the counterpart to tech-doc-consistency-check (which audits existing docs). Use this
  skill whenever you write or edit a Markdown doc, especially the user's guide docs
  ("Infra Guide to GCP AI Agents.md", "DevOps Guide to agents-cli.md"), or any time you
  add a heading, a section cross-reference (§X.Y), a link, an image, a Table-of-Contents
  entry, or an inline HTML anchor. Trigger on: "write this section", "add a heading",
  "link to section X", "cross-reference the other guide", "add an anchor", "insert an
  image", "edit the doc". Covers hard-wrap rules, in-doc and cross-doc link construction,
  GFM slug anchors, non-heading and image anchors, bidirectional cross-links, title-case
  headings, sibling-heading numbering, empty-section-preamble tolerance, list-vs-inline
  enumeration, and pretty-printing an embedded JSON string. Do NOT use this for auditing a
  finished doc (use tech-doc-consistency-check) or for general prose style (that stays in
  the user's global CLAUDE.md).
license: Apache-2.0
metadata:
  author: Vincent Yin
  version: "1.0.0"
---

# Tech Doc Authoring

Apply these rules while writing or editing Markdown technical documentation. They are authoring-time rules — they shape what you produce. Auditing an already-written doc is a separate skill (`tech-doc-consistency-check`); general prose style (short sentences, active voice, plain words) stays in the user's global CLAUDE.md.

## Hard Line Breaks

Do not insert hard line breaks inside a paragraph or list item. Write each paragraph and each list item as a single continuous line and let the viewer wrap to window width. Some of the user's renderers use markdown-it `breaks: true`, where a mid-paragraph newline becomes a literal `<br>` and does not reflow. Leave headings, table rows, and fenced code blocks untouched.

## In-Document Section References

In-document section references (e.g. `§6.4`, `§5.4`, `§4A.2`) must be written as Markdown hyperlinks to the section's GFM slug anchor — e.g. `[§6.4](#64-a2a-registration-on-cloud-run)`. Build the anchor from the heading using GFM slug rules, or copy it from the auto-generated Table of Contents. This applies to new prose and when editing existing sections — do not leave a plain `§X` reference behind.

## Cross-Document References to a Known Sibling Doc

Cross-document references to a **known sibling doc** — one the user distributes side-by-side in the same folder (e.g. the "Infra Guide to GCP AI Agents.md" and "DevOps Guide to agents-cli.md" pair, which share an `images/` subfolder) — are hyperlinked with an **angle-bracket relative link**, because the filenames contain spaces: `[link text](<Other Doc.md#anchor>)`. Angle brackets are required (a bare space breaks the link in strict parsers); do not percent-encode. Deep-link to the specific section wherever a section is cited: build the anchor from the target heading's GFM slug, copying it from that doc's own Table of Contents rather than recomputing it (the em-dash and `--flag=value` slug cases are easy to get wrong). When one sentence cites multiple sections of the other doc, make each `§X.Y` its own separate deep link. The visible link text must still name the target so it degrades gracefully in raw text — keep the full `.md` filename where the prose already uses it, or a clear short name like "the DevOps Guide" — never a vague name that leaves the reader guessing which file is meant. A reference to a file that is **not** a known sibling (not guaranteed to sit in the same folder) stays plain text and names the target by its full filename including the `.md` extension.

## Table of Contents and Heading Anchors

The user's Markdown editor auto-regenerates the Table of Contents on save (a "Markdown All in One"-style extension), rebuilding each entry from the heading text using GFM slug anchors. Do not hand-edit TOC entries' anchors, and do not add custom `<a id>` anchors inside headings (they are stripped when slugs are computed) — reuse the existing GFM slug for cross-references.

## Non-Heading Anchors

To link to a non-heading location — a paragraph, a `**Note:**`, a blockquote, a callout, a sidebar that is not itself a heading — add an inline HTML anchor at the start of that block: `<a id="kebab-name"></a>**Note:** …`. Place it on the same line as the text so it stays in the same paragraph (a separate line risks splitting into its own block under `breaks: true`). Then link to it with `[link text](#kebab-name)`. This is the counterpart to the heading rule above. A heading already has an auto-computed GFM slug, so never add a custom anchor there. A non-heading target has no slug, so an explicit `<a id>` is the only way to link to it. The anchor renders invisibly in GitHub and markdown-it.

## Image Anchors

Exception for an **image** target: put the anchor on its own line before the image, separated by a **blank line** (so the anchor sits in its own paragraph, with the `![...]` in the next block), not on the same line. On the same line the browser jumps to the *end* of the image and scrolls past it, missing the screenshot. A single line break with no blank line fixes some renderers (markdown-it) but not the GitHub web UI, which still scrolls past — the blank line is required for GitHub and works everywhere. With the blank line, the jump lands at the *top* of the image. This is the one non-heading target where a separate block is correct, overriding the same-line rule above.

## Bidirectional Cross-Links

Prefer bi-directional cross-links. When one passage points to another (body→sidebar, note→detail section, section→section), add the return link too, so each end references the other. Point each link at the target's anchor: the heading's GFM slug for a heading target, or a custom `<a id>` (per the rule above) for a non-heading target.

## Title-Case Headings

Write all section headings in title case, not sentence case. Capitalize the first and last word, and every major word in between. Lowercase only the minor words: articles (`a`, `an`, `the`), coordinating conjunctions (`and`, `but`, `or`, `nor`, `for`, `yet`, `so`), and prepositions regardless of length (`of`, `on`, `in`, `to`, `by`, `with`, `from`, `for`, `vs.`, etc.). Capitalize the first word after a colon. Capitalize both parts of a hyphenated compound (e.g. `Per-User`, `Three-Stage`, `High-Code`). Keep code-span/backtick terms (e.g. `` `curl` ``, `` `agents-cli run` ``) and proper nouns verbatim. Capitalization does not change GFM slugs, so converting case never breaks TOC anchors or in-doc cross-reference hyperlinks.

## Matching Sibling Heading Convention

When adding a heading to an existing document, match the convention of a nearby **content** heading of the same level: its numbering scheme and depth, title vs. sentence case, and anchor style. If sibling content headings are numbered (e.g. `##### 6.3.6.1.`), number the new one to continue the sequence; if they are unnumbered, leave it unnumbered. Do not model a content heading on a special-case named callout. `Sidebar` headings (and similar named blocks such as `Note:` or `Example:` callouts) deliberately break the numbering convention and are exempt, so they are the wrong exemplar to copy. The trap to avoid: reaching for the nearest heading when that heading is a callout exception rather than a real content sibling.

## Empty Section Preamble

An "empty section preamble" is fine. A section heading may be immediately followed by a subheading with no text between them (e.g. an `##` heading directly followed by its first `###`). Do not flag this as an issue, and do not insert filler text just to fill the gap. Add a lead-in paragraph only when it carries real orienting content.

## List vs. Inline Enumeration

Prefer a list over a paragraph that enumerates items inline — whether parallel/comparative ("A carries X, while B carries Y"; "on one hand… on the other…"; "if ADK then… if A2A then…"; "A does X, after which B does Y") or inline-enumerated ("(1)…, (2)…, (3)…"; "(a)…, (b)…, (c)…"; "(i)…, (ii)…, (iii)…"). Use a **numbered list** when items are sequential steps or when surrounding prose may need to refer to a specific item by number; use a **bullet list** for unordered parallel or comparative items. Use a lead-in clause ending in ":", one item per entry, and any non-parallel prose that follows as a separate paragraph after the list — never appended to the last item. Inside a parent list item, that means a blank line followed by indented continuation text. Keep prose for narrative/causal flow where sentences connect and for a single short comparison.

## Pretty-Printing an Embedded JSON String

A JSON field's value is sometimes itself a JSON document stored as an escaped string (JSON-in-JSON), such as `jsonAgentCard` or `source_extension_json`. When reflowing one for readability, keep it a **literal string** — do NOT expand it into a well-formed nested object. Retain every escape character verbatim (`\"` for inner quotes, `\\n` for a newline inside an inner string, `\\uXXXX` for a non-ASCII char). Add only structural line breaks and 2-space indentation between JSON tokens, as a pretty-printer would. Open the value with `"{` and close it with `}"`. Keep each string value on a single line (a JSON pretty-printer never breaks inside a string), even a long one. A trivially short nested object or array may stay inline. Two constraints on when to do this at all: (1) only pretty-print an embedded JSON string when the user explicitly asks — leave it as the captured single line otherwise; (2) usually apply it only to strings longer than 100 characters, leaving shorter ones on one line. Verify the reflow is lossless: strip the added whitespace and the result must decode back to the identical object. Example — before:

```json
"payload": "{\"id\": 7, \"note\": \"line1\\nline2\", \"tags\": [\"a\", \"b\"]}"
```

After:

```json
"payload": "{
  \"id\": 7,
  \"note\": \"line1\\nline2\",
  \"tags\": [
    \"a\",
    \"b\"
  ]
}"
```
