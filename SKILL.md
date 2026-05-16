---
name: kmemo
description: >
  Generate structured Linux kernel reports for this knowledge base.
user-invocable: true
---

# kmemo

Generate a Linux kernel reports following this project's conventions.

## Project Overview

This is a documentation knowledge base covering Linux kernel subsystems, hardware architecture, and driver development. It is built with MkDocs (Material theme) and consists of Markdown articles organized by subsystem.

Content structure:

- `docs/` — all documentation articles
- `docs/templates/TEMPLATE-FULL.md` — full page template with all sections
- Major subsystem directories under `docs/`: `acpi/`, `arm64/`, `concurrency/`, `debug/`, `dp/`, `drivers/`, `drm/`, `mm/`, `pci/`, `pm/`, `sound/`, `usb/`, `usb4/`, `v4l2/`, `workflows/`, `xhci/`

## Input

`$ARGUMENTS` or conversation context provides:
- The subsystem (e.g., xHCI, PCIe, ACPI, USB4, DRM)
- The topic name (e.g., "host controller initialization", "MSI-X vectors")
- Optionally, an output directory override

If `$ARGUMENTS` is empty, derive the subsystem and topic from the conversation context.

## Procedure

### 1. Read the template

Before generating any content, read this file relative to `${CLAUDE_SKILL_DIR}`:

- `docs/templates/TEMPLATE-FULL.md` (page structure and section order)

### 2. Determine subsystem and output path

Look up the subsystem in the Subsystem Map (at the end of this file) to find:

- `tag`: the value for the `topics` and `tags` front matter fields
- `dir`: the output directory under `docs/`
- `kernel_paths`: directories in the kernel source tree to search first
- `spec`: specification name(s) for the SPECIFICATIONS section
- `section6_heading`: the heading to use for section 6 (REGISTERS, METHODS, PRIMITIVES, INTERFACES, or omit)

Construct the output path: `${CLAUDE_SKILL_DIR}/docs/<dir>/<topic-slug>.md`

If the output directory does not exist, create it.

### 3. Search local kernel source code

Search the local kernel source tree (not the web) for relevant code.

If the semcode MCP tools are available (e.g., `find_function`, `find_type`, `grep_functions`), prefer them as the primary search method:

- `find_function` to locate functions and macros by name or regex (returns file path and line number)
- `find_type` to locate structs, enums, and typedefs by name or regex
- `find_callers` / `find_calls` to understand direct call relationships
- `find_callchain` to trace multi-level call chains (useful for the DETAILS section)
- `grep_functions` to search inside function bodies for keywords or spec references
- `find_commit` with `symbol_patterns` or `path_patterns` to find commits that introduced or modified key symbols (commit messages often cite spec sections)
- `dig` on relevant commits to find lore.kernel.org mailing list discussion (useful for OTHER SOURCES and SPECIFICATIONS)
- Fall back to Grep and Glob for things semcode does not cover: standalone macros in headers, `Documentation/` files, Kconfig entries, and non-function code

If semcode tools are not available (e.g., the MCP server is not running), fall back entirely to Grep and Glob:

- Source files in `kernel_paths` relevant to the topic
- Function definitions (with line numbers) using patterns like `^(static\s+)?\w+.*\bfunction_name\b\s*\(`
- Struct and macro definitions
- Comments referencing specification sections
- Files under `Documentation/` related to the topic

Record exact file paths and line numbers for every function, struct, or macro found.

### 4. Construct Elixir cross referencer URLs

Use the base URL: `https://elixir.bootlin.com/linux/v6.19/source/`

For file references:
```
[`path/to/file.c`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.c)
```

For function references (include line number). The `\<...\>` word-boundary markers make these references compatible with `git log -L`:
```
[`'\<function_name\>':'path/to/file.c'`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.c#L1234)
```

For kernel documentation files:
```
[`Documentation/subsystem/file.rst`](https://elixir.bootlin.com/linux/v6.19/source/Documentation/subsystem/file.rst): brief description
```

### 5. Identify specifications

Check source code comments and headers for references to specification chapters and sections. Map the subsystem to its known specifications using the `spec` field from the Subsystem Map.

If semcode tools are available, supplement source code comments with:

- `find_commit` with `symbol_patterns` for key functions/types: commit messages frequently cite spec sections
- `dig` on the commits that introduced the relevant code: the associated mailing list threads often reference specific spec chapters and provide review discussion suitable for OTHER SOURCES
- `grep_functions` with patterns like `section|chapter|spec|table` to find spec references embedded in function bodies or comments

Format each entry as: `<spec name>, section <N.N>: <section title>`

If no specification applies, leave the SPECIFICATIONS section present but empty.

### 5b. Find usage examples for LINUX KERNEL symbols

For every function, struct, macro, or enum listed in the LINUX KERNEL section, search for at least one concrete usage example in the kernel source before writing the DETAILS section. These examples show where and how the symbol is actually used in practice.

If semcode MCP tools are available:

- `find_callers` to find functions that call a given function or macro
- `find_callchain` to trace how a function fits into a larger call sequence
- `grep_functions` to find code that references a struct, enum, or macro inside function bodies

If semcode is not available, use Grep to search for usages:

- For functions: search for call sites (e.g., `\bfunction_name\(`)
- For structs: search for variable declarations or field accesses (e.g., `struct struct_name` or `->field_name`)
- For macros: search for invocations (e.g., `\bMACRO_NAME\(` or `\bMACRO_NAME\b`)
- For enums: search for usage of enum values

Record the caller/user function name, file path, and line number for each example found.

When writing the DETAILS section, incorporate these usage examples to show the symbol in context. For example, if `xhci_mem_init` is listed in LINUX KERNEL, the DETAILS section should show where it is called from (e.g., called by `xhci_init` during probe), what arguments it receives, and what it does with the structs and macros also listed in LINUX KERNEL.

### 6. Generate the page

Follow the template structure exactly. The page must contain these sections in order:

1. YAML front matter with `topics` and `tags` (include `"verification-needed"` tag)
2. H1: the topic name (just the name, no extra text)
3. A short summary paragraph with an ASCII diagram if appropriate
4. `## SUMMARY`
5. `## SPECIFICATIONS`
6. `## LINUX KERNEL`
7. `## KERNEL DOCUMENTATION`
8. `## OTHER SOURCES`
9. `## <section6_heading>` (from Subsystem Map; omit entirely if set to "none")
10. `## DETAILS`

### 7. Writing rules (mandatory)

All generated content must follow these rules:

- No em-dashes. Use parentheses instead: "CC (Command Completed)" not "CC --- Command Completed"
- No boldface (`**...**`)
- No negative constructions. Write "It is synchronous" not "It is synchronous, not asynchronous"
- No question-style or "Why X does Y" / "How X works" / "Where X happens" framings as H3 or H4 headings in DETAILS, SUMMARY, or any body section. Write declarative statements. The H3 catalog labels in LINUX KERNEL (e.g., `### Producer: gfp_to_alloc_flags`, `### Consumer: rmqueue_buddy`) are fine and should be kept; this rule only forbids question/explanation framings.
  - BAD: `### Why MIGRATE_RECLAIMABLE does not get ALLOC_CMA`
  - GOOD: `### MIGRATE_RECLAIMABLE allocations skip ALLOC_CMA`
  - BAD: `### How the slowpath wires it in`
  - GOOD: `### Slowpath wiring through retry`
  - BAD: `### Why no slowpath`
  - GOOD: `### Lock-free path skips the slowpath`

### 7a. Prose colon idioms (mandatory)

Body prose (everything outside H1, H2, H3, H4 headings, fenced code blocks, ASCII diagrams, list bullets, table cells, YAML frontmatter, and Elixir links) must never use the "label-colon-explanation" idiom. The colon-followed-by-clause pattern in prose is banned. State the same content as a plain declarative sentence.

This applies to forms like:

- "X: Y." where X is a noun phrase and Y is the explanation. BAD: `Two-phase pattern: a non-atomic test, then an atomic clear.` GOOD: `There are two phases in the usage pattern. A non-atomic test runs first, and an atomic clear only follows when the test was true.`
- "X is Y: Z." BAD: `The asymmetry: a regular allocation sees nr_free_highatomic subtracted.` GOOD: `A regular allocation sees nr_free_highatomic subtracted.`
- "X is the key: Y" / "X is essential: Y" / "X is explicit: Y" / "X is significant: Y" / "X is conservative: Y" / "X is deliberate: Y" / "X is the linchpin: Y" / "X is asymmetric: Y" / "X is intentional: Y" / "X is correct: Y" / "X becomes clear here: Y". BAD: `The check is essential: the wake-up is what makes the boost recoverable.` GOOD: `The check matters because the wake-up is what makes the boost recoverable.`
- "The intent: Y" / "The reasoning: Y" / "The result: Y" / "The fix: Y" / "The condition is: Y" / "The order of operations matters: Y" / "The pattern is: Y" / "The point is: Y" / "The takeaway is: Y". BAD: `The reasoning: CMA's contiguous-memory contract requires migration.` GOOD: `CMA's contiguous-memory contract requires migration.`
- "X says: <quote>" / "X makes Y explicit: <quote>" / "X spells this out: <quote>" / "Comment: <quote>" introducing direct quotes. BAD: `The comment "Ensure kswapd doesn't accidentally go to sleep as long as we loop" is the key: an allocator spinning on the slowpath needs kswapd.` GOOD: `According to the comment "Ensure kswapd doesn't accidentally go to sleep as long as we loop", an allocator spinning on the slowpath needs kswapd.`
- "X is called from N places: A, B, C." Replace with "X is called from N places. A does ..., B does ..., C does ...". The list-after-colon shape is banned even when the items are short.

Never editorialise with "The reasoning:" or any synonym ("The rationale is", "The motivation:", etc.) that asserts authorial reasoning. The page describes what the code does; if a comment or commit message states a rationale, quote it via "According to the comment <quote>, ..." instead.

The colon is acceptable inside H3/H4 headings (catalog labels like `### Producer: gfp_to_alloc_flags`), inside YAML frontmatter, inside Elixir link titles, inside code blocks, inside URLs, inside ratios (`M:N`), and after Markdown list bullets when the item is a catalog entry in the LINUX KERNEL or KERNEL DOCUMENTATION section. It is banned in flowing prose paragraphs and in the lead summary paragraph.

### 7b. Prose lists (mandatory)

Body prose in DETAILS, SUMMARY, and the lead summary paragraph must not use the "intro sentence + list" pattern when the list is explanatory. Fold the items into a single flowing paragraph.

- BAD:

  ```
  Two details deserve attention.
  
  - reclaim_order is max(order, pageblock_order) when defrag_mode is set.
  - The last_pgdat deduplication skips zones that share a pgdat.
  ```

- GOOD:

  ```
  reclaim_order is max(order, pageblock_order) when defrag_mode is set, which tells kswapd to reclaim at pageblock granularity. The last_pgdat deduplication skips zones that share a pgdat, since there is one kswapd thread per pgdat.
  ```

The forbidden shape is "<noun phrase ending in a period or colon> + <bullet/numbered list>" used as exposition. Phrases that head such lists ("Two notable details.", "Three layers stack.", "Four cases run from strongest to weakest.", "Concrete uses.", "Five upfront refusals.") are banned even with a period. Restate as a paragraph.

The H3 catalog lists in LINUX KERNEL (Producers, Consumers, GFP mapping, Watermarks and reserves, Migratetype and freelists, Cpuset interaction, Concurrency, Related types, Tracepoints, Flag definition) and the bullet lists in KERNEL DOCUMENTATION and OTHER SOURCES are reference catalogs and remain as lists. Tables remain as tables. This rule applies only to prose-explanation lists, not to reference catalogs.

### 7c. Forbidden phrases checklist

Before writing any body paragraph, scan for these patterns and rewrite if any appear:

- `^.*: [a-z]` (any line where prose ends in `: ` followed by a lowercase clause)
- `The reasoning` (in any case, with or without colon)
- `The intent:` / `The asymmetry:` / `The fix:` / `The point is:` / `The takeaway:` / `The pattern is:` / `Two-phase pattern:`
- `is the key:` / `is essential:` / `is explicit:` / `is significant:` / `is conservative:` / `is deliberate:` / `is the linchpin:` / `is asymmetric:` / `is intentional:` / `is correct:` / `becomes clear here:`
- `Comment: "` introducing a quote in prose (different from the LINUX KERNEL bullet form `[symbol]: bit 0xN. Comment: "..."` which is a catalog entry and acceptable)
- `says: "` / `spells this out: "` / `makes explicit: "` / `makes the trade-off explicit: "` introducing a direct quote in prose
- `X is called from N places: A, B, C` (intro-colon list)
- Any `"intro sentence." + bullet/numbered list` shape in DETAILS, SUMMARY, or lead summary paragraphs

If any of these appear in body prose, rewrite the paragraph as plain declarative sentences. Quote comments with "According to the comment <quote>, ..." or "The comment reads <quote>." instead of label-colon framing.

### 7d. Hollow superlatives and unsupported adjectives (mandatory)

Never characterize a kernel construct with a ranking adjective unless the same sentence (or the next one) names the concrete mechanic that justifies the ranking. Each kernel symbol, mode, or path is unique by definition; saying it is "the most X" or "the least Y" or "the strongest Z" without explaining the comparison adds zero information and is banned.

Banned phrasings (when not immediately followed by the supporting mechanic):

- "the most invasive" / "the most fragmenting" / "the most aggressive" / "the most consequential" / "the most preferred" / "the least preferred" / "the most expensive" / "the cheapest"
- "the cheap path" / "the slow path" / "the fast path" used as standalone characterization (use only when "fast" or "slow" is a defined kernel term, e.g. "fast path" of a specific lock implementation)
- "the strongest guarantee" / "the weakest guarantee" / "the strongest anti-fragmentation guarantee"
- "the worst outcome" / "the best outcome"
- "the entire performance benefit" / "the entire correctness benefit"
- "the key invariant" / "the key difference" / "the key innovation" / "the key role" / "the design assumption" / "the design intent"
- "the only mode that ..." (when the same is trivially true of every other mode under some other framing)
- "elaborate", "elegant", "fundamental", "cornerstone", "linchpin", "crucial", "critical" used as standalone characterizations

Acceptable forms:

- BAD: "RMQUEUE_CLAIM is the most invasive non-stealing fallback."
- GOOD: "RMQUEUE_CLAIM mutates the migratetype of one or more pageblocks. It calls set_pageblock_migratetype() to flip the pageblock_flags entry, and ... [exact mechanic]."
- BAD: "RMQUEUE_NORMAL is the cheap path through __rmqueue()."
- GOOD: "RMQUEUE_NORMAL calls __rmqueue_smallest() on the requested migratetype, which walks free_area[order..NR_PAGE_ORDERS-1] and never inspects other migratetypes' freelists."
- BAD: "This is the strongest anti-fragmentation guarantee."
- GOOD: "This restricts __rmqueue_claim() to converting pageblocks that are already entirely free, so set_pageblock_migratetype() never produces a pageblock with a recorded migratetype that disagrees with any of its in-use pages."
- BAD: "the key difference from RMQUEUE_CLAIM"
- GOOD: "RMQUEUE_CLAIM passes start_type to its tail call after set_pageblock_migratetype() flips the pageblock; RMQUEUE_STEAL passes fallback_mt and never calls set_pageblock_migratetype()."

Test for any adjective in body prose: ask "would the sentence still convey the mechanic if I deleted this adjective?" If yes, delete it. If no, replace the adjective with the actual mechanic. Hollow superlatives that cannot be reduced to a concrete code-level fact must not appear in body prose at all.

The two legitimate exceptions are direct quotes from kernel source comments and direct quotes from commit messages or LKML threads, which are reproduced verbatim even when they contain superlatives the rule would otherwise forbid.

### 7e. Self-contained kernel-source citation (mandatory)

Every page must read as a self-contained source. A reader who never opens the kernel tree must still finish the page knowing exactly what the relevant code does. Whenever the page explains how a function works, what a struct looks like, how a macro is used, or how a call site invokes a callee, the actual code goes inline as a fenced ` ```c ` block before or alongside the explanation. Linking to Elixir is not a substitute for showing the code; the link is for navigation, the code block is for comprehension.

Concrete requirements:

- For every function listed in LINUX KERNEL, the DETAILS section must contain at least one fenced ` ```c ` block showing either its full body (when it is small) or the body of the case label / branch / inner block that the page is actually describing. Do not describe a function's behavior in prose alone when the body would fit in a screen of code.
- For every struct or enum listed in LINUX KERNEL, the DETAILS section must contain a fenced ` ```c ` block reproducing the type definition (including comments and `#ifdef` regions). The reader must see the exact field list and any decorative comments without leaving the page.
- For every macro or static array (e.g. `fallbacks[][]`, `__used` lookup tables) referenced in body prose, reproduce the definition as a fenced block at the point where the prose first depends on it.
- When walking a call chain, show the caller's invocation site as a code block as well as (separately) the callee's body. The reader has to see both ends of the call, not just one.
- When explaining a switch statement, conditional, or loop whose structure is the point of the explanation, the code block must reproduce that structure verbatim. Paraphrasing the control flow in prose is forbidden when the actual code would convey it more directly.
- When citing a kernel comment, quote the comment text inside the same fenced code block that contains the surrounding code, and refer to it via "According to the comment <quote>, ..." in the prose.
- When citing a commit message that contains a benchmark table, ASCII figure, or other formatted text, reproduce it inside a fenced code block (use ` ``` ` without a language hint) so the formatting survives.

Each fenced code block stays as close to the kernel source as practical: tab indentation preserved, all original comments retained, no truncation other than `...` to elide irrelevant intermediate code (and only when the elided code would not change the reader's understanding). When a function body is too long to reproduce in full, split it across multiple code blocks at natural boundaries (one per case label, one per loop, one per error-handling tail) rather than truncating, and explain each block in the prose between them.

The test for whether enough code has been cited: assume the reader has the page open in one window with no other terminals, no other browser tabs, and no kernel tree. Could they still describe in their own words exactly which lines run on the path the page is documenting? If not, more code blocks are needed. Adding a sentence "see [`func()`](https://elixir...)" does not count as showing the code; the link is for the reader who wants to verify or explore further, not for understanding the page.

The DETAILS section is the canonical place for this. SUMMARY may include short snippets when a single line of code is the cleanest way to convey the topic, but bulk code citation belongs in DETAILS, interleaved with the prose that walks through it.
- H1 is always the topic name only
- Every generated or modified page gets the `"verification-needed"` tag (at most one instance)
- Do not add any tags other than `"verification-needed"`
- `Documentation/` references go in KERNEL DOCUMENTATION, never in OTHER SOURCES
- If an existing page has `Documentation/` links in OTHER SOURCES (or using `docs.kernel.org` / `kernel.org/doc` URLs), move them to KERNEL DOCUMENTATION. Do not convert existing URLs; instead, add a new Elixir cross referencer reference entry pointing to the same in-tree kernel doc file.
- No hard line wrapping in prose. Each paragraph of prose text must be a single long line, with line breaks only between paragraphs. Do not wrap lines at 80 or any other column width. Code blocks (between ` ``` ` markers), ASCII diagrams (indented lines), list items, and table rows are exempt from this rule.
- Every mention of a kernel symbol (function, macro, struct, enum, typedef) must be an Elixir cross referencer link. No exceptions. This applies to every inline code span (`` ` `` ... `` ` ``) in every section of the page: SUMMARY, LINUX KERNEL, INTERFACES, DETAILS, and prose paragraphs. This includes inline code with arguments such as `` `func(arg1, arg2)` `` in INTERFACES sections. Write [`function_name()`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.c#L123) instead of bare `function_name()`. Write [`func(arg1, arg2)`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.c#L123) instead of bare `func(arg1, arg2)`. Write [`struct foo`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.h#L45) instead of bare `struct foo`. Write [`MACRO_NAME`](https://elixir.bootlin.com/linux/v6.19/source/path/to/file.h#L78) instead of bare `MACRO_NAME`. The only place bare symbol names are acceptable is inside fenced code blocks (` ``` `) that show code snippets or struct definitions. If a symbol appears multiple times on the same page, every occurrence outside a code block must be linked (repeat the link). If you cannot determine the file and line number for a symbol, look it up before writing it. If it truly cannot be found in the kernel source (e.g., it is a spec-defined ACPI method name like `_PS0` or a hardware register name like `SLP_EN`), it may remain unlinked, but add a comment noting it is a spec/hardware name.
- When referencing a struct or enum type, always include the `struct` or `enum` keyword (e.g., `struct drm_format_info`, `enum drm_color_encoding`). Do not omit the keyword unless the type is a typedef. This applies everywhere: LINUX KERNEL entries, SUMMARY, INTERFACES, DETAILS, and inline prose. For LINUX KERNEL section entries using the `'\<...\>'` format, the keyword goes inside the angle brackets: `'\<struct drm_color_lut\>'`, `'\<enum drm_colorop_type\>'`.
- Do not reference internal pages. Do not add cross-links to other pages in the knowledge base (e.g., `[Page Title](other-page.md)`). Each page must be self-contained.
- When citing kernel source code in Markdown code blocks, preserve the exact indentation style from the kernel source. The kernel uses tabs (8-space width) for indentation. Do not convert tabs to spaces. This includes function bodies, switch/case statements, and multi-line expressions.
- Use markdown link format `[Title](URL)` for all entries in the OTHER SOURCES section. Do not use bare URLs or `Title — URL` style.
- The DETAILS section must include detailed kernel code walkthroughs: step-by-step traces through function call chains, real driver API usage examples, and lifecycle coverage for key objects. For every function/struct/enum in the LINUX KERNEL section, find at least one concrete driver usage and show it in DETAILS. When elaborating on kernel code paths, always cite the actual implementation as fenced Markdown code blocks (` ```c `) rather than only describing it in prose. Show the relevant code, then explain it.

### 7f. ASCII diagrams (mandatory)

Only include an ASCII diagram when it conveys a spatial or temporal relationship that prose cannot express efficiently. A diagram earns its place when it shows physical layout, parallel structure across multiple lanes, a non-linear graph, an address space, a bit field, a ring/queue with head and tail pointers, or two views of the same data side by side. Concrete examples that justify a diagram include a btrfs extent map showing compressed extents alongside uncompressed extents at matching file offsets, the buddy allocator's per-order freelist columns, a doorbell BAR partitioned across IPs, or a tree of devices with parent/child arrows.

Do not draw a diagram for a simple linear sequence of function calls, a top-down call chain, a state machine with two states, or any flow that reads naturally as a paragraph or as a fenced code block of pseudocode. "Function A calls B which calls C" is prose, not a diagram. A single arrow chain in a box is not a diagram. If a reader would understand the same content faster from one sentence of declarative prose, write the sentence and delete the diagram.

When a diagram is used, follow the style established in the `docs/mm/` pages. Use Unicode box-drawing characters (`┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼`) and `▼ ▲ ◀ ▶` for arrows. Title each sub-diagram with a short heading underlined by a `────` rule. Multiple sub-diagrams may share one fenced block when each has its own titled section. Indent the whole figure 4 spaces inside the fenced block so it reads as a figure, not as text. Keep every line under 80 columns so the figure renders without wrapping in plain-text views.

Pure ASCII `\`, `/`, and `|` are never used as box-drawing or connector characters. The `/` and `|` characters are acceptable inside the figure only as English word separators ("ROOT_PORT / DOWNSTREAM"), as C bitwise expressions (`LBMS | LABS`), or inside reproduced kernel source. All box sides, corners, junctions, and arrows are Unicode.

Diagram annotations (legends, per-bit meanings, code-like pseudocode lines, comments below the figure) live inside the same fenced block as the figure. The forbidden-phrase rules from 7a/7c/7d do not apply to text inside fenced code blocks, including ASCII figure blocks, but the prose surrounding the figure outside the fence still does.

#### Pattern catalog

When a diagram is justified, prefer one of the named patterns below. Each pattern has a use case and a shape; copying the shape and substituting names is usually enough to produce a clean figure. Reach for a new shape only when none of these fits the spatial relationship in question.

##### Pattern: parent + N children fan-out

Use when one parent object spawns multiple typed child objects on a different bus / queue / map / list, and the children's identity comes from a field inside the parent. Draw the parent as a wide top box with field-level content, then N children in a row underneath, joined by a single trunk that splits into N branches.

```
       struct parent (on bus_A)
       ┌──────────────────────────────────────────────────────────┐
       │  field_1  ...                                            │
       │  field_N  ...                                            │
       └──────────────────────────┬───────────────────────────────┘
                                  │ allocation / registration
              ┌──────────┬────────┼────────┬──────────┐
              ▼          ▼        ▼        ▼          ▼
         ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
         │ child0 │ │ child1 │ │ child2 │ │ child3 │ │ child4 │
         └────────┘ └────────┘ └────────┘ └────────┘ └────────┘
```

##### Pattern: sparse slot map with conditional backing

Use when an address space, slot table, or fixed-size index set is divided into uniformly-sized regions and each region may or may not have a backing data structure allocated for it. The figure shows which regions are present (have backing) and which are holes (no backing), with the lookup formulas below. Reach for it when the page needs to convey that the lookup is a direct dereference but the backing array is sparse and only allocated for populated slots.

Draw the slot row as a contiguous strip of N cells joined by `┬` dividers (no gaps between cells, since the slots are contiguous in the index space). Each cell contains a short status word (`present`, `hole`, `valid`, etc.). The bottom border has `┬` connectors only beneath populated slots (the ones that will descend to a backing box). For each populated slot, drop a `▼` arrow from the bottom of the strip to a backing object box drawn directly below; leave the hole slots with no arrow and no box below. Add accessor labels beneath each backing box (the name and field expression by which code reaches that backing object), connected by `▲` arrows pointing up into the backing box. Labels span multiple lines if the field path is long.

Close the figure with a one-line parenthetical noting which slots are skipped, then one or more formula blocks showing the lookup formula for each conversion mode (direct lookup, flat lookup, etc.). Each formula block has a short heading followed by indented pseudocode lines.

This pattern differs from `parent + N children fan-out` (one parent allocating all N children via a single trunk) and from `N-to-M source/destination mapping` (disjoint inputs feeding rows of a tabular destination): here a single uniform index space partitions into independent slots, each independently populated or absent.

```
       Sparse slot map (possibly with holes)
       ─────────────────────────────────────

       Slot 0        Slot 1        Slot 2        Slot 3
       ┌─────────────┬─────────────┬─────────────┬─────────────┐
       │   present   │    hole     │   present   │   present   │
       └──────┬──────┴─────────────┴──────┬──────┴──────┬──────┘
              │                           │             │
              ▼                           ▼             ▼
        ┌───────────┐               ┌───────────┐ ┌───────────┐
        │  backing  │               │  backing  │ │  backing  │
        │  object   │               │  object   │ │  object   │
        └───────────┘               └───────────┘ └───────────┘
              ▲                           ▲             ▲
              │                           │             │
        table[0]                    table[2]      table[3]
        .ptr                        .ptr          .ptr

       (Slot 1 has no backing object; the hole is skipped)

       Direct lookup:
         lookup(idx) = table[idx].ptr + intra_slot_offset

       Flat lookup:
         lookup(idx) = base + idx
         (base is a virtually contiguous array; only populated slots are mapped)
```

##### Pattern: truth table (input bits → output)

Use when a function's return value or the chosen branch is a deterministic function of a small number of input bits. Lay out the inputs as boxed columns on the left and the action / return on the right; one row per distinct input pattern. The handler's sequential flow (read, clear, return) lives in prose outside the diagram, not as additional arrows inside it.

```
       input_a   input_b   result
       ┌───────┬─────────┬───────────────────┐
       │  0    │   X     │ OUTCOME_NONE      │
       │  1    │   0     │ OUTCOME_HANDLED   │
       │  1    │   1     │ OUTCOME_WAKE ──▶  followup_handler
       └───────┴─────────┴───────────────────┘
```

##### Pattern: boxed flowchart with decision nodes

Use when a function has 3+ sequential decision points with side effects and back-edges, and showing each step in its own box adds clarity. Each step gets its own box; each decision node has explicit yes / no labels on outgoing edges; loops draw an explicit back-edge with an arrow. Reserve this for paths with real branching; a 2-state decision should be written as prose instead.

```
       ┌─────────────────┐
       │ acquire lock    │
       └────────┬────────┘
                │
       ┌────────▼────────┐    yes
       │ early-exit cond?│──────────▶  break
       └────────┬────────┘
                │ no
                ▼
       ┌─────────────────┐
       │ read register   │
       └────────┬────────┘
                │
       ┌────────▼────────┐    yes ┌──────────────┐
       │ event present?  │──────▶ │ handle event,│
       └────────┬────────┘        │ continue ────┼── back-edge to top
                │ no              └──────────────┘
                ▼
              break

       break ─▶ release lock, return
```

##### Pattern: side-by-side struct comparison

Use when two related types interact via a third operation (match function, encode / decode pair, pack / unpack helpers). Show both struct definitions as boxes side by side with the operation drawn underneath as the convergence point.

```
       struct lhs                     struct rhs
       ┌─────────────────┐            ┌─────────────────┐
       │ field_x         │            │ field_x         │
       │ field_y         │            │ field_y         │
       │ ...             │            │ ...             │
       └────────┬────────┘            └────────┬────────┘
                │                              │
                └────────► matcher / op ◀──────┘
                                │
                                ▼
                       returns match iff
                         lhs.field_x == rhs.field_x
                         AND (rhs.field_y == ANY || ...)
```

##### Pattern: linked structs via field-level pointers

Use when several related structs and arrays connect via pointer fields, encoded pointers, or per-cell bitmap pointers, and the relationship between them is the field-level pointer topology itself. Each struct is drawn as a labeled box: the struct name (with `struct` keyword) sits on a borderless heading line above the box, and field names are listed inside the box one per line with optional type or comment in parentheses. Fields with internal structure (bit-packed words, embedded arrays, fixed bitmaps) get drawn as nested cell strips inside the outer box.

Pointer fields exit the bottom border of the box (via `┼` or `─┐`), descend as vertical trunks (`│`), and terminate in a `▼` arrow that lands on the target box below. When a nested bitmap has per-cell pointing relationships (one bit per subsection, one entry per slot), each cell descends via its own vertical trunk and `▼` to its target in a parallel array drawn underneath. Length-bracket annotations of the form `|<── span ──>|` mark total spans beneath arrays. Close the figure with a legend block beneath the diagram listing flag and constant meanings as `NAME = MEANING` columns.

This pattern differs from `parent + N children fan-out` (which shows allocation or registration of typed children) and from `side-by-side struct comparison` (which shows two peer structs meeting at one operation): here the structs already exist and the visible shape of the figure is the chain of pointer fields linking them together.

```
       Linked struct hierarchy
       ───────────────────────

       struct outer_t
       ┌──────────────────────────────────────────────────────────┐
       │  packed_field  (unsigned long, encoded)                  │
       │  ┌──────────────────────────────────┬───┬───┬───┬───┐    │
       │  │   encoded target * pointer       │ D │ C │ B │ A │    │
       │  └──────────────────┬───────────────┴───┴───┴───┴───┘    │
       │                     │                                    │
       │  link ──┐           │  (biased: pointer + idx = target)  │
       │         │           │                                    │
       │  optional_field  (only with CONFIG_OPTIONAL)             │
       └─────────┼───────────┼────────────────────────────────────┘
                 │           │
                 │           ▼
                 │      struct target_t[N]
                 │      ┌────┬────┬────┬────┬─────┬─────┐
                 │      │ t0 │ t1 │ t2 │ t3 │ ... │tN-1 │
                 │      └────┴────┴────┴────┴─────┴─────┘
                 │      |<── one entry per index in outer ──>|
                 │
                 ▼
       struct linked_t
       ┌──────────────────────────────────────────────────────────┐
       │  preamble                                                │
       │                                                          │
       │  bitmap[BITS_TO_LONGS(N_CELLS)]                          │
       │       bit:   0    1    2    3    4   ...   N-1           │
       │       ┌────┬────┬────┬────┬────┬─────┬────┐              │
       │       │ 1  │ 0  │ 1  │ 1  │ 0  │ ... │ 1  │              │
       │       └─┬──┴─┬──┴─┬──┴─┬──┴─┬──┴─────┴─┬──┘              │
       │         │    │    │    │    │          │                 │
       │  trailer  (variable-length, computed by helper)          │
       └─────────┼────┼────┼────┼────┼──────────┼─────────────────┘
                 │    │    │    │    │          │
                 ▼    ▼    ▼    ▼    ▼          ▼
                 ┌────┬────┬────┬────┬────┬────┬─────┐
                 │ U0 │ U1 │ U2 │ U3 │ U4 │ ...│UN-1 │  fixed-size units
                 └────┴────┴────┴────┴────┴────┴─────┘  one bit per unit
                 |<──── total span of one linked_t ──────────>|

       Flag bits in packed_field (low bits):
         A = FLAG_NAME_A  (bit 0, marker)
         B = FLAG_NAME_B  (bit 1, has-pointer)
         C = FLAG_NAME_C  (bit 2, online)
         D = FLAG_NAME_D  (bit 3, early-init)
```

##### Pattern: bit field with per-bit legend

Use when a small register, mask, or enum needs to show field positions and per-bit / per-range conditions. Draw the bit cells as a single row of boxes with bit numbers above and field names inside; list each named field below with its condition or meaning. Multi-bit fields stretch across multiple bit positions in one cell.

```
       bit:    7      6:4         3       2:0
              ┌─────┬───────────┬───────┬───────────┐
              │ RW1C│   field_b │ field │ field_a   │
              │ INT │           │  c    │           │
              └─────┴───────────┴───────┴───────────┘

       INT     = MACRO_NAME_X   (RW1C)
       field_b = MACRO_NAME_Y   (R/O)
                  0b00 = meaning0
                  0b01 = meaning1
                  ...
       field_a = MACRO_NAME_Z   (RW1C; cleared by the handler)
```

##### Pattern: compound packed field (encoded pointer with status flags)

Use when a single struct field is a packed `unsigned long` (or similar word) that combines an encoded pointer to another struct with multiple status flag bits in the low bits, and the page needs to show both halves at once with the decode formula visible. This is common when the kernel reuses alignment-guaranteed low bits of a pointer to encode metadata; the figure shows the bit positions, the per-bit flag constants, and the formula that extracts the embedded pointer.

Lay out the bit positions above one row of cells using the form `63   ...   N   ...   3   2   1   0`. The cell row uses a wide range cell on the left (the encoded pointer or slot index), optional intermediate range cells (NID, type), and single-character cells on the right (one per flag bit). The top and bottom borders may visually group the rightmost flag cells under one widened `─` segment so the box edge stays compact, while the content row still shows each flag bit in its own `│ X │` cell. The total width of the top border, content row, and bottom border must match.

Below the cell row, drop vertical trunks (`│`) from each flag cell's right divider (the `│` to the right of the cell in the content row, not the cell character itself). Connect each trunk to its constant name with an L-shaped corner (`────┘`); the constant labels stack as a left-aligned column on the left and the dashes lengthen from line to line so each elbow lands on its trunk. The leftmost (highest-numbered) flag's trunk gets the shortest dashed line; the rightmost (lowest-numbered) flag's trunk gets the longest.

Close the figure with a multi-line pseudocode block showing the decode formula (`Pointer = field & FIELD_PTR_MASK`, `= real_pointer - base_index(slot)`) and a parenthetical note explaining any bias or invariant.

This pattern differs from `bit field with per-bit legend`: that pattern uses below-table `KEY = VALUE` textual lists to describe each bit or range; this one uses visual L-connector trunks linking each flag-bit cell to its constant name, plus an explicit decode formula. Reach for this variant when the packed field is the entry point into another struct (pointer encoding), not when the field is purely a status register.

```
       struct outer_t.packed_field bit layout (after initialization)
       ─────────────────────────────────────────────────────────────

       63                                    N     ...    3   2   1   0
       ┌────────────────────────────────────┬─────┬───┬───┬───┬───────┐
       │  encoded struct target * pointer   │ NID │...│ D │ C │ B │ A │
       └────────────────────────────────────┴─────┴───┴───┴───┴───────┘
                                                        │   │   │   │
                             FLAG_NAME_D ─────────────────┘   │   │   │
                             FLAG_NAME_C ─────────────────────┘   │   │
                             FLAG_NAME_B ─────────────────────────┘   │
                             FLAG_NAME_A ─────────────────────────────┘

       Pointer = packed_field & FIELD_PTR_MASK
               = real_pointer - base_index(slot)
       (biased so that pointer + idx yields the correct struct target)
```

##### Pattern: N-to-M source/destination mapping

Use when several disjoint inputs feed a single tabular destination, with some inputs feeding multiple rows of the destination. Sources go in a column on the left; the destination is a stacked box on the right; arrows cross the gap. Annotation that would otherwise hang off the right edge of the destination box belongs as prose below the figure, not as right-aligned brackets, so the figure stays under 80 columns.

```
       Source register              Destination slots
       ───────────────              ─────────────────

       SRC_FIELD_A                  ┌─────────────────────┐
         file path / spec § ──▶     │ slot[0] = result_A  │
                                    │ slot[2] = result_A  │
                                    │ slot[4] = result_A  │
                                    ├─────────────────────┤
       SRC_FIELD_B          ──▶     │ slot[1] = result_B  │
         file path / spec §         ├─────────────────────┤
       SRC_FIELD_C          ──▶     │ slot[3] = result_C  │
         file path / spec §         └─────────────────────┘
```

##### Pattern: queue / ring between two stages

Use when a producer and a consumer communicate through a bounded buffer (kfifo, work_struct, list_head ring). Show the buffer as a row of cells in the middle; the two stages flank it; arrows label the put / get operations.

```
       Producer side              kfifo / work / ring         Consumer side
       ┌──────────────────┐       ┌──┬──┬──┬─────┬───┐        ┌──────────────┐
       │ Stage A reads    │       │e0│e1│e2│ ... │e_n│        │ Stage B      │
       │ source, RW1C     │  put  └──┴──┴──┴─────┴───┘  get   │ dequeues,    │
       │ clear, enqueue   │ ──▶      ▲              │   ──▶   │ processes    │
       │ ... return       │          │              └──────▶  │ ... return   │
       │ IRQ_WAKE_THREAD  │          │                        │ IRQ_HANDLED  │
       └──────────────────┘          │                        └──────────────┘
```

### 8. Behavioral rules

- When asked to "discuss" or "review" a plan, engage conversationally with concise observations and questions. Do not immediately start executing, writing files, or producing verbose output. Wait for explicit approval before creating files.
- When creating large batches of documentation pages (10+), work in batches of 5-6 files at a time to avoid rate limits. After each batch, checkpoint progress and report what is done vs remaining.
- Always read template/reference files first before generating any content.
- When using parallel sub-agents (Agent tool), ensure they have Write permissions before spawning. If Write is unavailable to agents, fall back to sequential processing immediately rather than failing and retrying.
- When performing batch edits across many files, preserve existing content (e.g., lspci output, code references) that was added in prior passes. Read the full file before editing to avoid accidentally removing prior enrichments.

### 9. Save the page

Write the completed page to: `${CLAUDE_SKILL_DIR}/docs/<dir>/<topic-slug>.md`

Do not modify `SUMMARY.md` or `mkdocs.yml`.

Ask before doing actual save.

## Reference pages

For style and structure examples, look into existing pages in `${CLAUDE_SKILL_DIR}`.

## Subsystem Map

Each entry maps a subsystem to its tag, output directory, primary kernel source paths, specification name(s), and the heading to use for section 6.

### PCIe

- tag: `pcie`
- dir: `pci`
- kernel_paths: `drivers/pci/`, `include/linux/pci.h`, `include/uapi/linux/pci_regs.h`
- spec: PCI Express Base Specification
- section6_heading: REGISTERS

### xHCI

- tag: `usb` (secondary: `xhci`)
- dir: `xhci`
- kernel_paths: `drivers/usb/host/xhci*`, `include/linux/usb/hcd.h`
- spec: xHCI (eXtensible Host Controller Interface) Specification
- section6_heading: REGISTERS

### USB

- tag: `usb`
- dir: `usb`
- kernel_paths: `drivers/usb/core/`, `drivers/usb/common/`, `include/linux/usb.h`, `include/linux/usb/ch9.h`
- spec: USB 2.0 Specification, USB 3.2 Specification
- section6_heading: REGISTERS

### ACPI

- tag: `acpi`
- dir: `acpi`
- kernel_paths: `drivers/acpi/`, `include/acpi/`, `include/linux/acpi.h`
- spec: ACPI Specification
- section6_heading: METHODS

### USB4

- tag: `usb4`
- dir: `usb4`
- kernel_paths: `drivers/thunderbolt/`, `include/linux/thunderbolt.h`
- spec: USB4 Specification, Thunderbolt 3/4 Specification
- section6_heading: REGISTERS

### V4L2

- tag: `v4l2`
- dir: `v4l2`
- kernel_paths: `drivers/media/`, `include/media/`, `include/uapi/linux/videodev2.h`
- spec: (none; refer to V4L2 subsystem documentation)
- section6_heading: INTERFACES

### DisplayPort

- tag: `display-port`
- dir: `dp`
- kernel_paths: `drivers/gpu/drm/display/drm_dp*`, `include/drm/display/drm_dp*`
- spec: VESA DisplayPort Standard, VESA eDP Standard
- section6_heading: REGISTERS

### DRM

- tag: `graphics`
- dir: `drm`
- kernel_paths: `drivers/gpu/drm/`, `include/drm/`, `include/uapi/drm/`
- spec: (none; refer to DRM subsystem documentation)
- section6_heading: INTERFACES

### Sound

- tag: `sound`
- dir: `sound`
- kernel_paths: `sound/`, `include/sound/`, `include/uapi/sound/`
- spec: Intel High Definition Audio Specification, USB Audio Class Specification
- section6_heading: REGISTERS

### Power Management

- tag: `power-management`
- dir: `pm`
- kernel_paths: `drivers/base/power/`, `kernel/power/`, `include/linux/pm.h`, `include/linux/suspend.h`
- spec: ACPI Specification (power management chapters), PCI PM Specification
- section6_heading: none

### Concurrency

- tag: `concurrency`
- dir: `concurrency`
- kernel_paths: `kernel/locking/`, `include/linux/spinlock.h`, `include/linux/mutex.h`, `include/linux/rwsem.h`
- spec: (none)
- section6_heading: PRIMITIVES

### Drivers

- tag: `drivers`
- dir: `drivers`
- kernel_paths: `drivers/base/`, `include/linux/device.h`, `include/linux/platform_device.h`
- spec: (none)
- section6_heading: INTERFACES

### Debugging

- tag: `debugging`
- dir: `debug`
- kernel_paths: `kernel/trace/`, `lib/dynamic_debug.c`, `include/linux/ftrace.h`
- spec: (none)
- section6_heading: none

### ARM64

- tag: `arm64`
- dir: `arm64`
- kernel_paths: `arch/arm64/`, `include/asm-generic/`
- spec: Arm Architecture Reference Manual (Arm ARM)
- section6_heading: REGISTERS

### Workflows

- tag: `workflows`
- dir: `workflows`
- kernel_paths: (none; workflow pages describe development processes)
- spec: (none)
- section6_heading: none

### Networking

- tag: `networking`
- dir: `net`
- kernel_paths: `net/core/`, `net/netfilter/`, `net/sched/`, `net/dsa/`, `net/bridge/`, `net/switchdev/`, `net/netlink/`, `drivers/net/`, `include/linux/netdevice.h`, `include/linux/skbuff.h`, `include/net/`
- spec: (none; linux network subsystem constructs)
- section6_heading: INTERFACES

### Ethernet

- tag: `ethernet`
- dir: `ethernet`
- kernel_paths: `drivers/net/ethernet/`, `drivers/net/phy/`, `drivers/net/mdio/`, `net/ethtool/`, `include/linux/etherdevice.h`, `include/linux/ethtool.h`, `include/linux/phylink.h`, `include/linux/phy.h`, `include/linux/mdio.h`, `include/linux/mii.h`, `include/linux/of_mdio.h`, `include/uapi/linux/ethtool.h`, `include/uapi/linux/ethtool_netlink.h`, `include/uapi/linux/mii.h`
- spec: IEEE 802.3 (Ethernet)
- section6_heading: REGISTERS

### Bluetooth

- tag: `bluetooth`
- dir: `bluetooth`
- kernel_paths: `net/bluetooth/`, `drivers/bluetooth/`, `include/net/bluetooth/`
- spec: Bluetooth Core Specification
- section6_heading: INTERFACES

### Memory Management

- tag: `mm`
- dir: `mm`
- kernel_paths: `mm/`, `include/linux/mm.h`, `include/linux/mm_types.h`, `include/linux/mm_types_task.h`, `include/linux/mmzone.h`, `include/linux/gfp.h`, `include/linux/gfp_types.h`, `include/linux/page-flags.h`, `include/linux/page-flags-layout.h`, `include/linux/page_ref.h`, `include/linux/pageblock-flags.h`, `include/linux/page-isolation.h`, `include/linux/pagemap.h`, `include/linux/pfn.h`, `include/linux/memblock.h`, `include/linux/memremap.h`, `include/linux/slab.h`, `include/linux/nodemask.h`, `include/linux/numa.h`, `include/linux/percpu.h`, `include/linux/mmdebug.h`, `include/linux/poison.h`, `include/linux/highmem-internal.h`, `include/linux/hugetlb.h`, `include/linux/rmap.h`, `include/linux/sched/mm.h`, `include/vdso/page.h`, `include/asm-generic/memory_model.h`, `include/asm-generic/pgalloc.h`, `include/net/page_pool/types.h`, `arch/x86/include/asm/page.h`, `arch/x86/include/asm/page_types.h`, `arch/x86/include/asm/sparsemem.h`
- spec: (none)
- section6_heading: none
