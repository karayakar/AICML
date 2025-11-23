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
