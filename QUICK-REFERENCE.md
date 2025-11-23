# AI:CML Quick Reference Guide

A one-page reference for AI:CML (LLX) 1.0 syntax and features.

## File Format

- **Extensions**: `.aicml` or `.llx`
- **Encoding**: UTF-8
- **Line Endings**: LF or CRLF
- **MIME Type**: `text/x-aicml` or `text/x-llx`

## Basic Syntax

### Comments
```aicml
# This is a comment
# Comments start with # at the beginning of a line
```

### Narrative Sections
```aicml
This is free-form narrative text for humans.
It continues until a block delimiter is encountered.
```

## Block Types

### ENTITY - Rich Records
```aicml
ENTITY EntityName
  field1: value1
  field2: value2
  field3: @alias_reference
  tags: tag1, tag2, tag3
END
```

### DICT - Key-Value Pairs
```aicml
DICT DictName
  key1: value1
  key2: value2
  key3: value3
END
```

### SET - Unique Collections
```aicml
SET SetName
  value1
  value2
  value3
END
```

### ROW - Tabular Data
```aicml
ROW TableName
  | Column1 | Column2 | Column3 |
  | value1  | value2  | value3  |
  | value4  | value5  | value6  |
END
```

### QUERY - AICQL Queries
```aicml
QUERY QueryName
  SELECT field1, field2
  FROM ENTITY EntityName
  WHERE field1 > 100
  ORDER BY field2 DESC
  LIMIT 10
END
```

## Alias System

### Defining Aliases
```aicml
DICT aliases
  shortname: Full Description or Value
  electronics: Electronic Devices and Accessories
  wireless: Wireless Connectivity
END
```

### Using Aliases
```aicml
ENTITY Product
  category: @electronics
  features: @wireless
END
```

**Note**: Alias references use `@` prefix

## AICQL Query Language

### SELECT Clause
```aicml
SELECT *                          # All fields
SELECT field1, field2             # Specific fields
SELECT COUNT(*)                   # Count records
SELECT SUM(price), AVG(rating)    # Aggregations
SELECT price * quantity as total  # Arithmetic expressions
SELECT (revenue - cost) / cost * 100 as margin_pct  # Complex calculations
```

### FROM Clause
```aicml
FROM ENTITY                       # All entities
FROM ENTITY EntityName            # Specific entity
FROM DICT DictName                # From dictionary
FROM SET SetName                  # From set
FROM ROW TableName                # From table
```

### WHERE Clause

**Comparison Operators**:
```aicml
WHERE field = value               # Equal
WHERE field != value              # Not equal
WHERE field > value               # Greater than
WHERE field < value               # Less than
WHERE field >= value              # Greater or equal
WHERE field <= value              # Less or equal
```

**Arithmetic Operators**:
```aicml
SELECT price * quantity as total  # Multiplication
SELECT revenue - cost as profit   # Subtraction
SELECT price + tax as final       # Addition
SELECT total / count as average   # Division
SELECT value % 10 as remainder    # Modulo
```

**Logical Operators**:
```aicml
WHERE condition1 AND condition2   # Both true
WHERE condition1 OR condition2    # Either true
WHERE NOT condition               # Negation
```

**String Operators**:
```aicml
WHERE field CONTAINS value        # Contains substring
WHERE field STARTS_WITH value     # Starts with
WHERE field ENDS_WITH value       # Ends with
```

**Set Operators**:
```aicml
WHERE field IN (val1, val2)       # In list
WHERE field NOT IN (val1, val2)   # Not in list
```

### GROUP BY Clause
```aicml
GROUP BY field1                   # Group by single field
GROUP BY field1, field2           # Group by multiple
```

### ORDER BY Clause
```aicml
ORDER BY field ASC                # Ascending (default)
ORDER BY field DESC               # Descending
```

### LIMIT Clause
```aicml
LIMIT 10                          # Return max 10 results
```

## AICQL Functions

### Aggregation Functions
- `COUNT()` - Count records
- `SUM(field)` - Sum numeric values
- `AVG(field)` - Average value
- `MIN(field)` - Minimum value
- `MAX(field)` - Maximum value
- `DISTINCT(field)` - Unique values

### Example Usage
```aicml
QUERY Statistics
  SELECT 
    category,
    COUNT(*) as total,
    AVG(price) as avg_price,
    SUM(stock) as total_stock
  FROM ENTITY Product
  GROUP BY category
  ORDER BY total DESC
END
```

## Keywords (Case-Insensitive)

### Block Keywords
- `ENTITY` - Rich record block
- `DICT` - Dictionary block
- `SET` - Set block
- `ROW` - Table block
- `QUERY` - Query block
- `END` - Block terminator

### Query Keywords
- `SELECT` - Select fields
- `FROM` - Data source
- `WHERE` - Filter condition
- `GROUP BY` - Group results
- `ORDER BY` - Sort results
- `LIMIT` - Limit count
- `AND` - Logical AND
- `OR` - Logical OR
- `NOT` - Logical NOT
- `IN` - Set membership
- `ASC` - Ascending sort
- `DESC` - Descending sort

## Complete Example

```aicml
# Product Catalog

This catalog demonstrates all AI:CML features.

DICT aliases
  electronics: Electronic Devices and Accessories
  wireless: Wireless Connectivity
END

DICT categories
  kbd: Keyboards
  mouse: Pointing Devices
END

ENTITY Product_001
  id: KB-001
  name: Wireless Keyboard
  category: @kbd
  features: @wireless
  price: 99.99
  stock: 42
END

ENTITY Product_002
  id: MS-001
  name: Gaming Mouse
  category: @mouse
  price: 59.99
  stock: 67
END

ROW SalesData
  | ProductID | Quantity | Revenue |
  | KB-001    | 120      | 11998.80 |
  | MS-001    | 200      | 11998.00 |
END

SET AvailableColors
  Black
  White
  Silver
END

QUERY HighValueProducts
  SELECT name, price, stock
  FROM ENTITY Product
  WHERE price > 50 AND stock > 30
  ORDER BY price DESC
END

QUERY TotalSales
  SELECT SUM(Revenue) as total
  FROM ROW SalesData
END

This catalog includes 2 products with full sales data.
```

## Best Practices

### Naming
✅ Use descriptive names: `Product_001`, `CustomerList`  
❌ Avoid: `e1`, `data`, `thing`

### Aliases
✅ Use for repeated terms (5+ times)  
✅ Keep names short but meaningful  
❌ Don't alias everything

### Indentation
✅ Use 2 spaces for readability  
✅ Be consistent throughout  
❌ Not required but recommended

### Comments
✅ Explain complex sections  
✅ Provide context for data  
❌ Don't over-comment obvious things

### Queries
✅ One query, one question  
✅ Descriptive query names  
❌ Avoid SELECT * when possible

## Common Patterns

### Master-Detail
```aicml
ENTITY Order_001
  id: ORD-001
  total: 299.97
END

ENTITY OrderItem_001
  order_id: ORD-001
  product: PRD-001
  quantity: 3
END
```

### Hierarchical
```aicml
DICT categories
  electronics: Electronic Devices
  electronics.computers: Computer Systems
  electronics.computers.laptops: Laptops
END
```

### Temporal
```aicml
ROW DailyMetrics
  | Date       | Value |
  | 2024-11-01 | 1200  |
  | 2024-11-02 | 1350  |
END
```

### Tagging
```aicml
ENTITY Product
  tags: @featured, @sale, @new
END

QUERY Tagged
  SELECT name FROM ENTITY Product
  WHERE tags CONTAINS @featured
END
```

## Error Checking

### Common Errors
```aicml
# ❌ Missing END
ENTITY Product
  name: Test

# ❌ Wrong alias syntax
category: electronics  # Should be @electronics

# ❌ Inconsistent columns
ROW Data
  | A | B |
  | 1 | 2 | 3 |  # Too many columns

# ✅ Correct
ENTITY Product
  name: Test
END

category: @electronics

ROW Data
  | A | B | C |
  | 1 | 2 | 3 |
END
```

## Validation Checklist

- [ ] All blocks have matching END keywords
- [ ] Aliases use @ prefix in references
- [ ] ROW blocks have consistent column counts
- [ ] DICT blocks defined before alias usage
- [ ] Queries have SELECT and FROM clauses
- [ ] Field names are consistent
- [ ] File is UTF-8 encoded
- [ ] No syntax errors in queries

## Resources

- **Full Specification**: [SPECIFICATION.md](SPECIFICATION.md)
- **Grammar**: [GRAMMAR.md](GRAMMAR.md)
- **Usage Guide**: [USAGE-GUIDE.md](USAGE-GUIDE.md)
- **Examples**: [examples/](examples/)
- **FAQ**: [FAQ.md](FAQ.md)

---

**AI:CML Version**: 1.0.0  
**Last Updated**: 2024-11-23
