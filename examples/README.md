# AI:CML Examples Directory

This directory contains example AI:CML (LLX) documents demonstrating various use cases and features of the format.

## Examples

### 1. simple-project.aicml
**Use Case:** Software project tracking

A minimal example showing basic AI:CML features:
- Team member entities
- Task tracking
- Sprint progress tables
- Simple queries for project insights

**Best for:** Learning the basics of AI:CML syntax

### 2. product-catalog.aicml
**Use Case:** E-commerce product database

A comprehensive example demonstrating:
- Complex product entities with multiple attributes
- Sales data in tabular format
- Multiple dictionary types (aliases, categories, manufacturers)
- Advanced queries with aggregations
- Warehouse inventory tracking

**Best for:** Understanding how to structure catalog data

### 3. research-papers.aicml
**Use Case:** Academic research database

Shows how to organize:
- Author information and affiliations
- Research paper metadata
- Citation networks
- Publication statistics
- Research topic categorization

**Best for:** Academic and bibliographic applications

### 4. medical-records.aicml
**Use Case:** Healthcare data management (fictional data)

Demonstrates:
- Patient records with medical history
- Vital signs and lab results
- Appointment scheduling
- Medical staff information
- Clinical queries for risk assessment

**Best for:** Healthcare and clinical applications

## Common Patterns

All examples demonstrate these key AI:CML features:

1. **Alias System**: Using `@` prefix to reference dictionary-defined terms
2. **Block Types**: ENTITY, DICT, SET, ROW structures
3. **AICQL Queries**: Embedded queries for data retrieval
4. **Narrative Sections**: Human-readable context and documentation
5. **Token Efficiency**: Reducing AI token usage through aliases

## File Extension

All examples use the `.aicml` extension. You can also use `.llx` for AI:CML files.

## Getting Started

1. Open any example file in a text editor
2. Observe the structure and block types
3. Notice how aliases (prefixed with @) are defined and used
4. Examine the AICQL queries to understand data retrieval
5. Use the format as a template for your own documents

## Creating Your Own AI:CML Documents

1. Start with narrative sections describing your domain
2. Define dictionaries for common terms (use aliases to save tokens)
3. Create entities for your main data objects
4. Use ROW blocks for tabular data
5. Use SET blocks for collections
6. Add QUERY blocks for common data retrieval patterns
7. Keep the format human-readable while maintaining structure

## Best Practices

- **Define aliases early**: Place DICT blocks near the top of the document
- **Use meaningful names**: Entity and block names should be descriptive
- **Comment complex sections**: Use `#` for comments when needed
- **Keep queries focused**: Each query should answer one specific question
- **Balance structure and readability**: Don't over-structure simple data

## Token Efficiency

The alias system is particularly valuable for AI processing:

```aicml
# Without aliases (more tokens):
category: Electronic Devices and Accessories
category: Electronic Devices and Accessories

# With aliases (fewer tokens):
DICT aliases
  electronics: Electronic Devices and Accessories
END
category: @electronics
category: @electronics
```

The alias is expanded by AI systems when processing, but the document
itself uses fewer tokens during transmission and storage.
