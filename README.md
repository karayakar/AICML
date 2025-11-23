# AI:CML 1.0 — AI Content Markup Language

**Also known as LLX (Large-Language eXchange)**

AI:CML is a plain-text, block-oriented format that treats a document like a tiny semantic database. It combines narrative sections for humans with structured data blocks (ENTITY, DICT, SET, ROW) and inline AICQL queries, allowing AI systems to ask structured questions without leaving the file.

## 🎯 Key Features

- **📝 Plain-text format** - Easy to read, write, and version control
- **🧱 Block-oriented structure** - Clear separation of content types
- **🗄️ Semantic database** - Documents contain queryable structured data
- **⚡ Token-efficient** - Built-in dictionary system reduces AI token usage
- **🔍 Embedded queries** - AICQL queries enable in-document data retrieval
- **🤖 AI-friendly** - Designed for optimal AI system interaction

## 🚀 Quick Start

### Basic Example

```aicml
# Product Catalog

DICT aliases
  electronics: Electronic Devices and Accessories
  wireless: Wireless Connectivity
END

ENTITY Product_001
  name: Wireless Keyboard
  category: @electronics
  features: @wireless
  price: 79.99
  stock: 42
END

QUERY AvailableProducts
  SELECT name, price, stock
  FROM ENTITY
  WHERE stock > 0
  ORDER BY price ASC
END
```

## 📚 Block Types

### ENTITY - Rich Records
```aicml
ENTITY Product_001
  id: KB-001
  name: Mechanical Keyboard
  price: 149.99
  tags: @mechanical, @wireless
END
```

### DICT - Key-Value Pairs
```aicml
DICT categories
  electronics: Electronic Devices
  furniture: Home Furniture
END
```

### SET - Unique Collections
```aicml
SET Colors
  Red
  Blue
  Green
END
```

### ROW - Tabular Data
```aicml
ROW SalesData
  | Product | Quantity | Revenue |
  | KB-001 | 120 | 17998.80 |
  | MS-001 | 200 | 5999.00 |
END
```

## 🔍 AICQL - Query Language

AICQL enables structured queries embedded directly in documents:

```aicml
QUERY HighValueProducts
  SELECT name, price, stock
  FROM ENTITY
  WHERE price > 100 AND stock > 20
  ORDER BY price DESC
  LIMIT 10
END
```

**Supported Operations:**
- **Comparison**: `=`, `!=`, `>`, `<`, `>=`, `<=`
- **Logical**: `AND`, `OR`, `NOT`
- **String**: `CONTAINS`, `STARTS_WITH`, `ENDS_WITH`
- **Set**: `IN`, `NOT IN`
- **Aggregation**: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`

## 💡 Alias System - Token Efficiency

Define terms once, reuse everywhere:

```aicml
DICT aliases
  electronics: Electronic Devices and Accessories
  mechanical: Mechanical Switch Technology
END

# Use with @ prefix
category: @electronics
features: @mechanical
```

**Benefits:**
- Reduces token usage for AI processing
- Maintains consistency across documents
- Improves readability with short references

## 📖 Documentation

- **[SPECIFICATION.md](SPECIFICATION.md)** - Complete format specification
- **[examples/](examples/)** - Example documents for various use cases
  - `simple-project.aicml` - Basic project tracking
  - `product-catalog.aicml` - E-commerce catalog
  - `research-papers.aicml` - Academic database
  - `medical-records.aicml` - Healthcare records

## 🎓 Use Cases

- **Data Catalogs** - Product databases, inventories, asset management
- **Knowledge Bases** - Documentation, wikis, research collections
- **Configuration** - Application settings, feature flags, environment configs
- **Analytics** - Dataset definitions with embedded analysis queries
- **AI Training Data** - Structured datasets for machine learning
- **Project Management** - Task tracking, sprint planning, team coordination

## 📄 File Format

- **Extension**: `.aicml` or `.llx`
- **MIME Type**: `text/x-aicml` or `text/x-llx`
- **Encoding**: UTF-8
- **Line Endings**: LF or CRLF

## 🛠️ Syntax Rules

1. Block keywords (ENTITY, DICT, SET, ROW, QUERY, END) are case-insensitive
2. Block names and field names are case-sensitive
3. Alias references use `@` prefix (e.g., `@electronics`)
4. Comments start with `#` at the beginning of a line
5. Indentation is optional but recommended for readability
6. Empty lines between blocks are optional

## 🌟 Why AI:CML?

### For Humans
- **Readable**: Plain text format, familiar structure
- **Writable**: Simple syntax, no complex markup
- **Maintainable**: Easy to edit, version control friendly

### For AI Systems
- **Structured**: Clear data organization for parsing
- **Queryable**: Built-in query language (AICQL)
- **Efficient**: Alias system reduces token usage
- **Self-contained**: Queries and data in one file

### For Both
- **Flexible**: Supports multiple data types and structures
- **Scalable**: Works for small configs to large databases
- **Extensible**: Easy to add new block types or fields

## 🤝 Contributing

AI:CML is an open format. Contributions, suggestions, and implementations are welcome!

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Version

**AI:CML 1.0** - Current stable version

---

**AI:CML (LLX)** - Making documents smarter, one block at a time.
