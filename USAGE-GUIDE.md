# AI:CML Usage Guide and Best Practices

This guide provides practical advice for creating effective AI:CML documents.

## Table of Contents

1. [Getting Started](#getting-started)
2. [Document Structure](#document-structure)
3. [Naming Conventions](#naming-conventions)
4. [Alias Management](#alias-management)
5. [Query Design](#query-design)
6. [Performance Tips](#performance-tips)
7. [Common Patterns](#common-patterns)
8. [Anti-Patterns](#anti-patterns)

## Getting Started

### Creating Your First Document

1. **Start with context**: Begin with a narrative section explaining the document's purpose
2. **Define dictionaries**: Create DICT blocks for common terms
3. **Structure your data**: Use appropriate block types for your data
4. **Add queries**: Include AICQL queries for common data retrieval needs

### Basic Template

```aicml
# Document Title

Brief description of what this document contains and its purpose.

DICT aliases
  term1: Full Description of Term 1
  term2: Full Description of Term 2
END

ENTITY YourEntity_001
  field1: value1
  field2: @term1
END

QUERY YourQuery
  SELECT field1, field2
  FROM ENTITY
END
```

## Document Structure

### Recommended Order

1. **Title and description** (narrative)
2. **Dictionary definitions** (DICT blocks)
3. **Main data structures** (ENTITY, SET, ROW blocks)
4. **Queries** (QUERY blocks)
5. **Closing notes** (narrative)

### Example Structure

```aicml
# Title

Overview and context...

## Dictionaries

DICT aliases
  ...
END

DICT categories
  ...
END

## Data

ENTITY Data_001
  ...
END

## Analysis

QUERY Analysis_001
  ...
END

## Summary

Final thoughts and conclusions...
```

## Naming Conventions

### Block Names

- **ENTITY**: Use descriptive names with optional sequence numbers
  - Good: `Product_001`, `Customer_Smith`, `Order_2024_001`
  - Avoid: `e1`, `thing`, `data`

- **DICT**: Use plural nouns describing the category
  - Good: `aliases`, `categories`, `manufacturers`
  - Avoid: `dict1`, `mydict`, `stuff`

- **SET**: Use plural nouns
  - Good: `AvailableColors`, `SupportedFormats`
  - Avoid: `set1`, `items`

- **ROW**: Use descriptive table names
  - Good: `SalesData_Q1_2024`, `UserMetrics`
  - Avoid: `table1`, `data`

- **QUERY**: Use action-oriented names
  - Good: `FindHighValueCustomers`, `CalculateTotalRevenue`
  - Avoid: `query1`, `q`, `search`

### Field Names

- Use camelCase or snake_case consistently
- Be descriptive but concise
- Good: `totalAmount`, `customer_name`, `createdAt`
- Avoid: `ta`, `cn`, `x`

## Alias Management

### When to Use Aliases

✅ **Use aliases for:**
- Terms repeated multiple times (5+ occurrences)
- Long descriptions that can be abbreviated
- Domain-specific terminology
- Categorical values
- Status codes and types

❌ **Don't use aliases for:**
- Unique values that appear only once
- Simple words (unless standardizing terminology)
- Numeric values
- Identifiers

### Alias Organization

Group related aliases in separate dictionaries:

```aicml
DICT business_terms
  revenue: Total Revenue from Sales
  profit: Net Profit After Expenses
END

DICT status_codes
  active: Currently Active
  inactive: Temporarily Inactive
  archived: Permanently Archived
END

DICT locations
  ny: New York Office
  la: Los Angeles Office
  sf: San Francisco Office
END
```

### Alias Naming

- Use short, memorable identifiers
- Reflect the full term's meaning
- Use consistent prefixes for related terms
- Good: `@electronics`, `@wireless`, `@mechanical`
- Avoid: `@e`, `@term1`, `@x`

## Query Design

### Query Structure

Keep queries focused on a single question:

**Good:**
```aicml
QUERY HighValueCustomers
  SELECT name, total_purchases
  FROM ENTITY Customer
  WHERE total_purchases > 10000
  ORDER BY total_purchases DESC
END
```

**Better - Multiple Focused Queries:**
```aicml
QUERY HighValueCustomers
  SELECT name, total_purchases
  FROM ENTITY Customer
  WHERE total_purchases > 10000
  ORDER BY total_purchases DESC
END

QUERY CustomersByRegion
  SELECT region, COUNT(*)
  FROM ENTITY Customer
  GROUP BY region
END
```

### Query Naming

Use descriptive, action-oriented names:

- **Analysis**: `CalculateAveragePrice`, `FindOutliers`
- **Filtering**: `GetActiveUsers`, `FindExpiredItems`
- **Aggregation**: `SumTotalRevenue`, `CountByCategory`
- **Sorting**: `RankByScore`, `OrderByDate`

### Query Optimization

1. **Filter early**: Place WHERE clauses before ORDER BY
2. **Limit results**: Use LIMIT for large datasets
3. **Use specific fields**: Avoid `SELECT *` when possible
4. **Index-friendly**: Query fields that are commonly used

## Performance Tips

### Token Efficiency

1. **Use aliases for repeated terms**:
   ```aicml
   # Without aliases: ~200 tokens
   category: Electronic Devices and Accessories
   category: Electronic Devices and Accessories
   category: Electronic Devices and Accessories
   
   # With aliases: ~120 tokens
   DICT aliases
     electronics: Electronic Devices and Accessories
   END
   category: @electronics
   category: @electronics
   category: @electronics
   ```

2. **Avoid redundant text**: Don't repeat descriptions in multiple places

3. **Use abbreviations in dictionaries**: Define full terms in DICT, use short refs

### Document Size

- **Small documents** (< 10KB): Single file works well
- **Medium documents** (10-100KB): Consider splitting by domain
- **Large documents** (> 100KB): Split into multiple files with cross-references

### Block Organization

Place frequently accessed data near the top of the document:

1. Critical dictionaries
2. Primary entities
3. Supporting data
4. Historical/archived data
5. Supplementary queries

## Common Patterns

### Pattern 1: Master-Detail Relationship

```aicml
ENTITY Order_001
  id: ORD-001
  customer: CUST-001
  total: 299.97
END

ENTITY OrderItem_001
  order_id: ORD-001
  product: PRD-001
  quantity: 3
  price: 99.99
END

QUERY OrderDetails
  SELECT o.id, oi.product, oi.quantity
  FROM ENTITY Order o, ENTITY OrderItem oi
  WHERE o.id = oi.order_id
END
```

### Pattern 2: Hierarchical Categories

```aicml
DICT categories
  electronics: Electronic Devices
  electronics.computers: Computer Systems
  electronics.computers.laptops: Laptop Computers
  electronics.phones: Mobile Phones
END

ENTITY Product_001
  name: Gaming Laptop
  category: @electronics.computers.laptops
END
```

### Pattern 3: Temporal Data

```aicml
ROW Metrics_Daily
  | Date | Visitors | Conversions | Revenue |
  | 2024-11-01 | 1200 | 45 | 5400.00 |
  | 2024-11-02 | 1350 | 52 | 6240.00 |
END

QUERY WeeklyTrend
  SELECT Date, Visitors, Conversions
  FROM ROW Metrics_Daily
  WHERE Date >= 2024-11-01 AND Date <= 2024-11-07
  ORDER BY Date ASC
END
```

### Pattern 4: Tagging System

```aicml
DICT tags
  featured: Featured Product
  sale: On Sale
  new: New Arrival
  seasonal: Seasonal Item
END

ENTITY Product_001
  name: Winter Jacket
  tags: @featured, @seasonal
END

QUERY FeaturedProducts
  SELECT name, tags
  FROM ENTITY Product
  WHERE tags CONTAINS @featured
END
```

## Anti-Patterns

### ❌ Anti-Pattern 1: Over-Structuring

**Bad:**
```aicml
ENTITY SimpleText_001
  text: Hello
END

ENTITY SimpleText_002
  text: World
END
```

**Good:**
```aicml
Just use narrative text when structure isn't needed:

Hello World
```

### ❌ Anti-Pattern 2: Alias Overuse

**Bad:**
```aicml
DICT aliases
  a: A
  b: B
  c: C
  the: The
END

name: @a simple name
```

**Good:**
```aicml
name: A simple name
```

### ❌ Anti-Pattern 3: Redundant Data

**Bad:**
```aicml
ENTITY Product_001
  name: Keyboard
  name_uppercase: KEYBOARD
  name_lowercase: keyboard
END
```

**Good:**
```aicml
ENTITY Product_001
  name: Keyboard
END

# Transform in queries if needed
QUERY UppercaseNames
  SELECT UPPER(name)
  FROM ENTITY Product
END
```

### ❌ Anti-Pattern 4: Unclear Queries

**Bad:**
```aicml
QUERY Query1
  SELECT *
  FROM ENTITY
  WHERE x > 5
END
```

**Good:**
```aicml
QUERY HighValueProducts
  SELECT name, price, stock
  FROM ENTITY Product
  WHERE price > 5
END
```

### ❌ Anti-Pattern 5: Missing Context

**Bad:**
```aicml
ENTITY E001
  f1: v1
  f2: v2
END
```

**Good:**
```aicml
# Customer Database

This section contains customer records for Q4 2024.

ENTITY Customer_001
  name: John Smith
  email: john@example.com
END
```

## Validation Checklist

Before finalizing your AI:CML document:

- [ ] Document has a clear title and description
- [ ] Dictionaries are defined before they're used
- [ ] Aliases are used consistently (@ prefix)
- [ ] Block names are descriptive and unique
- [ ] Queries have clear, action-oriented names
- [ ] ROW blocks have consistent column counts
- [ ] All END keywords match their opening blocks
- [ ] Comments explain complex sections
- [ ] No redundant or duplicate data
- [ ] File uses UTF-8 encoding

## Tools and Editors

### Text Editors

Any text editor works, but these provide good experience:

- **VS Code**: Syntax highlighting via custom extension
- **Sublime Text**: Custom syntax definitions
- **Vim/Emacs**: Custom syntax files
- **Notepad++**: User-defined language

### Recommended Extensions

- Syntax highlighting for `.aicml` files
- Bracket matching for block delimiters
- Indent guides for nested structures
- Search functionality for @alias references

## Versioning

When versioning AI:CML documents:

1. Use semantic versioning in filenames: `data-v1.0.aicml`
2. Add version metadata as comments:
   ```aicml
   # Version: 1.0.0
   # Last Updated: 2024-11-23
   # Author: Jane Doe
   ```
3. Track changes in git or other version control
4. Document breaking changes in commit messages

## Migration Guide

### From JSON

```json
{
  "products": [
    {"id": 1, "name": "Keyboard", "price": 99.99}
  ]
}
```

```aicml
ENTITY Product_001
  id: 1
  name: Keyboard
  price: 99.99
END
```

### From CSV

```csv
id,name,price
1,Keyboard,99.99
```

```aicml
ROW Products
  | id | name | price |
  | 1 | Keyboard | 99.99 |
END
```

### From YAML

```yaml
categories:
  electronics: Electronic Devices
  furniture: Home Furniture
```

```aicml
DICT categories
  electronics: Electronic Devices
  furniture: Home Furniture
END
```

## Further Reading

- [SPECIFICATION.md](SPECIFICATION.md) - Complete format specification
- [GRAMMAR.md](GRAMMAR.md) - Formal grammar definition
- [examples/](examples/) - Example documents
