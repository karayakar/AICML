# AICML Language Usage Guide

This guide explains how to write, read, and extend documents expressed in AICML (AI Content Markup Language)—a block-based, plain-text syntax designed for AI-first content pipelines.

## 1. Mental Model

- **Plain text, structured meaning**: Every semantic unit is a block identified by an uppercase header (`PAGE`, `SECTION`, `ROW`, etc.). There is no punctuation beyond colons and indentation, so files stay diff-friendly.
- **Deterministic parsing**: Consumers scan line-by-line, switching context whenever they encounter a new uppercase header ending with `:`.
- **Layered design**: Narrative blocks coexist with entity records, compact datasets, and embedded analytic queries in one file.

## 2. Block Basics

| Block | Purpose | Key Fields |
| --- | --- | --- |
| `PAGE` | Declares the document. | `id`, `title`, `lang`, `type`, `owner`, custom metadata. |
| `META` | Arbitrary key/value metadata. | `name`, `value`. |
| `SECTION` | Groups narrative content. | `id`, `level`, `importance`, `entities`, `sets`. |
| Narrative (`TITLE`, `P`, `LIST`, `ITEM`, `NOTE`, `QA`, `CODE`, `MEDIA`) | Express prose without HTML. | `TITLE` holds one line, `P` can span multiple sentences, etc. |
| `ENTITY` | Rich object with dotted attributes. | `kind`, `id`, and domain-specific keys (`price.amount`, `advisor.primary`). |
| `DICT` | Defines short aliases for tabular data. | `field.<alias>: <full_path_or_label>`. |
| `SET` | Binds a DICT to a dataset. | `id`, `dict`, `kind`, optional descriptors. |
| `ROW` | Instance data that uses DICT aliases. | `alias: value` pairs only. |
| `QUERY` | Embedded analytics / AICQL. | `from.set`, `using.dict`, `select`, `where`, `group`, `sort`, `limit`. |

### Syntax rules

1. **Headers**: Uppercase word + colon, e.g., `SECTION:`. No inline text on the header line.
2. **Fields**: Indent two spaces (convention) and write `key: value`. Keys are case-sensitive.
3. **Bodies**: For `P`, `CODE`, or other text blocks, everything after the initial field lines belongs to that block until the next uppercase header.
4. **Blank lines**: Optional but recommended between blocks for readability; parsers ignore consecutive blank lines.

## 3. Writing Narrative Layers

1. Start with a `PAGE` block and at least one `META` describing context.
2. Create `SECTION` blocks for each major topic. Optional attributes:
    - `level`: heading depth (1 = top-level).
    - `importance`: `high`, `medium`, `low`, or numeric weight.
    - `entities` / `sets`: comma-separated references that tie structured data to the section.
3. Add textual blocks under each section:
    ```
    SECTION:
       id: overview
       level: 1

    TITLE:
       Product Overview

    P:
       This release introduces ...
    ```
4. Use `LIST` + `ITEM` for bullet summaries, `NOTE` for callouts, `QA` for FAQ-style prompts, and `CODE`/`MEDIA` when embedding snippets or media metadata.

## 4. Entities & Dictionaries

### ENTITY

- Ideal for detailed records (products, patients, policies).
- Use dotted keys to express hierarchies: `price.amount`, `advisor.primary.email`.
- Example:
   ```
   ENTITY:
      id: client-profile
      kind: investor
      name: Aurora Chen Family Trust
      risk.score: 54
      advisor.primary: Devon Ortiz
   ```

### DICT + SET + ROW

1. **DICT** creates a shared vocabulary of aliases:
    ```
    DICT:
       id: dict.product.v1
       kind: product
       field.i: sku
       field.n: display_name
       field.p: price_usd
    ```
2. **SET** references the dictionary:
    ```
    SET:
       id: spring-lineup
       dict: dict.product.v1
       kind: footwear
    ```
3. **ROW** instances only use the aliases:
    ```
    ROW:
       i: AF-RACE-VOLT
       n: Race Volt X
       p: 159.00
    ```

> **Why aliases?** Rows stay compact, saving tokens and making large tables easy to diff. Parsers expand each alias via the DICT definition.

### Alias guidelines

- Keep aliases 1–3 characters; avoid spaces or punctuation.
- Make them mnemonic when possible (`i` = id, `p` = price, `st` = stack height).
- Do not reuse the same alias twice within one DICT.

## 5. Queries (AICQL)

`QUERY` blocks embed lightweight analytics:

```
QUERY:
   id: q.top.performers
   from.set: spring-lineup
   using.dict: dict.product.v1
   select: i,n,p
   where: p >= 150
   sort: p desc
   limit: 5
```

- **from.set** must match a `SET` id.
- **using.dict** is optional when the set already points to a dict, but including it increases clarity.
- **select** accepts comma-separated aliases or aggregate expressions (`sum(p)`).
- **where/group/sort/limit** mirror SQL semantics. Strings should be quoted.

## 6. Conventions & Best Practices

### Identification
- Use stable, lowercase/hyphen IDs for PAGE/SECTION/ENTITY/SET names (`products-2026`, `client-profile`).
- Reference IDs literally; the language is case-sensitive.

### Whitespace
- Two-space indentation is conventional but not mandated; be consistent.
- Avoid tab characters—mixing tabs/spaces complicates diffing.

### Comments
- AICML intentionally avoids inline comments. Store notes in `NOTE` blocks or `META` entries to keep parsers simple.

### Versioning
- Track schema evolution with `META` keys (`schema.version`, `generated.on`).
- When changing a DICT alias, rename it in the DICT **and** update every ROW/QUERY referencing it.

## 7. Validation & Tooling Tips

- **Parser smoke tests**: run your favorite parser or the web-based token lab to ensure each block converts into the expected AST.
- **Diffing**: Because files are plain text, Git diffs remain readable. Group related changes (e.g., update DICT + ROWS together).
- **Token planning**: Use the token lab or scripts with `js-tiktoken`/OpenAI encoders to estimate savings versus raw HTML or JSON.
- **Static analysis**: Simple linters can check for missing `dict` references, unused aliases, or duplicate IDs by scanning headers.

## 8. Example Mini-Document

```
PAGE:
   id: sample-brief
   title: Battery Launch Notes

SECTION:
   id: highlights
   level: 1

TITLE:
   Highlights

LIST:
   style: unordered

ITEM:
   12% higher energy density versus prior gen.

DICT:
   id: dict.metrics.v1
   kind: metric
   field.i: metric_id
   field.n: name
   field.v: value

SET:
   id: perf-metrics
   dict: dict.metrics.v1

ROW:
   i: M-ENERGY
   n: Energy Density
   v: 280 Wh/kg

QUERY:
   id: q.metric-count
   from.set: perf-metrics
   using.dict: dict.metrics.v1
   select: count(i)
```

This small file demonstrates every core layer in under 30 lines.

## 9. Getting Comfortable

- Start with narrative sections, then layer in structured data as DICT/SET/ROW once aliases feel natural.
- Reuse patterns: copy a DICT/SET block from another file and modify the alias descriptions to match your domain.
- Keep validation simple: if a human can skim and understand the intent, an AICML parser should be able to map it to JSON deterministically.

AICML’s goal is to keep AI-facing documents human-writable, versionable, and cheap to tokenize—use these guidelines as your checklist whenever you author or review an `.aicml` file.
