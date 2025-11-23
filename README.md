Contents
--------

1.  [1\. Mission & Layers](#overview)
2.  [2\. Block Reference](#blocks)
3.  [3\. Document layer](#document-layer)
    *   [3.1 PAGE & META](#page-meta)
    *   [3.2 SECTION & text blocks](#sections)
4.  [4\. Entity layer](#entity-layer)
5.  [5\. Compact/tabular layer](#compact-layer)
6.  [6\. Query layer (AICQL)](#query-layer)
7.  [7\. HTML coverage & mapping](#html-coverage)
8.  [8\. Full mini example](#example)
9.  [9\. Tooling & schemas](#next-steps)

What Chat GPT says about AI:CML ?
---------------------------------

![](aicml-vs-json-comp.jpg)

What Chat GPT says about AI:CML ?
---------------------------------

![](aicml-vs-toon.jpg)

1\. Mission & Layers
--------------------

AI:CML 1.0 keeps the “no presentation noise” spirit of the earliest drafts but remains layered enough to describe entire content sites, structured catalogs, and the queries people run on top of them—all inside one friendly, token-efficient text file. Blocks are written with UPPERCASE headers (e.g., `PAGE:`, `ROW:`). Everything until the next header belongs to the current block.

Layer

Purpose

Key blocks

Document

Readable narrative equivalent to HTML articles, docs, FAQs.

PAGE, META, SECTION, TITLE, P, LIST, NOTE, QA, CODE, MEDIA

Entity

Rich single objects (product, book, event, persona).

ENTITY

Compact/tabular

Thousands of similar items with minimal tokens.

DICT, SET, ROW

Query

Inline analytics or API-like requests.

QUERY (AICQL)

**Design goals:** 100% plain text, no XML/HTML/Markdown; dictionary-driven compression for large datasets; all semantics explicit so LLMs or classic parsers can build a JSON AST and execute queries deterministically.

2\. Block Reference
-------------------

Block headers are uppercase followed by a colon. Recognised headers for 1.0:

PAGE, META, SECTION, TITLE, P, LIST, ITEM, NOTE, EXAMPLE, QA, Q, A, CODE, MEDIA, ENTITY, DICT, SET, ROW, QUERY. Custom blocks may be added; consumers must ignore unknown blocks safely.

Each block stores attributes as `key: value` lines. Multi-line bodies (e.g., paragraph text, code) continue until the next uppercase header.

3\. Document layer
------------------

Replace HTML scaffolding while keeping semantics intact. PAGE and SECTION metadata track IDs, language, ordering, and importance so downstream agents can rebuild navigation, breadcrumbs, summaries, or localization matrices.

### 3.1 PAGE & META

    PAGE:
      id: shop-001
      lang: en
      title: Spring Catalog
      type: landing
    
    META:
      name: audience
      value: consumer
    
    META:
      name: region
      value: global

PAGE is required once per file. META blocks are optional but unlimited. Use them for analytics, targeting, or build metadata.

### 3.2 SECTION & narrative blocks

    SECTION:
      id: catalog-intro
      type: intro
      importance: high
      level: 1
      order: 1
    
    TITLE:
      Spring Catalog Overview
    
    P:
      This catalog lists the running shoes shipping this season.
      Narrative blocks mirror HTML semantics: TITLE → h*, P → paragraph,
      LIST+ITEM → ul/ol, QA → FAQ, CODE → pre/code, MEDIA → img/video/audio.
    
    NOTE:
      Pinned data set: products-running-2026

SECTION can reference structured data via `sets:`, `entities:`, or custom keys so readers jump directly from prose to data.

4\. Entity layer
----------------

ENTITY blocks describe a single rich object without aliases. They are perfect for hero products, a featured event, or any item where readability matters more than token count.

    ENTITY:
      id: product-hero-001
      kind: product
      name: Ultra Light Runner
      category: footwear
      category.sub: running
      price.amount: 129.90
      price.currency: USD
      weight_kg: 0.72
      shipping.weight_kg: 0.90
      shipping.regions: US,CA,EU
      image.main: https://cdn.example.com/p/hero001-main.png
      image.thumbnail: https://cdn.example.com/p/hero001-thumb.png

Keys are dotted paths so downstream consumers can map them to JSON or graph structures losslessly.

5\. Compact/tabular layer
-------------------------

DICT + SET + ROW keep large catalogs small. DICT stores aliases, SET declares which dictionary applies, ROW instances carry only alias=value pairs. This is the “table mode” that scales to thousands of rows without repeating semantic text.

    DICT:
      id: dict.product.v1
      kind: product
      field.i: entity.id
      field.n: entity.name
      field.c: category
      field.sc: category.sub
      field.pa: price.amount
      field.pc: price.currency
      field.sz: size.available
      field.cl: color.options
      field.w: weight_kg
      field.sw: shipping.weight_kg
      field.sr: shipping.regions
    
    SET:
      id: products-running-2026
      dict: dict.product.v1
      kind: product
      description: Running shoes for Spring 2026
    
    ROW:
      i: prod-run-1001
      n: Road Runner Lite
      c: footwear
      sc: running
      pa: 89.90
      pc: USD
      sz: 40,41,42,43,44
      cl: black,blue
      w: 0.75
      sw: 0.95
      sr: US,CA,EU

Dict aliases are short (1–3 characters) so the ROW body is dense yet self-describing once the DICT is parsed.

6\. Query layer (AICQL)
-----------------------

QUERY blocks provide embedded analytics. They reference ENTITY IDs or SET IDs, optionally specify a DICT, and expose simple line-based clauses reminiscent of SQL. Tools can evaluate these against the parsed AST to generate answers on the fly.

    QUERY:
      id: q.products.over100
      from.set: products-running-2026
      using.dict: dict.product.v1
      where: pa > 100 and pc = "USD"
      select: i,n,pa,pc,sc
      sort: pa desc
      limit: 5

Clause summary: `from.set` or `from.entity`; optional `using.dict`; `where` with lightweight boolean expressions; `select` comma list; optional `sort` and `limit`.

7\. HTML coverage & mapping
---------------------------

Every core HTML content element maps directly to an AICML block. Because blocks are agnostic to layout, the mapping is one-to-many friendly and survives aggressive extraction pipelines.

HTML element(s)

AICML block

Notes

html/head/body meta tags

PAGE, META

PAGE captures document-level attributes; META mirrors `<meta>` tags.

h1–h6, section headings

SECTION + TITLE

`level` attribute ensures hierarchy.

p, span, div (textual)

P

Multi-line friendly; inline emphasis may use lightweight markup if needed.

ul/ol/li, dl/dt/dd

LIST + ITEM

`LIST` can express bulleted or numbered; description lists map to ITEM with nested keys.

blockquote, aside

NOTE or EXAMPLE

Use `type: info|tip|warning` for semantic hints.

pre/code

CODE

Language captured via `lang:`.

figure/img/video/audio

MEDIA

Attributes: `type`, `src`, optional `alt`/`title`.

faq accordions, details/summary

QA + Q + A

Supports multiple answers per question.

data tables, cards, repeated components

ENTITY or DICT+SET+ROW

ENTITY for small sets; DICT/SET for thousands of records.

This mapping ensures “support for all HTML elements” relevant to semantic content. Presentation-only tags (layout divs, CSS classes) are intentionally ignored to keep the signal clean.

8\. Full mini example
---------------------

    PAGE:
      id: shop-001
      lang: en
      title: Spring Running Catalog 2026
      type: landing
    
    META:
      name: audience
      value: consumer
    
    SECTION:
      id: intro
      type: intro
      importance: high
      level: 1
      order: 1
    
    TITLE:
      Spring Running Collection
    
    P:
      This catalog lists our running shoes for the spring season.
      Data is stored in a compact format for efficient processing.
    
    SECTION:
      id: running-products
      type: concept
      importance: high
      level: 1
      order: 2
      sets: products-running-2026
    
    TITLE:
      Running Shoes
    
    P:
      The following data set contains all running shoe models, including
      prices, sizes, colors, and shipping regions.
    
    DICT:
      id: dict.product.v1
      kind: product
      field.i: entity.id
      field.n: entity.name
      field.c: category
      field.sc: category.sub
      field.pa: price.amount
      field.pc: price.currency
      field.sz: size.available
      field.cl: color.options
      field.w: weight_kg
      field.sw: shipping.weight_kg
      field.sr: shipping.regions
    
    SET:
      id: products-running-2026
      dict: dict.product.v1
      kind: product
      description: Running shoes for Spring 2026
    
    ROW:
      i: prod-run-1001
      n: Road Runner Lite
      c: footwear
      sc: running
      pa: 89.90
      pc: USD
      sz: 40,41,42,43,44
      cl: black,blue
      w: 0.75
      sw: 0.95
      sr: US,CA,EU
    
    ROW:
      i: prod-run-1002
      n: City Sprint
      c: footwear
      sc: running
      pa: 79.00
      pc: USD
      sz: 39,40,41,42,43
      cl: gray,white
      w: 0.72
      sw: 0.92
      sr: US,EU
    
    ROW:
      i: prod-run-1003
      n: Trail Climber Pro
      c: footwear
      sc: trail
      pa: 115.50
      pc: USD
      sz: 41,42,43,44,45
      cl: green,black
      w: 0.85
      sw: 1.05
      sr: US,CA
    
    QUERY:
      id: q.running.over100
      from.set: products-running-2026
      using.dict: dict.product.v1
      where: pa > 100 and pc = "USD"
      select: i,n,pa,pc,sc
      sort: pa desc
      limit: 5

9\. Tooling & schemas
---------------------

AICML 1.0 is the canonical plain-text format going forward. Legacy XML-based AICML deployments remain available for archival needs; consult the developer appendix for the JSON AST and parser patterns that power modern pipelines.

👉 Developer appendix: [JSON AST + Parser Patterns](schema.html)

Next ideas:

*   Publish reference parser packages (npm, PyPI, NuGet, Maven) built on the shared AST.
*   Deliver HTML → AICML 1.0 extraction recipes for common CMS stacks.
*   Ship an AICQL evaluator + playground so embedded queries can run in CI or docs sites.




Contents
--------

1.  [1\. Overview](#overview)
2.  [2\. Canonical JSON AST](#json-ast)
    *   [2.1 Document](#json-document)
    *   [2.2 Page & meta](#json-page)
    *   [2.3 Sections & blocks](#json-sections)
    *   [2.4 Entities](#json-entities)
    *   [2.5 Dict / Set / Row](#json-dict)
    *   [2.6 Queries](#json-queries)
3.  [3\. Parser state machine](#parser)
    *   [3.1 Token model](#parser-token)
    *   [3.2 State loop](#parser-loop)
    *   [3.3 Flush semantics](#parser-flush)
4.  [4\. Language skeletons](#languages)
    *   [4.1 JavaScript / React](#js)
    *   [4.2 Python](#python)
    *   [4.3 C#](#csharp)
    *   [4.4 Java](#java)
    *   [4.5 Other runtimes](#others)
5.  [5\. Next steps](#next)

1\. Overview
------------

The plain-text AICML syntax is human-friendly, but production systems should consume a consistent JSON graph. The structures below are the canonical representation—emit them from every parser, no matter the stack.

Looking for the XML-era XSD/Relax NG? Fetch the previous revision of this appendix from source control. This edition focuses on the AICML 1.0 block syntax → JSON pipeline and parser architecture.

2\. Canonical JSON AST
----------------------

Interfaces are shown in TypeScript notation; mirror them as dataclasses, records, or POCOs in your language.

### 2.1 Top-level document

    interface AicmlDocument {
      page: Page;
      meta: Meta[];
      sections: Section[];
      entities: Entity[];
      dicts: Dict[];
      sets: DataSet[];
      queries: Query[];
    }

### 2.2 Page & meta

    interface Page {
      id: string;
      lang?: string;
      title?: string;
      type?: string; // article | docs | landing | faq | ...
    }
    
    interface Meta {
      name: string;
      value: string;
    }

### 2.3 Sections & blocks

    interface Section {
      id: string;
      type?: string;
      importance?: "high" | "normal" | "low";
      level?: number;
      order?: number;
      parentId?: string | null;
      sets?: string[];
      entities?: string[];
      blocks: SectionBlock[];
    }
    
    type SectionBlock =
      | TitleBlock
      | ParagraphBlock
      | ListBlock
      | NoteBlock
      | ExampleBlock
      | QaBlock
      | CodeBlock
      | MediaBlock;
    
    interface TitleBlock { kind: "title"; text: string; }
    interface ParagraphBlock { kind: "paragraph"; text: string; }
    
    interface ListBlock {
      kind: "list";
      style: "ordered" | "unordered";
      items: ListItem[];
    }
    
    interface ListItem {
      order?: number;
      text: string;
    }
    
    interface NoteBlock {
      kind: "note";
      noteType: "info" | "warning" | "tip";
      importance?: "high" | "normal" | "low";
      blocks: ParagraphBlock[]; // extend with lists/code if needed
    }
    
    interface ExampleBlock {
      kind: "example";
      blocks: Array;
    }
    
    interface QaBlock {
      kind: "qa";
      question: string;
      answer: string | string[];
    }
    
    interface CodeBlock {
      kind: "code";
      lang?: string;
      code: string;
    }
    
    interface MediaBlock {
      kind: "media";
      mediaType: "image" | "video" | "audio" | "diagram";
      src: string;
      alt?: string;
      title?: string;
    }

### 2.4 Entities

    interface Entity {
      id: string;
      kind: string; // product | book | event | ...
      name?: string;
      order?: number;
      parentId?: string | null;
      role?: string;
      properties: Record; // dotted semantic keys
    }

### 2.5 Dict / Set / Row

    interface DictField {
      alias: string; // pa
      path: string;  // price.amount
    }
    
    interface Dict {
      id: string;
      kind?: string;
      fields: DictField[];
    }
    
    interface DataRow {
      values: Record; // alias -> value
    }
    
    interface DataSet {
      id: string;
      dictId: string;
      kind?: string;
      description?: string;
      rows: DataRow[];
    }

### 2.6 Queries

    type SortDirection = "asc" | "desc";
    
    interface SortSpec {
      field: string;
      direction: SortDirection;
    }
    
    interface Query {
      id: string;
      fromSetId?: string;
      fromEntityId?: string;
      usingDictId?: string;
      where?: string;
      select?: string[];
      sort?: SortSpec[];
      limit?: number;
    }

3\. Parser state machine
------------------------

### 3.1 Token model

*   Headers: `^[A-Z]+:` (PAGE, SECTION, ROW, QUERY, ...).
*   Key/value lines: `key: value` inside a block.
*   Paragraph and code blocks buffer plain text until the next header.
*   Whitespace-only lines inside text blocks should be preserved for CODE, collapsed for P.

### 3.2 State loop

    state = {
      currentBlock: null,
      currentSection: null,
      currentList: null,
      currentDict: null,
      currentSet: null,
      currentRow: null,
      currentQuery: null,
      paragraphLines: [],
      codeLang: null,
      codeLines: []
    }
    
    for each line in file:
      if matchesHeader(line):
        flushCurrentBlock()
        startBlock(headerName)
      else:
        appendToBlock(line)
    
    flushCurrentBlock() // handle final block

### 3.3 Flush semantics & helpers

*   **P:** join `paragraphLines`, trim, push ParagraphBlock.
*   **CODE:** join `codeLines` with `\n`, attach `codeLang`.
*   **LIST/ITEM:** instantiate ListBlock when LIST starts; each ITEM appends immediately.
*   **NOTE/EXAMPLE/QA:** maintain temporary objects just like SECTION; push when a new header begins.
*   **DICT/SET/ROW:** DICT goes to `doc.dicts`; SET goes to `doc.sets`; ROWs append to `currentSet.rows`.
*   **QUERY:** parse `select` into arrays (split commas) and `sort` into `SortSpec` tuples.

4\. Language skeletons
----------------------

Each sample references the shared logic above; expand the model classes per Section 2.

### 4.1 JavaScript / React

    const HEADER_RE = /^([A-Z]+):\s*$/;
    const KV_RE = /^\s*([a-zA-Z0-9_.]+):\s*(.*)$/;
    
              export function parseAicml(text) {
      const lines = text.split(/\r?\n/);
      const doc = { page: { id: "" }, meta: [], sections: [], entities: [], dicts: [], sets: [], queries: [] };
      let currentBlock = null;
      let currentSection = null;
      let paragraph = [];
      let codeLang;
      let codeLines = [];
      // ...holders for list/note/example/dict/set/row/query/entity/meta
    
      const flush = () => {
        if (currentBlock === "P" && currentSection) {
          const text = paragraph.join(" ").trim();
          if (text) currentSection.blocks.push({ kind: "paragraph", text });
          paragraph = [];
        }
        if (currentBlock === "CODE" && currentSection) {
          currentSection.blocks.push({ kind: "code", lang: codeLang, code: codeLines.join("\n") });
          codeLang = undefined;
          codeLines = [];
        }
        if (currentBlock === "SECTION" && currentSection) {
          doc.sections.push(currentSection);
          currentSection = null;
        }
        // flush DICT, SET, ROW, QUERY, ENTITY, NOTE, EXAMPLE, QA as needed
        currentBlock = null;
      };
    
      for (const rawLine of lines) {
        const line = rawLine.trimEnd();
        const header = line.match(HEADER_RE);
        if (header) {
          flush();
          currentBlock = header[1];
          if (currentBlock === "SECTION") currentSection = { id: "", blocks: [] };
          if (currentBlock === "P") paragraph = [];
          if (currentBlock === "CODE") { codeLang = undefined; codeLines = []; }
          // initialize LIST, NOTE, EXAMPLE, QA, DICT, SET, ROW, QUERY, ENTITY, META etc.
          continue;
        }
    
        const kv = line.match(KV_RE);
        if (currentBlock === "PAGE" && kv) { doc.page[kv[1]] = kv[2]; continue; }
        if (currentBlock === "SECTION" && kv && currentSection) {
          currentSection[kv[1]] = isFinite(+kv[2]) ? Number(kv[2]) : kv[2];
          continue;
        }
        if (currentBlock === "P") { paragraph.push(line.trim()); continue; }
        if (currentBlock === "CODE") {
          if (kv && kv[1] === "lang") codeLang = kv[2];
          else codeLines.push(rawLine);
          continue;
        }
        // handle META, LIST, ITEM, NOTE, ENTITY, DICT, SET, ROW, QUERY with similar guards
      }
    
      flush();
      return doc;
    }
    
    // React usage: const ast = useMemo(() => parseAicml(raw), [raw]);

### 4.2 Python

    HEADER_RE = re.compile(r'^([A-Z]+):\s*$')
    KV_RE = re.compile(r'^\s*([a-zA-Z0-9_.]+):\s*(.*)$')
    
    @dataclass
    class Page:
        id: str = ""
        lang: str | None = None
        title: str | None = None
        type: str | None = None
    
    @dataclass
    class AicmlDocument:
        page: Page
        meta: list[Meta] = field(default_factory=list)
        sections: list[Section] = field(default_factory=list)
        entities: list[Entity] = field(default_factory=list)
        dicts: list[Dict] = field(default_factory=list)
        sets: list[DataSet] = field(default_factory=list)
        queries: list[Query] = field(default_factory=list)
    
    
    def parse_aicml(text: str) -> AicmlDocument:
      doc = AicmlDocument(page=Page())
        current_block = None
        current_section = None
        paragraph: list[str] = []
        code_lines: list[str] = []
        code_lang: str | None = None
        # initialize holders for dict/set/row/query/list/note/example/qa/meta/entity
    
        def flush():
            nonlocal current_block, paragraph, code_lines, code_lang, current_section
            if current_block == "P" and current_section:
                text = " ".join(paragraph).strip()
                if text:
                    current_section.blocks.append(ParagraphBlock(kind="paragraph", text=text))
                paragraph = []
            if current_block == "CODE" and current_section:
                current_section.blocks.append(CodeBlock(kind="code", lang=code_lang, code="\n".join(code_lines)))
                code_lines, code_lang = [], None
            if current_block == "SECTION" and current_section:
                doc.sections.append(current_section)
                current_section = None
            # flush DICT/SET/ROW/QUERY/META/ENTITY/etc.
            current_block = None
    
        for raw in text.splitlines():
            m_header = HEADER_RE.match(raw)
            if m_header:
                flush()
                current_block = m_header.group(1)
                if current_block == "SECTION":
                    current_section = Section(id="", blocks=[])
                elif current_block == "P":
                    paragraph = []
                elif current_block == "CODE":
                    code_lines, code_lang = [], None
                # init other block objects
                continue
    
            m_kv = KV_RE.match(raw)
            if current_block == "PAGE" and m_kv:
                setattr(doc.page, m_kv.group(1), m_kv.group(2))
                continue
            if current_block == "SECTION" and current_section and m_kv:
                setattr(current_section, m_kv.group(1).replace('.', '_'), coerce(m_kv.group(2)))
                continue
            if current_block == "P":
                paragraph.append(raw.strip())
                continue
            if current_block == "CODE":
                if m_kv and m_kv.group(1) == "lang":
                    code_lang = m_kv.group(2).strip()
                else:
                    code_lines.append(raw)
                continue
            # handle remaining blocks here
    
        flush()
        return doc

### 4.3 C#

    public sealed class AicmlParser {
      private static readonly Regex Header = new("^([A-Z]+):\\s*$", RegexOptions.Compiled);
      private static readonly Regex Kv = new("^\\s*([a-zA-Z0-9_.]+):\\s*(.*)$", RegexOptions.Compiled);
    
      public AicmlDocument Parse(string text) {
        var doc = new AicmlDocument();
        var lines = text.Split(new[] { "\r\n", "\n" }, StringSplitOptions.None);
    
        string currentBlock = null;
        Section currentSection = null;
        var paragraph = new List();
        var codeLines = new List();
        string codeLang = null;
        // holders for dict/set/row/query/list/note/example/qa/meta/entity
    
        void Flush() {
          if (currentBlock == "P" && currentSection != null) {
            var textValue = string.Join(" ", paragraph).Trim();
            if (textValue.Length > 0)
              currentSection.Blocks.Add(new ParagraphBlock { Kind = "paragraph", Text = textValue });
            paragraph.Clear();
          }
          if (currentBlock == "CODE" && currentSection != null) {
            currentSection.Blocks.Add(new CodeBlock { Kind = "code", Lang = codeLang, Code = string.Join("\n", codeLines) });
            codeLines.Clear();
            codeLang = null;
          }
          if (currentBlock == "SECTION" && currentSection != null) {
            doc.Sections.Add(currentSection);
            currentSection = null;
          }
          // flush DICT/SET/ROW/QUERY/META/ENTITY/etc.
          currentBlock = null;
        }
    
        foreach (var raw in lines) {
          var line = raw.TrimEnd();
          var mHeader = Header.Match(line);
          if (mHeader.Success) {
            Flush();
            currentBlock = mHeader.Groups[1].Value;
            if (currentBlock == "SECTION") currentSection = new Section { Blocks = new List() };
            else if (currentBlock == "P") paragraph.Clear();
            else if (currentBlock == "CODE") { codeLines.Clear(); codeLang = null; }
            // init other block objects
            continue;
          }
    
          var kv = Kv.Match(line);
          if (currentBlock == "PAGE" && kv.Success) {
            doc.Page.Assign(kv.Groups[1].Value, kv.Groups[2].Value);
            continue;
          }
          if (currentBlock == "SECTION" && currentSection != null && kv.Success) {
            currentSection.Assign(kv.Groups[1].Value, kv.Groups[2].Value);
            continue;
          }
          if (currentBlock == "P") { paragraph.Add(line.Trim()); continue; }
          if (currentBlock == "CODE") {
            if (kv.Success && kv.Groups[1].Value == "lang") codeLang = kv.Groups[2].Value;
            else codeLines.Add(raw);
            continue;
          }
          // mirror logic for other block types
        }
    
        Flush();
        return doc;
      }
    }

### 4.4 Java

    public final class AicmlParser {
      private static final Pattern HEADER = Pattern.compile("^([A-Z]+):\\s*$");
      private static final Pattern KV = Pattern.compile("^\\s*([a-zA-Z0-9_.]+):\\s*(.*)$");
    
      public AicmlDocument parse(String text) {
        AicmlDocument doc = new AicmlDocument();
        String[] lines = text.split("\\r?\\n");
        String currentBlock = null;
        Section currentSection = null;
        List paragraph = new ArrayList<>();
        List codeLines = new ArrayList<>();
        String codeLang = null;
        // instantiate dict/set/row/query/list/note/example helpers
    
        Runnable flush = () -> {
          if ("P".equals(currentBlock) && currentSection != null) {
            String textValue = String.join(" ", paragraph).trim();
            if (!textValue.isEmpty()) currentSection.blocks.add(new ParagraphBlock("paragraph", textValue));
            paragraph.clear();
          }
          if ("CODE".equals(currentBlock) && currentSection != null) {
            currentSection.blocks.add(new CodeBlock("code", codeLang, String.join("\n", codeLines)));
            codeLines.clear();
            codeLang = null;
          }
          if ("SECTION".equals(currentBlock) && currentSection != null) {
            doc.sections.add(currentSection);
            currentSection = null;
          }
          // flush DICT/SET/ROW/QUERY/etc.
          currentBlock = null;
        };
    
        for (String raw : lines) {
          String line = raw.trimEnd();
          Matcher header = HEADER.matcher(line);
          if (header.matches()) {
            flush.run();
            currentBlock = header.group(1);
            if ("SECTION".equals(currentBlock)) currentSection = new Section();
            else if ("P".equals(currentBlock)) paragraph.clear();
            else if ("CODE".equals(currentBlock)) { codeLines.clear(); codeLang = null; }
            // init helpers for other block types
            continue;
          }
    
          Matcher kv = KV.matcher(line);
          if ("PAGE".equals(currentBlock) && kv.matches()) {
            doc.page.assign(kv.group(1), kv.group(2));
            continue;
          }
          if ("SECTION".equals(currentBlock) && currentSection != null && kv.matches()) {
            currentSection.assign(kv.group(1), kv.group(2));
            continue;
          }
          if ("P".equals(currentBlock)) { paragraph.add(line.trim()); continue; }
          if ("CODE".equals(currentBlock)) {
            if (kv.matches() && "lang".equals(kv.group(1))) codeLang = kv.group(2);
            else codeLines.add(raw);
            continue;
          }
          // handle META, LIST, ITEM, NOTE, ENTITY, DICT, SET, ROW, QUERY
        }
    
        flush.run();
        return doc;
      }
    }

### 4.5 Other runtimes

*   **Go:** structs + `bufio.Scanner`; reuse header/KV regex, keep streaming.
*   **Rust:** enums/structs + `lines()`; prefer manual prefix checks to avoid regex overhead.
*   **Kotlin/Swift:** sealed interfaces/data classes map cleanly to `SectionBlock` unions.
*   Whichever language you use, mirror the flush rules exactly so ROW/QUERY boundaries stay deterministic.

5\. Next steps
--------------

*   Publish shared AST typings/packages (npm, PyPI, NuGet, Maven).
*   Add schema validation (Zod, Pydantic, FluentValidation, Jakarta Bean Validation) right after parsing.
*   Implement an AICQL evaluator module that operates on the JSON AST for analytics and previews.

[Back to the main AICML 1.0 specification](index.html)
