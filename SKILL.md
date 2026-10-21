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
  GFM slug anchors, non-heading and image anchors, hyperlinking figure/diagram/table
  references, bidirectional cross-links, title-case headings, sibling-heading numbering,
  un-numbered named headings (`Sidebar N:`, `Figure N:`) nesting exactly one level below
  their parent section, sidebar bodies written as a blockquote,
  empty-section-preamble tolerance, list-vs-inline enumeration, no automatic bold lead-in on
  bullets, color-blind-safe diagram
  color and palette economy, forward-reference fragility, and pretty-printing an embedded
  JSON string. Do NOT use this for auditing a
  finished doc (use tech-doc-consistency-check) or for general prose style (that stays in
  the user's global CLAUDE.md).
license: Apache-2.0
metadata:
  author: Vincent Yin
  version: "1.33.0"
---

# Tech Doc Authoring

Apply these rules while writing or editing Markdown technical documentation. They are authoring-time rules — they shape what you produce. Auditing an already-written doc is a separate skill (`tech-doc-consistency-check`); general prose style (short sentences, active voice, plain words) stays in the user's global CLAUDE.md.

## Rewriting Is Authoring — Re-Ask the Shape

Every rule below applies as fully to a rewrite as to a first draft. This needs saying because it is where the rules go quiet. When the task is *fix this sentence*, *make this complete*, *tighten this*, the existing text hands you a form, and improving it inside that form feels like the whole job. It is not. Expanding prose into better prose is still a choice of structure, made by default instead of on purpose.

Before editing any passage longer than a sentence, re-ask the structural question as if the passage did not exist: **what shape does this content want?** Sequential steps want a numbered list. Parallel items want bullets. A comparison wants a table. Narrative cause-and-effect wants prose. Inheriting the shape of what you are replacing is not an answer.

The cue is the task verb. *Rewrite*, *expand*, *complete*, *clarify* all read as sentence-level instructions and all can require a structural change to satisfy.

(Set 2026-09-20. In one Infra Guide co-edit session Claude missed 2 rules in a row this way. Asked to complete a paragraph about the pre-token Lambda, Claude expanded it into 5 sentences of sequential prose — Cognito invokes → Lambda returns → Cognito mints — with **List vs. Inline Enumeration** loaded and explicitly naming that construction. The user restructured it into 3 bullets and asked: *"Why didn't you do that? You have an authoring rule that says to prefer bullets than a sequential prose."*)

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

## Referencing a Figure, Diagram, or Table

When prose refers to a captioned object in the same doc — a Figure, System Diagram, Table, Listing, or the like — make the reference a Markdown hyperlink to that object, the same way an in-document `§X` reference is hyperlinked. Do not leave a plain-text "Figure 3" or "Table 2" behind.

Create the target anchor if it does not exist yet. A caption is a non-heading target, so it needs an explicit `<a id>` (a heading's auto GFM slug does not apply). Use a stable kebab-case id that names the object: `figure-3`, `system-diagram-6`, `table-2`. Place the anchor per the anchor rules above — inline at the start of a **text** caption (`<a id="figure-3"></a>**Figure 3: …**`, so the reader lands on the caption with the object right below), or on its own line before a blank line when the target is an actual `![...]` image (the image-anchor exception).

Point the reference at the caption anchor, never at the enclosing section heading. Linking "Figure 2" to its section's heading slug lands the reader at the section top, not the figure, and silently breaks if the figure moves within the section or a sibling figure is added ahead of it.

Only anchor the doc's own captioned objects. A caption borrowed from an external source (e.g. "RFC 6749's own Figure 3") stays plain text — it is not a target in this doc.

## Mermaid Diagrams

For a line break inside any Mermaid label — node text, edge label, `note for ...`, subgraph title — use `<br/>`, never `\n`. `<br/>` is the portable break that renders everywhere the user's diagrams appear (GitHub, markdown-it, the consistency-check pipeline, artifacts). `\n` is renderer- and diagram-type-dependent: some renderers ignore it or print it literally. Standardizing on `<br/>` also matches the existing System Diagrams in the guides (which use `<br/>` and `&nbsp;`). Related global rule: call a connector between nodes an "arrow", not an "edge".

Fold a node's annotation INTO the node, not into a floating note box. A `note for X` (classDiagram) or a detached note node renders as its own box, competing visually with the real nodes and pulling the reader's eye away from the topology. Put the annotation text inside the class body (or node label) instead. Reserve a separate note only when the annotation genuinely belongs to no single node. Prefer conveying a structural fact through the arrows themselves over restating it in text.

### Diagram Color Must Survive Color Blindness

**The user is color blind.** These rules are not a nicety; a diagram that fails them is unreadable to its primary reader.

**Color is never the only channel.** Any distinction a diagram draws — works vs blocked, primary vs secondary, changed vs unchanged — must survive being read in greyscale. Carry it on at least one non-color channel as well: line style (solid vs dotted), terminator shape (arrowhead vs `x`), border weight, or explicit label text. Color is then reinforcement, never the message itself. This rule outranks the palette rule below, because a palette blocklist only names the combinations already known to fail.

**Red is the least reliable hue.** Do not pair red with green, black, dark grey, or brown to mark contrasting components. Red-green is the most common deficiency. Red against a dark neutral fails separately, under protan-type deficiency, where red's perceived brightness collapses so red reads as dark rather than as a distinct hue. Blue against red is a safe pair. The guides use blue `#2563eb` for a working path and red `#dc2626` for a blocked one.

**A marker can silently inherit the wrong color.** Mermaid mints per-color arrow markers keyed by hex (`crossEnd__dc2626`, `pointEnd__2563eb`) and derives that color from `linkStyle default`, ignoring an indexed `linkStyle`. So `linkStyle default stroke:<color>` recolors a blocked arrow's `x` terminator to the working-path color while leaving the line itself red — the shape that means "blocked" ends up wearing the color that means "works." Assign stroke **by index** whenever any arrow needs its own color. Then confirm in the rendered SVG that each `marker-end` points at the expected `__<hex>` variant:

```bash
mmdc -i diagram.mmd -o diagram.svg
grep -o 'marker-end="[^"]*"' diagram.svg | sort | uniq -c
```

Verify color changes by rendering, never by reading the source. Related: the `reference-mermaid-cli-diagram-verification` memory.

### Keep the Palette Small

Every distinct color in a diagram reads as a claim that something differs. A diagram that gives each distinction its own color becomes a mosaic, and it communicates less than one built from a few coarse colors. Two rules follow.

**Do not spend a color on a meaning another channel already carries.** Topology, shape, position, line style, and label text all encode meaning, and unlike color they survive greyscale and any color deficiency. A node that a dozen arrows converge on is already visibly the centerpiece of the diagram, so a unique border color for it buys nothing and costs a hue. Before adding a color, name the channel that would be missing without it. If you cannot, do not add it. This is the converse of the never-only-color rule above: color must never be the sole channel, and it must not be a redundant one either.

**Sharing one color across boxes that mean different things is an acceptable trade, not a defect.** Weigh palette size against precision. Reserve distinct colors for the differences the section is actually about, and let everything else share. "Same color, different meaning" is only a real problem when the reader must tell those things apart to follow the diagram. Do not propose recoloring on the grounds of exactness alone.

In practice, prefer adding a node to an existing `classDef` over writing a one-off `style` line for it. That removes the exception rather than restyling it, and it keeps the palette from growing one node at a time.

**Never leave a node or subgraph on mermaid's default styling.** An unstyled node renders `#ECECFF` on `#9370DB` purple, and an unstyled subgraph renders `#ffffde` on `#aaaa33` yellow. Those are theme defaults nobody chose, and nothing in the output distinguishes them from a deliberate decision — so a reader takes the purple as a category that means something. Give every node a `classDef` and every subgraph an explicit `style`, even where the value you pick is close to the default it replaces.

**Use one color for all subgraphs, at every nesting depth.** A nested region does not get its own fill. The nesting already shows containment and the subgraph title already names the region, so a second color spends a hue on what 2 channels carry. The inner box stays legible regardless, because its own stroke still draws the outline. Vary a subgraph only when it differs in *kind* rather than in position — a conceptual grouping versus a deployment boundary, say — and express that with `stroke-dasharray` rather than with another color.

(Set 2026-08-04. I had recommended 6 edits to fix a shared green that no reader would have been confused by, and separately kept a unique color on a node whose importance the arrows already showed.)

## Bidirectional Cross-Links

Prefer bi-directional cross-links. When one passage points to another (body→sidebar, note→detail section, section→section), add the return link too, so each end references the other. Point each link at the target's anchor: the heading's GFM slug for a heading target, or a custom `<a id>` (per the rule above) for a non-heading target.

## Title-Case Headings

Write all section headings in title case, not sentence case. Capitalize the first and last word, and every major word in between. Lowercase only the minor words: articles (`a`, `an`, `the`), coordinating conjunctions (`and`, `but`, `or`, `nor`, `for`, `yet`, `so`), and prepositions regardless of length (`of`, `on`, `in`, `to`, `by`, `with`, `from`, `for`, `vs.`, etc.). Capitalize the first word after a colon. Capitalize both parts of a hyphenated compound (e.g. `Per-User`, `Three-Stage`, `High-Code`). Keep code-span/backtick terms (e.g. `` `curl` ``, `` `agents-cli run` ``) and proper nouns verbatim. Capitalization does not change GFM slugs, so converting case never breaks TOC anchors or in-doc cross-reference hyperlinks.

## A Parent Heading Lists Its Contents Only When They Are Obscure

A parent section's heading may name the concepts its subsections cover, or it may name only the topic (`12.2. Primer: Model Armor`). Choose by how familiar the concepts are to the reader.

- **Obscure or confusing concepts:** Name them in the heading. The names catch a reader who has been tripped up by exactly those concepts, and that payoff outweighs the cost of renaming the heading when a subsection is added. Example: `6.1. Primer: Session, State, and Memory`, `11.4. Primer: Topology of App, Collection, Connector, Data Store`.
- **Familiar concepts:** Name only the topic. A list of familiar terms is unstable, because every new subsection forces a rename. It also tells the reader "you know all this", which discourages them from reading content they need. Example: `12.2. Primer: Template, Endpoint, Screening Point, Calling Identity` became `12.2. Primer: Model Armor`.

⛔ Do not flag an enumerating heading for instability alone. Ask first whether its concepts are obscure. (Set 2026-10-17, in the `#119` Infra Guide co-edit. Claude listed §6.1 and §11.4 as having the same problem as the old §12.2. The user explained the difference.)

## A Quotable Subsection Heading Names Its Subject

Decide case by case whether a subsection heading names its subject. The test is whether the section is likely to be quoted on its own, without its parent: in a link from another chapter, a quote in another doc, a search hit, or a browser tab title.

- Likely quoted alone: Name the subject. A self-contained section that explains a concept is the typical case, such as a primer's subsection. Write `12.2.1. Model Armor Template: Filters, Enforcement`, not `12.2.1. Template: Filters, Enforcement`. Out of context, "Template" has no owner, and a chapter that involves several products leaves the reader guessing.
- Unlikely quoted alone: Leave the heading short. A procedural step that only makes sense inside its walkthrough is the typical case, such as `11.3.2.3. Set the Shell Variables`.

The balance matters. Adding the subject everywhere makes the TOC dense, and the repeated words drown out the keywords that are supposed to catch the reader's eye.

- Placement: Put the subject where it reads naturally. A trailing phrase is fine (`4 Places to Screen With Model Armor`).
- Already named: Leave a heading alone when it already names the subject (`Identity That Calls Model Armor`).
- Scope: Apply this to new and edited headings. Do not sweep existing chapters unless the user asks. A rename changes the GFM slug, so retarget every inbound link in the same edit.

(Set 2026-10-17, in the `#119` Infra Guide co-edit. The user renamed §12.2.1–§12.2.3 to add "Model Armor", then limited the rule to sections likely to be quoted on their own.)

## Matching Sibling Heading Convention

When adding a heading to an existing document, match the convention of a nearby **content** heading of the same level: its numbering scheme and depth, title vs. sentence case, and anchor style. If sibling content headings are numbered (e.g. `##### 6.3.6.1.`), number the new one to continue the sequence; if they are unnumbered, leave it unnumbered. Do not model a content heading on a special-case named callout. `Sidebar` headings (and similar named blocks such as `Note:` or `Example:` callouts) deliberately break the **numbering** convention and are exempt from it, so they are the wrong exemplar to copy. The trap to avoid: reaching for the nearest heading when that heading is a callout exception rather than a real content sibling. ⚠️ The numbering exemption does not extend to nesting — see the next rule.

## Un-Numbered Named Headings Nest Exactly One Level Deeper

An un-numbered named heading — `Sidebar N:`, `Figure N:`, `System Diagram N:`, `Note:`, `Example:`, and similar named callouts — takes a level **exactly one deeper than the content heading it belongs to**. Never two or more deeper, and never the same level or shallower. A sidebar under `### 1.1.` is `#### Sidebar 1: …`. A sidebar under `#### 6.3.1.` is `##### Sidebar 4: …`. The same sidebar sequence therefore lands at different depths in different parts of a doc, and that is correct — depth follows the parent, not the label.

**The reason is rendering, not tidiness.** GitHub builds its rendered Table of Contents from heading levels. A skipped level inserts a phantom nesting step, so the entry indents one notch too far and reads as filed under a section that does not exist. The same skip degrades screen-reader outline navigation.

Two consequences when you change a heading's level:

- Re-indent its Table-of-Contents entry to match. Use the doc's own indent unit — the Infra Guide's TOC indents 4 spaces per level, the DevOps Guide's 2.
- Nothing else needs touching. A level change does not alter the GFM slug, so every anchor and cross-reference to that heading keeps working.

(Set 2026-08-10. The rule replaces an earlier convention that held all of a document's sidebars at one uniform depth, which is what produced the `### 1.1.` → `##### Sidebar 1:` skip in the Infra Guide.)

## Sidebar Body Is a Blockquote

Write every line of a `Sidebar N:` body as a blockquote, from the line after the heading up to the next heading. Prefix each content line with `> `, put a bare `>` on each blank line between blocks, and indent a nested bullet as `>   - `. A fenced code block inside a sidebar is quoted line by line too (`> ```python`). The heading itself stays outside the quote.

The blockquote is what sets a sidebar apart from the body text around it when the doc renders. A sidebar written as plain paragraphs reads as part of the section it interrupts. When you add a sidebar, copy the shape of an existing one in the same doc.

(Set 2026-10-05. Claude wrote Infra Guide Sidebar 14 as plain text. The user: *"The content of a sidebar should be blockquoted."* A scan then found Sidebars 15 and 16 unquoted too.)

## Empty Section Preamble

An "empty section preamble" is fine. A section heading may be immediately followed by a subheading with no text between them (e.g. an `##` heading directly followed by its first `###`). Do not flag this as an issue, and do not insert filler text just to fill the gap. Add a lead-in paragraph only when it carries real orienting content.

## Nested Parentheses Are Fine

A parenthetical may contain a parenthesized link, such as `(whose tokens are certificate-bound ([§12.2.2](…)))`. ⛔ Do not flag the stacked closing parentheses, and do not propose rewriting to avoid them. (Set 2026-10-19, Infra Guide §12.2.4. The user: *"don't worry about the nested parentheses"*.)

## Cold Open

A section may open directly on its first line of content, with no orienting lead-in. The heading has already named the subject, so a paragraph restating it adds nothing. Do not flag a cold open, and do not propose adding a lead-in — not when the section follows code blocks, not when it follows another section's conclusion, and not when the first line is a bare list lead-in.

The user's guide docs use a settled cold-open form for a walkthrough: the heading, then `<Actor> starts like this:`, then the list of steps. Infra Guide §5.3.1 opens `Bob starts like this:` and §6.3.2.4 opens `Peter (the end user) starts like this:`. Both are deliberate.

Add a lead-in only when it carries orienting content the heading does not — a scope limit, a prerequisite, or a pointer the reader needs before step 1. "The reader just came from a code block and needs easing in" is not such content.

(Set 2026-07-29, after I flagged §6.3.2.4's cold open twice and the user pointed at §5.3.1 as the established precedent.)

## List vs. Inline Enumeration

Prefer a list over a paragraph that enumerates items inline — whether parallel/comparative ("A carries X, while B carries Y"; "on one hand… on the other…"; "if ADK then… if A2A then…"; "A does X, after which B does Y") or inline-enumerated ("(1)…, (2)…, (3)…"; "(a)…, (b)…, (c)…"; "(i)…, (ii)…, (iii)…"). Use a **numbered list** when items are sequential steps or when surrounding prose may need to refer to a specific item by number; use a **bullet list** for unordered parallel or comparative items. Use a lead-in clause ending in ":", one item per entry, and any non-parallel prose that follows as a separate paragraph after the list — never appended to the last item. Inside a parent list item, that means a blank line followed by indented continuation text. Keep prose for narrative/causal flow where sentences connect and for a single short comparison.

**Trigger — announcing a count commits you to a list.** The moment you write a cardinal followed by a countable noun ("Two things differ", "3 reasons", "four main differences", "two ways", "several caveats"), you have promised the reader an enumeration. Deliver it as a Markdown list or a table, never as the following prose sentence. This is the single most-missed case of the rule above, because the inline form reads fine one sentence at a time — "Two things differ. The path carries an extra segment, and the URL uses a number." — and only the promise makes it wrong. Treat the count word itself as the cue: on writing it, stop and set up a `:` lead-in with bullets.

**A second trigger shape: the presence/absence pair.** Two clauses of the form "with X, A happens" and "without X, B happens" are parallel comparative items, so they want 2 bullets under a lead sentence naming X. The inline form hides it, because each clause reads as ordinary cause-and-effect prose on its own. (Set 2026-09-21, Infra Guide §10.11.2. Claude wrote *"The header's value is the routing key: each session id gets its own microVM. A request without the header gets a new session, and therefore a new microVM."* as one paragraph, with this rule loaded. The user split it into a lead sentence plus those 2 bullets. ⭐ Note the lead sentence ends in a period, ⛔ not the `:` this rule's main form calls for — correct here, because the bullets are consequences of the lead, not a set the lead promises.)

The trigger does **not** fire on a back-reference to something already enumerated — "the two forms", "all four types", "the same 3 steps". A definite determiner before the number marks it as a pointer backward, not a promise forward.

**Mixed-altitude passages — commit the structure, and use the list as a diagnostic.** A flat paragraph linearizes the logical tree of its sentences: some are parallel siblings, and one may be a summary that closes over several earlier ones. Flattening loses that shape, so adjacency implies false parentage — a sentence that concludes *several* earlier ones, left trailing after the last, reads as elaborating only that last one. When a passage mixes per-item detail with a cross-item conclusion, structure it up front: a list for the parallel items, then the conclusion as its own paragraph after the list (per the rule above). The list is also a diagnostic. If you cannot place every sentence as either a parallel leaf or the closing summary, the paragraph has a hidden scoping error.

## No Automatic Bold Lead-In on Bullets

Do not open a bullet with a bold lead-in phrase by default. Write `- The access token is signed, not encrypted. …`, not `- **The access token is signed, not encrypted.** …`. The same applies to a label-style bullet: write `- Arrow 8 (Flow B): …` in plain text.

Bold is an emphasis signal, and it only works while it is rare. A bold lead-in on every bullet spreads bold across the whole doc, and the few words that truly need emphasis then drown among them. Use bold only where the words are a real emphasis the reader must not miss, such as a warning, a counter-intuitive fact, or the one term a passage turns on. The test is whether the reader would lose something if that bold were plain text. If not, leave it plain.

**A contrast pair is a legitimate use.** When 2 bullets exist to set 2 named things against each other, bold the keyword each bullet turns on — `- **Pre sign-up** Lambda can be used as an AWS-side control.` against `- **Pre-token-generation** trigger cannot deny.` The bold there is not decoration on a lead-in. It is the contrast itself, and it passes the test above, because plain text would make the reader hunt for which 2 things are being compared.

Scope:

- Apply this to new and edited bullets only. Do not sweep the existing bold lead-ins out of a doc unless the user asks.
- When the user removes bold lead-ins, treat it as deliberate. Do not flag the result as inconsistent with neighboring sections that still carry them.
- When the user *adds* bold, treat that as deliberate too. ⛔ Do not flag it as drift, and do not flag it as reversing an unbolding sweep they asked for earlier in the same range. Ask what the bold is doing before assuming it is a lead-in.
- ⛔ **Do not raise bold density in the user's text at all.** Not as a flag, not as a suggestion, not as a take-it-or-leave-it aside. This rule binds your own authoring, and the user's bold is theirs to judge.
- Leading with the key word still matters. Put the most important word first in plain text, per the global CLAUDE.md bullet rule.

(Set 2026-09-18. The user stripped the bold lead-ins from 3 bullets in the Infra Guide's §10.4, and I flagged the change as a consistency break against §10.5.3. The user: *"It makes the whole doc littered with bold words which drowns the real important bold words."*)

(Do-not-raise-density bullet set 2026-09-20, minutes after the contrast-pair one. Claude suggested thinning the user's 5 `**service**`/`**people**` marks in Q4, having written 122 of the doc's own bold bullet lead-ins. The user: *"you're accusing me of a crime which you yourself are more guilty of."* ⭐ The asymmetry is the point: the user's bold carries meaning, Claude's default bold is decoration.)

### Sweeping an Existing Range

When the user *does* ask you to sweep a range, do not strip every bold lead-in in it. **Judge one list at a time, not one bullet at a time.** The same bold shape earns opposite verdicts in 2 different lists, and the surrounding list is what settles it.

The test, in the user's own words: *"If the leading sentence only has several selected key words bolded, then that's deliberate, and you should keep them that way. If the entire leading sentence is bold, and such bold pattern is uniform across all bullets in that same list, then obviously this is a good candidate and you should unbold them."*

Unbold when the bold span is a complete sentence whose only job is to lead the bullet, and the list around it uses that same shape. Keep when the bold is a term the sentence then continues from:

- Field or property labels — `**Token URI**:`, `**App (engine):**`, `**Case A:**`, `**Phase A:**`, `**Q:**` / `**A:**`, `**Note:**`.
- Definition lists, where each bullet names and then defines a thing.
- Contrast pairs where only the keyword is bold (per the rule above).
- Actor names carrying the sentence's subject — `- **Cognito**, acting as the OAuth Client, consumes …`.
- Anchored labels that are also `<a id>` link targets, since inbound links render the label in bold too.

A lone short label sitting among full-sentence bullets goes with its list, not with its shape. It is playing the lead-in role there, so unbold it.

**Show the user the ones you cannot call, rather than guessing.** The recurring hard case is a label plus a bolded gloss — `- **X-Axis: What the child is (its class).** Which BaseAgent subclass …`. **The user's resolution, 3 times out of 3, was to shrink the bold to the term and let the gloss go plain:** `**X-Axis:**` / `**Y-Axis:**`, and `**Authentication** — signing in to …` / `**Authorization** — granting …`. Offer that middle option explicitly; do not present it as a binary between keep and unbold.

Sweep blockquote bullets (`> - **…**`) on the same rule. A top-level-only scan undercounts — in the Infra Guide it missed 25 of 162.

(Set 2026-09-21. The user authorized the last unswept range of the Infra Guide, §1–§10.8.1, with *"Don't blindly unbold everything … Use your judgement. For the ones that you cannot easily judge, show me and let me decide."* 87 came out, 77 stayed. ⭐ The clearest illustration of the list-is-the-unit rule: `**Cloud NAT.**` was kept because its list is a 3-term glossary, while `**Telemetry.**` was unbolded because its list is 4 full-sentence observations. ⭐ Unbolding a warning improved it — the bullet's surviving mid-sentence `**proxy VM's own**` is the counter-intuitive fact the note exists for, and the bold lead-in had been out-competing it by position.)

(Contrast-pair case and the added-bold scope bullet set 2026-09-20, during the `#135` Infra co-edit of §10.10. The user bolded `**Pre sign-up**` and `**Pre-token-generation**` on a bullet pair, inside the very range they had asked me to unbold hours earlier, and I flagged it as possible drift. The user: *"They are the keywords that contrast each other between the 2 bullets, and that's the purpose of those 2 bullets."*)

## Show a Repeated Action as Pseudocode

When an action repeats over a set, such as one grant per entity data store of each connector, show the loop as pseudocode in a `text` fence. Do not describe it in prose. A developer reads `for each … :` with a nested body at a glance. A prose version ("run the commands once per data store, and for a federated connector once for every …") hides the loop's shape and its count. A singular noun in prose ("the grant on a data store") can also read as a single action when the real action repeats.

- Fence: Use `text`, not a shell language, so no reader pastes it as runnable code.
- Body: Name each iteration's action in plain words, then say in a sentence after the block which real commands run each one.
- Precision: Keep the exact set in the loop header, such as "that the app attaches", not a looser set.

(Set 2026-10-07, Infra Guide §11.5.6. Claude's sentence "Grant it … with `RESOURCE` set to the data store" read as 1 call per connector. Its prose rewrite was still "not intuitive for a software developer", and the user asked for pseudocode.)

## Match a Copied Sample to the Doc's Spelling

When you copy a command or request body from a vendor page into a doc or script, rewrite its identifiers into the spelling the doc already uses. An API often accepts 2 spellings of a field, such as protobuf snake_case and JSON camelCase. A vendor sample may use the one your doc does not. A copied sample that still works will not flag the mismatch, so check it by eye. Test a converted call before you publish it.

(Set 2026-10-07, Infra Guide §11.5.2. Claude copied Google's snake_case `data_store_access_control_enabled` PATCH into a run sheet, the guide and `grant-connector-access.sh`, while every other mention used camelCase `dataStoreAccessControlEnabled`. The user asked why Claude had introduced the inconsistency.)

## A Linked Script Carries Its Own Usage

When a doc links to a script file instead of showing it, the doc carries only the link. Everything that explains the script goes in its header comment: what it does, how it maps to the doc's procedure, its behavior (e.g. read-modify-write), what it leaves in place, what it requires, which variables to set, which lines to edit, re-run safety, and options for partial runs. In the doc, a short list of links is enough, such as `- Grant: [link]` and `- Revoke: [link]`. Do not give each script its own subsection.

(Set 2026-10-06, Infra Guide §11.6.1. The user: *"you tried to explain how to run the script. Don't do that. Move any such explanations to the comment in the script files instead."* A follow-up the same day moved the remaining "what the script does" sentences too, and collapsed §11.6.1/§11.6.2 into a 2-bullet link list.)

## Write to the Reader's Altitude

Include a fact only if the target reader needs the concept at that point in the doc. The test is not "is it true" or "is it a real API" — it is "does *this reader* need this *here*." A fact can be accurate and still not belong: harness-internals and access-mechanism detail sit below the line of a reader who only writes application code, so they are cut even when correct. Name the reader for the passage (e.g. the `agent.py` author, the operator, the client developer), keep what that reader must know to use the thing, and drop what only its maintainer would.

This is why a concept primer omits the plumbing that *implements* the concept. Example — a Session/State primer for an app developer: keep `session`, `state`, and what the framework deposits there; cut the classes that drive the runtime (`Runner`, the session-service implementation) and the several code surfaces that reach state (`ToolContext.state`, `callback_context.state`, `invocation_context.session.state`). The reader never writes them, so naming one implies a precision it lacks and naming all is sprawl. This is distinct from concision, which trims wordiness: this rule decides inclusion by *whose need*, and it can cut a whole true subtopic, not just words.

## Goal and Solution Before Reference Material

Order a chapter so the reader reaches its payoff while their attention is fresh. Put the goal first, then the solution, then the runnable automation of the solution. Put heavy reference material after them: a terminology primer, a knob-by-knob explanation, raw per-setting reads. A reader who only wants the answer can then stop early. Link the solution's steps forward to the reference sections, and link the first use of each primer term to its definition.

A heavy primer placed first wears the reader out before the question is even stated. If a primer must come before something, open it with 1 line saying why it is worth the effort.

Keep a section as a numbered subsection, not a sidebar, when a solution step depends on it, or when it holds a table, a diagram, or mechanism. A sidebar holds a short conclusion or caveat.

(Set 2026-10-07, when the user reordered Infra Guide §11 to Goal → Solution → Automate → Primer → Knobs, arguing that a reader's attention is spent by the time they reach a solution placed after the long explanations.)

## Give a Heading Only to What Earns One

A heading puts an item in the Table of Contents and gives it a number. Give one only to an item a reader would look for or link to on its own.

- Section body: Do not leave a section's main content as a lead-in above a lone subsection. Move it into its own numbered subsection, so the section has 2 or more siblings.
- Diagram: A diagram that serves only the paragraphs around it gets no numbered heading. Put its caption inside the diagram, with a Mermaid `title:` frontmatter line.
- Callout: A caveat that serves 1 paragraph is a `**Note:**` with an `<a id>` anchor, not a sidebar.
- Sidebar: A sidebar that qualifies the section it sits in, at that section's end, is body text. Rewrite it as the section's closing paragraphs, or move it next to the rule it explains, and lead with its conclusion.
- Pair: 2 diagrams that show the same thing from 2 perspectives take the same label kind, such as System Diagram 16 and 17, not a System Diagram and a Figure.

Renumber the remaining items of each kind in document order after such a change, and retarget every inbound link.

(Set 2026-10-07, Infra Guide §11. The user flattened §11.2's lone §11.2.1, de-headed a 1-paragraph diagram, turned a sidebar into a Note, turned 2 sidebars into body text, and relabelled Figure 4 as System Diagram 17.)

## Name a Mechanism, Not One of Its Effects

Name a mechanism by what it is, not by one of the things it does. A name taken from 1 effect makes readers think that effect is the whole mechanism. Pair a generic noun such as "check", "filter", or "gate" with the thing that makes it specific, such as "`.get` IAM check".

(Set 2026-10-07. Infra Guide §11 called the `dataStoreAccessControlEnabled` IAM check "the filter", and the user read it as picker-list filtering, although it also withholds answer content. The user then rejected bare "check" as just as vague.)

## No Item Count in a List Lead-In

Do not put a count in the sentence that introduces a list, even when the list sits directly below it. Write `Notes:`, `Characteristics:`, `These settings must line up:` — never `3 settings must line up:` or `Four facts follow:`. The same ban covers a sentence after a list that counts back into it (`All 3 must hold`, `The 2 routes above`).

A count is a second place the list's length is recorded, and only one of the two gets updated when a bullet is added or removed. The count is then silently wrong, and nothing flags it. The lead-in earns nothing by carrying it, because the reader can see how many bullets there are.

Keeping the framing is a separate matter, and usually the real objection to a weak lead-in. `Notes:` does not tell the reader the bullets are prerequisites; `These settings must line up:` does, without a number. Fix framing by naming what the items *are*, never by counting them.

(Split out 2026-09-20 from **Forward-Reference Fragility** below, which had named the count trap only as an example of a sentence describing a *different* part of the doc. That framing did not fire for a lead-in sitting directly above its own list — the commonest case — and Claude argued to restore `3 settings must line up:` in an Infra Guide co-edit. The user: *"in the future, if we add/subtract a bullet, we'd have to change the stated count, too."*)

## Forward-Reference Fragility

Do not write a sentence in one part of a doc that hard-codes a fact about a *different* part of the doc — an item count ("Four facts follow:") or a description of what another section covers ("[§7.2.1] covers what the runtime cannot reach"). An edit to that other part later can silently make the sentence wrong, and nothing flags the break. Prefer a phrasing with no dependency on the other location's current state: a count-free lead-in ("Characteristics:" instead of "Four facts follow:"), or simply cutting a cross-section description that the reader can get by following the link instead.

This is the general form of the item-count trap already named in **List vs. Inline Enumeration** above (a stale "The 3 behaviors listed above" after a 4th bullet was added elsewhere) — that trap is one instance of this rule, not a separate concern. The same trap inside a single list's own lead-in has its own rule above, **No Item Count in a List Lead-In**.

(Set 2026-08-07, from a 2026-08-04 co-edit of the Infra Guide. The user rejected both "Four facts follow:" and "[§7.2.1] covers what the runtime cannot reach" on this reasoning, over my initial argument that both were more precise phrasings.)

## Text Trees: Hang a Child Near the End of Its Parent's Label

In a box-drawing text tree (`├──`, `└──`, `│`), column position is the only signal of nesting. Place each child's `│` or field block so it starts near the end of its parent's label. A child that starts at or before the start of its parent's label reads as the parent's sibling, even when it is a property of the parent. Judge the layout by where each child sits relative to its own parent's label. Equal column offsets across different parents do not matter, so do not flag unequal offsets as an inconsistency.

(Set 2026-10-05, from Infra Guide Figure 4. I pulled the `engines/…` field block left to the start of the node's name to save width, and the user pointed out that `appType` then read as a sibling of the engine. I then flagged a "14 columns vs 7 columns" offset difference, and the user said the absolute count does not matter.)

The same holds for any other layout choice inside a text figure, such as where a closing bracket goes or which value column a key is padded to. The user chooses each one for how it reads at that spot, not for uniformity across the figure. Do not flag 2 styles in 1 figure as an inconsistency, and do not offer to unify them. Flag a layout only when it misstates the structure: wrong nesting depth, a stray `│`, or a missing separator.

(Added 2026-10-05, after I flagged Figure 4's mix of inline closers like `}],` and own-line closers like `],` as "2 styles". The user: each style "is designed for better visual presentation at the right place".)

## Side-by-Side Code: Blank Lines Align the Compared Lines

When 2 code blocks sit side by side for comparison, such as the cells of a 2-column table, the user may give them different blank lines so the lines being compared sit on the same row. ⛔ Do not flag the uneven blank lines, and do not suggest making the 2 blocks match line for line. When you write such a pair yourself, pad with blank lines so the compared lines align.

(Set 2026-10-20, Infra Guide Sidebar 17. Claude suggested a blank line in column 2 to match column 1. The user had left it out on purpose, to line up `prompt_template_name` and `response_template_name` across the 2 cells.)

## Explain Code in Comments, Not in Prose Around the Block

When a doc shows a code block, put the explanation of what the code does inside the block as comments, next to the lines it explains. Do not write it as prose before or after the fence. This covers how the code works (read-modify-write), how to call an API (GET, not POST), what a value must look like, and what error appears when a step is skipped. A reader who copies the block takes the explanation along, and each comment sits next to the line it is about.

Keep prose outside the block only for what is not about the code itself, such as when to run it or why the doc needs it at that point. A section may then open directly on the code block.

This is the inline counterpart to **A Linked Script Carries Its Own Usage**.

(Set 2026-10-08, Infra Guide §11.5.2. The user had Claude delete 2 prose paragraphs before the grant commands, one about `RESOURCE` and one about `getIamPolicy`/`setIamPolicy` and the etag. Their content moved into the block as comments.)

**Moving prose into a comment is not a reason to keep it.** Before you move a sentence into the block, ask whether the code already says it. A comment that restates what the code's structure shows, such as a scope implied by the call it sits in, is noise. So is a framework internal the reader never acts on, such as how ADK forwards a config field. Drop such a sentence instead of moving it. Keep a comment only for what the code can't show: a constraint, a trap, a non-obvious value.

(Set 2026-10-19, Infra Guide §12.4.1. Claude moved "ADK copies that config into every model request" from a lead-in into a comment on `generate_content_config=`. The user: the snippet sits inside `root_agent = LlmAgent(...)`, so the comment explained "what is already obvious in the Python code". It was deleted.)

## Set Off a Code Comment and Its Lines With Blank Lines

When you add a comment inside a code block that explains specific lines, put a blank line before the comment and a blank line after the last line it covers. The blank lines make the comment and its lines one visible group. Without them, the reader cannot tell where the comment's scope ends, and it can read as a comment on the whole block or on the lines around it.

```bash
ROLE=roles/discoveryengine.agentspaceUser

# Sample values of RESOURCE, by resource type:
#   App:                  collections/default_collection/engines/my-app_1788743341079
#   Plain data store:     collections/default_collection/dataStores/my-datastore_1791248846914
RESOURCE=<RESOURCE>

URL=https://…/${RESOURCE}
```

Skip a blank line where the group already sits at the start or end of the block, or next to an existing blank line. This rule covers comments you add. Do not sweep existing comments in a doc unless the user asks.

(Set 2026-10-08, Infra Guide §11.5.2. Claude added a sample-values comment above `RESOURCE=` with no blank lines around the group. The user asked for them so the reader sees which line the comment is for.)

## Mark Every Omission Inside a Code or Capture Block

When you abridge anything inside a fenced block — omitted HTTP headers, dropped JSON fields, skipped code, truncated output — mark the cut **in the block** with a bare `...` line at the point of the omission. Saying "abridged" in the surrounding prose is not enough on its own. The reader studies the block, not the lead-in, and an unmarked cut silently misrepresents the block as complete.

Put the marker where the removed content was, at the indentation of its neighbors. A bare `...` on its own line is the form for a cut between lines (headers, statements, array elements); an inline `{ ... }` or `"parameters": { ... }` is the form for a collapsed object or argument list. Keep the surrounding prose's "abridged" note as well — the two work together, the prose explaining *why* and the marker showing *where*.

Redaction is a separate act from omission and gets its own marker, so do not collapse the two. Replacing a secret's value keeps the field visible and only hides what it held (`Bearer ya29.******<REDACTED, the parent runtime SA>******`), whereas `...` says a field or line is missing entirely. Match whatever redaction idiom the doc already uses, and preserve any non-secret prefix that carries meaning — a `ya29.` left in front of a redacted token still tells the reader what kind of credential it is.

## Fencing a Block Whose Bulk Is JSON

When JSON is the bulk of a code block, fence it as ```` ```json ```` — even when the block also holds lines that are not JSON. Pretty-printing and highlighting the part the reader studies is what matters. A `console`/`shell`/`text` fence renders the JSON flat and unhighlighted, which loses far more than it gains. The cost is that the few non-JSON lines highlight oddly, and that cost is accepted deliberately.

The rule is about proportion, not about the block being valid JSON. Apply it whatever the non-JSON lines are:

- A shell command followed by its JSON output — the 2 or 3 leading `$ curl …` lines.
- A server-sent-events line, where a `data: ` prefix sits in front of a pretty-printed JSON event. (Set 2026-09-20, Infra Guide §3.2.1. The user: *"I fenced it with 'json' rather than 'text' in order to render it better."*)
- Any comparable prefix, label, or framing line that introduces the JSON.

Never "correct" such a fence to `text` or to a shell language, and never report the language as a mismatch — it is an intentional choice about which part of the block gets good rendering. ⛔ Do not argue for `text` on the grounds that the block will not parse as JSON, and do not argue for it on the grounds that sibling blocks elsewhere in the doc use `text`. A sibling block that is a single unformatted line has nothing to gain from a `json` fence, so the two choices are consistent, not contradictory.

The user may also split a long URL value inside such a block across lines at its query parameters (`…/authorize?` then one `name=value&` per line), for readability. That is a deliberate presentation choice. Do not flag it as invalid JSON or as a copy-paste hazard, and do not propose rejoining it. (Set 2026-09-19, Infra Guide §10.5.2 `authorizationUri`.)

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


