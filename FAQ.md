# AI:CML Frequently Asked Questions (FAQ)

## General Questions

### What is AI:CML?

AI:CML (AI Content Markup Language), also known as LLX (Large-Language eXchange), is a plain-text, block-oriented format that treats documents like tiny semantic databases. It combines human-readable narrative with structured data blocks and embedded queries.

### Why create a new format?

AI:CML fills a unique niche:
- **More structured than Markdown** but still human-readable
- **More readable than JSON/XML** while maintaining queryability
- **More efficient than plain text** for AI systems (via alias system)
- **Self-contained** with embedded queries (no external database needed)

### Is AI:CML production-ready?

Yes! Version 1.0 is stable and ready for:
- Document authoring
- Data cataloging
- Knowledge bases
- Configuration files
- AI training datasets

### What's the difference between AI:CML and LLX?

They're the same format. AI:CML is the full name, LLX is the short name. Use whichever you prefer.

### How do I pronounce "AI:CML"?

"A-I-C-M-L" (saying each letter) or "AI-camel" (colloquially).

## Format Questions

### What file extension should I use?

Both `.aicml` and `.llx` are supported. Choose based on your preference:
- `.aicml` - More descriptive
- `.llx` - Shorter, easier to type

### Can I mix block types in one document?

Yes! You can use any combination of ENTITY, DICT, SET, ROW, and QUERY blocks in a single document.

### Are block keywords case-sensitive?

No, block keywords (ENTITY, DICT, SET, ROW, QUERY, END) are case-insensitive. However, block names and field names ARE case-sensitive.

```aicml
# All valid:
ENTITY Product_001
entity Product_002
Entity Product_003
```

### Do I need to indent?

No, indentation is optional but strongly recommended for readability. The format uses explicit END keywords to mark block boundaries.

```aicml
# Both valid:

ENTITY Product_001
name: Keyboard
price: 99.99
END

ENTITY Product_002
  name: Mouse
  price: 49.99
END
```

### Can I have comments in AI:CML documents?

Yes! Use `#` at the start of a line for comments:

```aicml
# This is a comment
ENTITY Product_001
  name: Keyboard  # Inline comments not supported
END
```

**Note**: Inline comments (after content on the same line) are not supported.

### How do I handle multi-line values?

Currently, values are single-line only. For multi-line text, consider:

1. Using escaped newlines (implementation-specific)
2. Storing long text in narrative sections
3. Using multiple fields for structured multi-line data

This may be enhanced in future versions.

## Alias Questions

### When should I use aliases?

Use aliases when:
- A term appears 5+ times in your document
- You want to ensure consistent terminology
- Token efficiency is important for AI processing
- You're abbreviating long descriptions

### Can aliases reference other aliases?

Not in version 1.0. Aliases must reference literal values, not other aliases.

```aicml
# Not supported:
DICT aliases
  electronics: @devices  # Can't reference another alias
  devices: Electronic Devices
END

# Supported:
DICT aliases
  electronics: Electronic Devices and Accessories
  devices: Electronic Devices
END
```

### What happens if an alias isn't defined?

If a parser encounters `@undefined_alias` and no matching definition exists in any DICT block, the behavior is implementation-specific:
- Some parsers may leave it as-is
- Others may warn or error
- Some may treat it as a literal string

### Can I use aliases in DICT definitions?

No, DICT values should be literal strings. Use aliases only in ENTITY, SET, ROW blocks, and QUERY conditions.

## Query Questions

### Is AICQL a full SQL implementation?

No, AICQL is a simplified query language inspired by SQL but optimized for document-based queries. It supports:
- SELECT, FROM, WHERE, GROUP BY, ORDER BY, LIMIT
- Basic operators and functions
- Single-source queries

It does NOT support:
- Complex JOINs (may be added in future)
- Subqueries (may be added in future)
- CREATE, UPDATE, DELETE operations
- Transactions

### Can queries modify data?

No, AICQL is read-only. It's designed for querying and analyzing data, not modifying it. To change data, edit the document directly.

### How do I execute AICQL queries?

AICQL queries are embedded in the document for documentation purposes. To execute them:
1. Use an AI:CML parser with AICQL support
2. Build your own query engine
3. Use them as documentation of query intent (AI systems can interpret them)

### Can I query across multiple documents?

Not in version 1.0. Queries operate on the document they're defined in. Multi-document queries may be added in future versions.

## Performance Questions

### How much does the alias system save?

Example comparison:

**Without aliases** (~180 tokens for AI processing):
```aicml
category: Electronic Devices and Accessories
category: Electronic Devices and Accessories
category: Electronic Devices and Accessories
```

**With aliases** (~100 tokens):
```aicml
DICT aliases
  electronics: Electronic Devices and Accessories
END
category: @electronics
category: @electronics
category: @electronics
```

Savings increase with more usage of the alias.

### What's the recommended document size?

- **Small** (< 10KB): Optimal for single-file documents
- **Medium** (10-100KB): Still manageable, consider organization
- **Large** (> 100KB): Consider splitting into multiple files

### Are AI:CML documents efficient to parse?

Yes! The format is designed for efficient parsing:
- Linear structure (top to bottom)
- Clear block boundaries (END keywords)
- No complex nesting
- Simple tokenization

## Implementation Questions

### Are there existing parsers?

The format is new (v1.0 released Nov 2024). Parsers are being developed. Check the repository for updates on:
- Reference implementations
- Community parsers
- Editor plugins

### How hard is it to build a parser?

Relatively easy! AI:CML has a simple grammar:
- **Basic parser**: A few hundred lines of code
- **With validation**: Add a few hundred more
- **With AICQL**: More complex but still manageable

See [GRAMMAR.md](GRAMMAR.md) for formal specification.

### What programming languages work best?

Any language works! AI:CML is language-agnostic. Consider:
- **Python**: Great for AI/ML integration
- **JavaScript/Node**: Good for web tools
- **Go**: Excellent for CLI tools
- **Rust**: Best for high-performance parsers
- **Java/C#**: Solid for enterprise use

### Can I extend the format?

For personal use, yes! For public specifications, please:
1. Open an issue to discuss
2. Follow the contribution guidelines
3. Maintain backward compatibility
4. Update the specification

## Use Case Questions

### When should I use AI:CML vs. JSON?

Use **AI:CML** when:
- Humans need to read/edit regularly
- You want embedded queries
- Token efficiency matters
- You need narrative context

Use **JSON** when:
- Purely machine-to-machine
- Strict schema validation needed
- Existing JSON tooling required

### When should I use AI:CML vs. Markdown?

Use **AI:CML** when:
- You need structured, queryable data
- Alias system benefits you
- You want type-specific blocks

Use **Markdown** when:
- Pure documentation (no structured data)
- Maximum simplicity needed
- Standard Markdown tooling required

### When should I use AI:CML vs. CSV?

Use **AI:CML** when:
- You need multiple data types
- You want context and queries
- Data relationships matter

Use **CSV** when:
- Simple tabular data only
- Spreadsheet compatibility needed
- Minimal structure required

### Can I use AI:CML for configuration files?

Yes! AI:CML works great for configuration:

```aicml
# Application Configuration

DICT database
  host: localhost
  port: 5432
  name: myapp_db
END

DICT features
  auth: enabled
  cache: enabled
  debug: disabled
END

SET allowed_origins
  https://example.com
  https://app.example.com
END
```

### Can I use AI:CML for AI training data?

Absolutely! The format is designed for AI systems:

```aicml
# Training Dataset - Customer Sentiment

ENTITY Sample_001
  text: Great product, very satisfied!
  sentiment: positive
  confidence: 0.95
END

ENTITY Sample_002
  text: Disappointed with quality
  sentiment: negative
  confidence: 0.88
END
```

## Migration Questions

### How do I migrate from JSON?

See the [USAGE-GUIDE.md](USAGE-GUIDE.md#migration-guide) for detailed examples. General approach:
- Objects → ENTITY blocks
- Arrays of objects → Multiple ENTITYs or ROW blocks
- Simple arrays → SET blocks
- Key-value maps → DICT blocks

### Can I auto-convert from other formats?

Potentially! Basic conversions are straightforward:
- JSON → AI:CML (mostly automated)
- CSV → AI:CML ROW blocks (fully automated)
- YAML → AI:CML (mostly automated)

Consider building conversion tools or use existing ones (check repository).

### Will my documents work with future versions?

AI:CML follows semantic versioning:
- **Patch** (1.0.x): Bug fixes, fully compatible
- **Minor** (1.x.0): New features, backward compatible
- **Major** (2.0.0): May include breaking changes

Version 1.x documents should work with any 1.x parser.

## Troubleshooting

### My parser says "unexpected END"

Check that:
1. Each block has exactly one END
2. Block names are correctly specified
3. No extra END keywords

### Aliases aren't being recognized

Verify:
1. Aliases are defined in DICT blocks
2. References use `@` prefix
3. DICT blocks appear before usage (recommended)
4. Your parser supports alias expansion

### Queries aren't working

Remember:
1. Queries are documentation/specification
2. You need a query engine to execute them
3. They don't automatically run
4. Check your implementation's AICQL support

## Getting Help

- **Documentation**: Read [SPECIFICATION.md](SPECIFICATION.md)
- **Examples**: Check [examples/](examples/)
- **Issues**: Open an issue on GitHub
- **Community**: Join discussions

## Contributing

Have a question not answered here? 

1. Check existing documentation
2. Search closed issues
3. Open a new issue
4. Consider adding it to this FAQ!

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

---

**Last Updated**: 2024-11-23  
**Version**: 1.0.0
