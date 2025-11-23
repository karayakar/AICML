# AI:CML Grammar Specification

This document provides a formal grammar specification for AI:CML (LLX) 1.0.

## Notation

This grammar uses Extended Backus-Naur Form (EBNF) notation:

- `::=` means "is defined as"
- `|` means "or"
- `[]` means optional
- `{}` means zero or more repetitions
- `()` means grouping
- `"..."` means literal text

## Top-Level Grammar

```ebnf
document ::= { block | narrative | comment | empty_line }

block ::= entity_block
        | dict_block
        | set_block
        | row_block
        | query_block

narrative ::= text_line { text_line }

comment ::= "#" text newline

empty_line ::= newline
```

## Block Structures

### ENTITY Block

```ebnf
entity_block ::= "ENTITY" whitespace identifier newline
                 { field_line }
                 "END" newline

field_line ::= [ whitespace ] identifier ":" whitespace value newline

value ::= simple_value
        | alias_reference
        | value_list

simple_value ::= text

alias_reference ::= "@" identifier

value_list ::= value { "," whitespace value }
```

### DICT Block

```ebnf
dict_block ::= "DICT" whitespace identifier newline
               { dict_entry }
               "END" newline

dict_entry ::= [ whitespace ] identifier ":" whitespace text newline
```

### SET Block

```ebnf
set_block ::= "SET" whitespace identifier newline
              { set_item }
              "END" newline

set_item ::= [ whitespace ] value newline
```

### ROW Block

```ebnf
row_block ::= "ROW" whitespace identifier newline
              header_row
              { data_row }
              "END" newline

header_row ::= [ whitespace ] "|" column_name { "|" column_name } "|" newline

data_row ::= [ whitespace ] "|" cell_value { "|" cell_value } "|" newline

column_name ::= whitespace identifier whitespace

cell_value ::= whitespace value whitespace
```

### QUERY Block (AICQL)

```ebnf
query_block ::= "QUERY" whitespace identifier newline
                { query_clause }
                "END" newline

query_clause ::= select_clause
               | from_clause
               | where_clause
               | group_by_clause
               | order_by_clause
               | limit_clause

select_clause ::= [ whitespace ] "SELECT" whitespace field_list newline

from_clause ::= [ whitespace ] "FROM" whitespace source_name newline

where_clause ::= [ whitespace ] "WHERE" whitespace condition newline

group_by_clause ::= [ whitespace ] "GROUP BY" whitespace field_list newline

order_by_clause ::= [ whitespace ] "ORDER BY" whitespace field_name [ whitespace sort_order ] newline

limit_clause ::= [ whitespace ] "LIMIT" whitespace number newline

field_list ::= field_name { "," whitespace field_name }
             | function_call { "," whitespace function_call }
             | "*"

field_name ::= identifier [ "." identifier ]

source_name ::= "ENTITY" [ whitespace identifier ]
              | "DICT" [ whitespace identifier ]
              | "SET" [ whitespace identifier ]
              | "ROW" whitespace identifier

condition ::= simple_condition
            | condition whitespace logical_operator whitespace condition
            | "NOT" whitespace condition
            | "(" condition ")"

simple_condition ::= field_name whitespace comparison_operator whitespace value
                   | field_name whitespace string_operator whitespace value
                   | field_name whitespace set_operator whitespace "(" value_list ")"

comparison_operator ::= "=" | "!=" | ">" | "<" | ">=" | "<="

logical_operator ::= "AND" | "OR"

string_operator ::= "CONTAINS" | "STARTS_WITH" | "ENDS_WITH"

set_operator ::= "IN" | "NOT IN"

function_call ::= function_name "(" [ field_name ] ")"

function_name ::= "COUNT" | "SUM" | "AVG" | "MIN" | "MAX" | "DISTINCT"

sort_order ::= "ASC" | "DESC"
```

## Lexical Elements

```ebnf
identifier ::= letter { letter | digit | "_" | "-" }

letter ::= "a"..."z" | "A"..."Z"

digit ::= "0"..."9"

number ::= digit { digit } [ "." digit { digit } ]

text ::= { character - newline }

character ::= any_unicode_character

whitespace ::= { " " | "\t" }

newline ::= "\n" | "\r\n"
```

## Keywords

The following keywords are reserved (case-insensitive in block headers):

- `ENTITY`
- `DICT`
- `SET`
- `ROW`
- `QUERY`
- `END`
- `SELECT`
- `FROM`
- `WHERE`
- `GROUP BY`
- `ORDER BY`
- `LIMIT`
- `AND`
- `OR`
- `NOT`
- `IN`
- `ASC`
- `DESC`

## Operator Precedence (AICQL)

From highest to lowest:

1. Function calls: `COUNT()`, `SUM()`, etc.
2. Comparison operators: `=`, `!=`, `>`, `<`, `>=`, `<=`
3. String operators: `CONTAINS`, `STARTS_WITH`, `ENDS_WITH`
4. Set operators: `IN`, `NOT IN`
5. NOT
6. AND
7. OR

## Comments

Comments start with `#` at the beginning of a line and continue until the end of the line.

```ebnf
comment ::= "#" text newline
```

Comments are ignored by parsers and do not affect the document structure.

## Whitespace Rules

1. **Indentation**: Optional but recommended for readability (2 spaces is conventional)
2. **Block delimiters**: Must start at the beginning of a line or after optional whitespace
3. **Field separators**: Colon (`:`) in ENTITY and DICT blocks must be followed by at least one space
4. **ROW separators**: Pipe (`|`) in ROW blocks can have surrounding whitespace
5. **Empty lines**: Allowed between blocks and within narrative sections

## Alias Resolution

Alias references (e.g., `@electronics`) are resolved by:

1. Searching all DICT blocks in the document
2. Finding a key matching the identifier after `@`
3. Replacing the alias with the corresponding value
4. If no match is found, the alias remains as-is

Aliases can be used in:
- ENTITY field values
- SET items
- ROW cell values
- QUERY conditions and field references

## Type System

AI:CML is dynamically typed. Values are interpreted based on context:

- **Numeric**: Values that parse as numbers (integers or floats)
- **String**: All other values, including alias references
- **Boolean**: The keywords `true` and `false` (case-insensitive)
- **List**: Comma-separated values in a single field
- **Null**: Empty values or the keyword `null`

## Validation Rules

1. Block names must be unique within their block type (e.g., two ENTITYs can have different names)
2. END keywords must match their opening block type
3. QUERY blocks must contain at least a SELECT and FROM clause
4. ROW blocks must have consistent column counts across all rows
5. Alias references must use the `@` prefix
6. Field names in QUERY clauses must exist in the referenced blocks

## Example Parse Tree

For the following AI:CML document:

```aicml
DICT aliases
  electronics: Electronic Devices
END

ENTITY Product_001
  name: Keyboard
  category: @electronics
END
```

Parse tree:

```
document
├── dict_block
│   ├── identifier: "aliases"
│   └── dict_entry
│       ├── key: "electronics"
│       └── value: "Electronic Devices"
├── empty_line
└── entity_block
    ├── identifier: "Product_001"
    ├── field_line
    │   ├── key: "name"
    │   └── value: "Keyboard"
    └── field_line
        ├── key: "category"
        └── alias_reference: "@electronics"
```

## Grammar Extension Points

The grammar is designed to be extensible. Future versions may add:

1. New block types (e.g., `GRAPH`, `MATRIX`)
2. Additional AICQL operators and functions
3. Type annotations for fields
4. Schema validation directives
5. Import/include mechanisms for multi-file documents

## Error Handling

Parsers should handle errors gracefully:

1. **Syntax errors**: Report line number and expected vs. actual token
2. **Unmatched blocks**: Report missing END keyword
3. **Invalid aliases**: Warn about unresolved references
4. **Type mismatches**: Warn about incompatible operations in queries
5. **Recovery**: Skip malformed blocks and continue parsing when possible

## Conformance Levels

### Level 1 (Basic)
- Parse all block types
- Support narrative sections
- Handle comments

### Level 2 (Standard)
- Implement alias resolution
- Validate block structure
- Support basic AICQL queries

### Level 3 (Advanced)
- Execute AICQL queries
- Perform type checking
- Optimize query execution

## Character Encoding

AI:CML documents must be encoded in UTF-8 without BOM (Byte Order Mark).

## Line Endings

Both Unix-style (LF: `\n`) and Windows-style (CRLF: `\r\n`) line endings are acceptable. Parsers should normalize line endings during processing.

## Reserved Characters

The following characters have special meaning and should be escaped when used literally:

- `@` (alias prefix)
- `#` (comment marker, at line start)
- `|` (row separator)
- `:` (field separator)

To include these characters literally in values, place them inside quoted strings or escape them with a backslash (implementation-specific).
