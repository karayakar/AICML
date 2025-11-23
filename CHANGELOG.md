# Changelog

All notable changes to AI:CML (LLX) will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2024-11-23

### Added

#### Core Specification
- Complete AI:CML 1.0 format specification
- Plain-text, block-oriented document structure
- Support for treating documents as semantic databases
- Five core block types: ENTITY, DICT, SET, ROW, QUERY
- Narrative sections for human-readable content
- Comment support using `#` prefix

#### Block Types
- **ENTITY blocks**: Rich record structures with named fields
- **DICT blocks**: Dictionary/key-value pair structures for aliases and configuration
- **SET blocks**: Unordered collections of unique values
- **ROW blocks**: Tabular data with column headers and data rows
- **QUERY blocks**: AICQL query language for embedded data retrieval

#### Alias System
- Dictionary-based alias definitions
- `@` prefix for alias references (e.g., `@electronics`)
- Token efficiency optimization for AI processing
- Consistent term usage across documents

#### AICQL Query Language
- SELECT, FROM, WHERE, GROUP BY, ORDER BY, LIMIT clauses
- Comparison operators: `=`, `!=`, `>`, `<`, `>=`, `<=`
- Logical operators: `AND`, `OR`, `NOT`
- String operators: `CONTAINS`, `STARTS_WITH`, `ENDS_WITH`
- Set operators: `IN`, `NOT IN`
- Aggregation functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`, `DISTINCT()`
- Support for complex conditions and nested queries

#### Documentation
- Comprehensive format specification (SPECIFICATION.md)
- Formal grammar definition using EBNF (GRAMMAR.md)
- Usage guide with best practices (USAGE-GUIDE.md)
- Contributing guidelines (CONTRIBUTING.md)
- Enhanced README with quick start guide
- Examples directory with README

#### Examples
- Simple project tracking example (simple-project.aicml)
- E-commerce product catalog example (product-catalog.aicml)
- Academic research papers database example (research-papers.aicml)
- Medical records example with fictional data (medical-records.aicml)
- Comprehensive examples README explaining each use case

#### Features
- UTF-8 encoding support
- Cross-platform line ending support (LF and CRLF)
- Case-insensitive block keywords
- Case-sensitive field names and identifiers
- Flexible indentation (recommended 2 spaces)
- Support for multi-line values
- Comma-separated value lists in fields

### Documentation Details

#### SPECIFICATION.md
- Complete format overview and core principles
- Detailed syntax for all block types
- Alias system explanation with examples
- AICQL query language reference
- Complete example documents
- Parsing rules and conventions
- File extension and MIME type recommendations

#### GRAMMAR.md
- Formal EBNF grammar definition
- Lexical element specifications
- Operator precedence rules
- Type system documentation
- Validation rules
- Error handling guidelines
- Conformance levels (Basic, Standard, Advanced)

#### USAGE-GUIDE.md
- Getting started guide with templates
- Document structure recommendations
- Naming conventions for blocks and fields
- Alias management best practices
- Query design patterns
- Performance optimization tips
- Common patterns and anti-patterns
- Validation checklist
- Migration guides from JSON, CSV, YAML

#### Examples
- Four complete example documents covering different domains
- Real-world use case demonstrations
- Best practices implementation
- Token efficiency examples
- Query pattern demonstrations

### Technical Specifications

#### File Format
- Extension: `.aicml` or `.llx`
- MIME Type: `text/x-aicml` or `text/x-llx`
- Encoding: UTF-8 without BOM
- Line Endings: LF or CRLF

#### Reserved Keywords
- Block types: ENTITY, DICT, SET, ROW, QUERY, END
- Query clauses: SELECT, FROM, WHERE, GROUP BY, ORDER BY, LIMIT
- Logical operators: AND, OR, NOT, IN
- Sort orders: ASC, DESC

#### Design Goals
- Human readability and writability
- AI system optimization (token efficiency)
- Version control friendly (plain text)
- Self-contained queries and data
- Extensible for future enhancements
- Simple parser implementation

### Project Structure
```
AICML/
├── README.md              # Project overview and quick start
├── SPECIFICATION.md       # Complete format specification
├── GRAMMAR.md            # Formal grammar definition
├── USAGE-GUIDE.md        # Best practices and patterns
├── CONTRIBUTING.md       # Contribution guidelines
├── CHANGELOG.md          # This file
├── LICENSE               # MIT License
└── examples/
    ├── README.md                  # Examples overview
    ├── simple-project.aicml       # Basic project tracking
    ├── product-catalog.aicml      # E-commerce example
    ├── research-papers.aicml      # Academic database
    └── medical-records.aicml      # Healthcare example
```

### Initial Release Notes

AI:CML (LLX) 1.0 is the initial stable release of the AI Content Markup Language. This release establishes the core format specification and provides comprehensive documentation for implementers and users.

The format is designed to bridge human readability with AI system requirements, treating documents as tiny semantic databases that can be queried using the embedded AICQL language. The built-in alias system reduces token usage for AI processing while maintaining document clarity.

This release is considered stable for:
- Document authoring
- Parser implementation
- Tool development
- Production use

### Known Limitations

- No built-in schema validation (can be added by implementations)
- No import/include mechanism for multi-file documents (planned for future)
- AICQL does not support complex JOIN operations yet
- No binary data support (text only)

### Future Considerations

Features being considered for future versions:
- Schema validation directives
- Import/include statements for modular documents
- Extended AICQL with JOIN support
- Additional block types (GRAPH, MATRIX, etc.)
- Type annotations for fields
- Binary data encoding support
- Namespace support for large projects

### Credits

- **Format Design**: Karay Akar
- **Initial Implementation**: GitHub Copilot
- **License**: MIT

### Community

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Release Instructions for Maintainers

When preparing a new release:

1. Update version numbers in documentation
2. Update this CHANGELOG.md with new features/fixes
3. Review all examples for accuracy
4. Update README.md if needed
5. Tag the release: `git tag -a v1.0.0 -m "Release 1.0.0"`
6. Push tags: `git push origin --tags`
7. Create GitHub release with release notes

## Version History

- **1.0.0** (2024-11-23): Initial stable release

[1.0.0]: https://github.com/karayakar/AICML/releases/tag/v1.0.0
