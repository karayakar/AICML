# AI:CML (LLX) 1.0 Specification

**AI Content Markup Language** - Also known as **LLX (Large-Language eXchange)**

## Overview

AI:CML 1.0 is a plain-text, block-oriented format that treats a document like a tiny semantic database. It combines human-readable narrative sections with structured data blocks and inline queries, enabling AI systems to ask structured questions without leaving the file.

## Core Principles

1. **Plain-text format** - Easy to read, write, and version control
2. **Block-oriented structure** - Clear separation of different content types
3. **Semantic database model** - Documents contain queryable structured data
4. **Token efficiency** - Built-in dictionary blocks reduce AI token usage
5. **Self-contained queries** - AICQL queries embedded in the document

## Block Types

### 1. Narrative Sections

Free-form text for human consumption. Narrative sections start without any special marker and continue until a block delimiter is encountered.

```
This is a narrative section describing the context of the document.
It can span multiple lines and paragraphs.

Another paragraph in the same narrative section.
```

### 2. ENTITY Blocks

Rich record structures for complex objects with named fields.

**Syntax:**
```
ENTITY <entity_name>
  <field_name>: <value>
  <field_name>: <value>
  ...
END
```

**Example:**
```
ENTITY Product_001
  name: Professional Keyboard
  price: 149.99
  category: @electronics
  tags: @mechanical, @wireless, @rgb
  stock: 45
END
```

### 3. DICT Blocks

Dictionary/map structures for key-value pairs. Ideal for configuration, aliases, and compact data.

**Syntax:**
```
DICT <dict_name>
  <key>: <value>
  <key>: <value>
  ...
END
```

**Example:**
```
DICT ProductCategories
  electronics: Electronic Devices and Accessories
  furniture: Home and Office Furniture
  apparel: Clothing and Fashion Items
END
```

### 4. SET Blocks

Unordered collections of unique values.

**Syntax:**
```
SET <set_name>
  <value>
  <value>
  ...
END
```

**Example:**
```
SET AvailableColors
  red
  blue
  green
  black
  white
END
```

### 5. ROW Blocks

Tabular data with column headers and data rows.

**Syntax:**
```
ROW <table_name>
  | <col1> | <col2> | <col3> | ...
  | <val1> | <val2> | <val3> | ...
  | <val1> | <val2> | <val3> | ...
END
```

**Example:**
```
ROW SalesData
  | Product | Quantity | Revenue |
  | Keyboard | 120 | 17998.80 |
  | Mouse | 200 | 5999.00 |
  | Monitor | 45 | 13495.50 |
END
```

## Alias System

The built-in dictionary system allows defining aliases once and reusing them throughout the document, reducing token usage.

**Defining Aliases:**
```
DICT aliases
  electronics: Electronic Devices and Accessories
  mechanical: Mechanical Switch Technology
  wireless: Wireless Connectivity
  rgb: RGB Backlighting
END
```

**Using Aliases:**
- Prefix with `@` to reference an alias: `@electronics`, `@mechanical`
- AI systems expand aliases when processing the document

## AICQL (AI Content Query Language)

AICQL allows structured queries to be embedded directly in the document, enabling AI systems to ask questions about the structured data.

**Syntax:**
```
QUERY <query_name>
  SELECT <fields>
  FROM <block_name>
  WHERE <condition>
  ORDER BY <field> [ASC|DESC]
  LIMIT <number>
END
```

**Example:**
```
QUERY HighValueProducts
  SELECT name, price, stock
  FROM ENTITY
  WHERE price > 100
  ORDER BY price DESC
  LIMIT 10
END
```

**AICQL Operators:**
- Comparison: `=`, `!=`, `>`, `<`, `>=`, `<=`
- Arithmetic: `+`, `-`, `*`, `/`, `%` (modulo)
- Logical: `AND`, `OR`, `NOT`
- String: `CONTAINS`, `STARTS_WITH`, `ENDS_WITH`
- Set: `IN`, `NOT IN`

**AICQL Functions:**
- `COUNT()` - Count matching records
- `SUM()` - Sum numeric values
- `AVG()` - Average of numeric values
- `MIN()` - Minimum value
- `MAX()` - Maximum value
- `DISTINCT()` - Unique values only

## Comments

Comments can be added using `#` at the start of a line:

```
# This is a comment
# Comments are ignored by parsers
```

## Complete Example

```aicml
# AI:CML Document - Product Catalog

This is a product catalog for our electronics store. The catalog
contains various products with their details and pricing information.

DICT aliases
  electronics: Electronic Devices and Accessories
  peripherals: Computer Peripherals
  mechanical: Mechanical Switch Technology
  wireless: Wireless Connectivity
END

DICT categories
  kbd: Keyboards
  mouse: Pointing Devices
  monitor: Display Devices
END

ENTITY Product_001
  id: KB-PRO-001
  name: Professional Mechanical Keyboard
  price: 149.99
  category: @kbd
  tags: @mechanical, @wireless
  stock: 45
  description: Premium mechanical keyboard with wireless connectivity
END

ENTITY Product_002
  id: MS-WRL-001
  name: Wireless Gaming Mouse
  price: 79.99
  category: @mouse
  tags: @wireless
  stock: 120
  description: High-precision wireless gaming mouse
END

ENTITY Product_003
  id: MN-4K-001
  name: 4K Ultra HD Monitor
  price: 399.99
  category: @monitor
  tags: @electronics
  stock: 23
  description: 27-inch 4K Ultra HD monitor with HDR support
END

ROW SalesData
  | ProductID | UnitsSold | Revenue |
  | KB-PRO-001 | 120 | 17998.80 |
  | MS-WRL-001 | 200 | 15998.00 |
  | MN-4K-001 | 45 | 17999.55 |
END

QUERY HighValueProducts
  SELECT name, price, stock
  FROM ENTITY
  WHERE price > 100 AND stock > 20
  ORDER BY price DESC
END

QUERY LowStockAlert
  SELECT name, stock
  FROM ENTITY
  WHERE stock < 30
  ORDER BY stock ASC
END

The above catalog demonstrates the complete AI:CML format with
entities, dictionaries, rows, and queries. The alias system helps
reduce token usage when processing this document with AI systems.
```

## Parsing Rules

1. **Block delimiters** are case-insensitive but conventionally uppercase
2. **Whitespace** is significant for indentation but not for structure
3. **Field names** are case-sensitive
4. **Alias references** must start with `@`
5. **Queries** are executed against the document's structured blocks
6. **Empty lines** between blocks are optional but recommended for readability

## File Extension

Recommended file extension: `.aicml` or `.llx`

## MIME Type

Suggested MIME type: `text/x-aicml` or `text/x-llx`

## Version

This is version 1.0 of the AI:CML specification.
