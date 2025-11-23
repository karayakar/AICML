# Contributing to AI:CML

Thank you for your interest in contributing to AI:CML (LLX)! This document provides guidelines for contributing to the project.

## Ways to Contribute

### 1. Specification Improvements

- Clarify ambiguous syntax rules
- Propose new block types or features
- Improve grammar definitions
- Enhance documentation

### 2. Examples and Use Cases

- Create example documents for new domains
- Share real-world use cases
- Provide templates for common scenarios
- Document best practices from experience

### 3. Implementation

- Build parsers in various languages
- Create validators and linters
- Develop AICQL query engines
- Build editor plugins and tools

### 4. Documentation

- Fix typos and improve clarity
- Add tutorials and guides
- Translate documentation
- Create video tutorials or blog posts

### 5. Community

- Answer questions in discussions
- Help newcomers get started
- Share your AI:CML documents
- Organize community events

## Getting Started

1. **Read the documentation**:
   - [SPECIFICATION.md](SPECIFICATION.md) - Format specification
   - [GRAMMAR.md](GRAMMAR.md) - Formal grammar
   - [USAGE-GUIDE.md](USAGE-GUIDE.md) - Best practices

2. **Explore examples**:
   - Check out the [examples/](examples/) directory
   - Try creating your own AI:CML documents

3. **Join the community**:
   - Open an issue for discussion
   - Share your ideas and feedback

## Contribution Guidelines

### Specification Changes

For changes to the AI:CML specification:

1. **Open an issue** first to discuss the proposed change
2. **Explain the motivation**: What problem does it solve?
3. **Provide examples**: Show how it would work
4. **Consider compatibility**: Will it break existing documents?
5. **Update documentation**: Include changes to all relevant docs

#### Specification Change Checklist

- [ ] Issue opened with detailed proposal
- [ ] Examples provided showing the new feature
- [ ] Grammar updated (if applicable)
- [ ] Specification document updated
- [ ] Usage guide updated (if applicable)
- [ ] Example documents created/updated
- [ ] Backward compatibility considered
- [ ] Community feedback addressed

### Example Contributions

When contributing example documents:

1. **Choose a clear domain**: Use a well-understood problem space
2. **Follow best practices**: Demonstrate proper AI:CML usage
3. **Add documentation**: Include comments explaining key concepts
4. **Keep it realistic**: Use plausible data and scenarios
5. **Test completeness**: Ensure all examples work as described

#### Example Document Template

```aicml
# Domain Name - AI:CML Example

Brief description of what this example demonstrates and the use case.

DICT aliases
  # Define common terms
END

# Main content with explanatory comments

## Summary

Explanation of what was demonstrated and key takeaways.
```

### Code Contributions

For implementations (parsers, tools, etc.):

1. **Follow language conventions**: Use idiomatic code for your language
2. **Add tests**: Include unit tests and integration tests
3. **Document your code**: Add comments and API documentation
4. **Provide examples**: Show how to use your implementation
5. **License compatibility**: Ensure MIT-compatible licensing

#### Implementation Guidelines

- **Conformance levels**: Aim for Level 2 (Standard) or higher
- **Error handling**: Provide helpful error messages
- **Performance**: Optimize for typical document sizes
- **Extensibility**: Design for future specification additions

### Documentation Improvements

For documentation changes:

1. **Be clear and concise**: Use simple language
2. **Provide examples**: Show, don't just tell
3. **Check spelling and grammar**: Proofread carefully
4. **Test examples**: Ensure all code examples work
5. **Follow existing style**: Match the tone and format

## Pull Request Process

1. **Fork the repository** (for external contributors)
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**: Follow the guidelines above
4. **Test your changes**: Verify everything works
5. **Commit with clear messages**: Describe what and why
6. **Push to your branch**: `git push origin feature/your-feature-name`
7. **Open a pull request**: Provide a clear description

### Pull Request Template

```markdown
## Description
Brief description of changes

## Motivation
Why is this change needed?

## Changes Made
- Change 1
- Change 2

## Checklist
- [ ] Documentation updated
- [ ] Examples added/updated
- [ ] Grammar updated (if applicable)
- [ ] Backward compatible (or breaking changes documented)
- [ ] Tested thoroughly
```

## Commit Message Guidelines

Write clear, descriptive commit messages:

**Good:**
```
Add SET block examples to usage guide

- Added three examples showing SET block usage
- Included examples for different data types
- Updated table of contents
```

**Bad:**
```
update docs
```

### Commit Message Format

```
<type>: <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting changes
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive experience for everyone, regardless of:

- Age, body size, disability, ethnicity, gender identity and expression
- Level of experience, education, socio-economic status
- Nationality, personal appearance, race, religion
- Sexual identity and orientation

### Our Standards

**Positive behavior:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Accepting constructive criticism gracefully
- Focusing on what's best for the community
- Showing empathy towards others

**Unacceptable behavior:**
- Trolling, insulting comments, or personal attacks
- Public or private harassment
- Publishing others' private information
- Any conduct inappropriate in a professional setting

### Enforcement

Report unacceptable behavior by opening an issue or contacting the maintainers. All complaints will be reviewed and investigated promptly and fairly.

## Style Guide

### Documentation Style

- **Headings**: Use sentence case
- **Lists**: Start with capital letter, no ending period
- **Code blocks**: Include language identifier
- **Links**: Use descriptive text, not "click here"
- **Examples**: Label as "Good" or "Bad" when comparing

### AI:CML Style

For AI:CML documents in this repository:

- **Indentation**: 2 spaces
- **Block names**: PascalCase for entities, lowercase for dictionaries
- **Field names**: camelCase or snake_case (be consistent)
- **Comments**: Use `#` for explanatory comments
- **Empty lines**: One line between blocks

## Testing

### Manual Testing

For specification or documentation changes:

1. Read through all changes carefully
2. Check cross-references and links
3. Verify examples are correct and complete
4. Ensure consistent terminology

### Implementation Testing

For code contributions:

1. Unit tests for all functions
2. Integration tests for complete workflows
3. Example documents as test cases
4. Error handling verification
5. Performance benchmarks (for parsers)

## Release Process

The maintainers will:

1. Review and merge pull requests
2. Update version numbers as needed
3. Tag releases following semantic versioning
4. Update CHANGELOG.md
5. Announce releases to the community

## Getting Help

- **Questions**: Open a discussion issue
- **Bugs**: Open a bug report issue
- **Features**: Open a feature request issue
- **Other**: Contact the maintainers

## Recognition

Contributors will be:

- Credited in CHANGELOG.md
- Listed in a CONTRIBUTORS.md file
- Mentioned in release notes
- Thanked in the community

## License

By contributing, you agree that your contributions will be licensed under the MIT License, the same license as the project.

## Resources

### AI:CML Resources

- [Specification](SPECIFICATION.md)
- [Grammar](GRAMMAR.md)
- [Usage Guide](USAGE-GUIDE.md)
- [Examples](examples/)

### External Resources

- [Markdown Guide](https://www.markdownguide.org/)
- [Semantic Versioning](https://semver.org/)
- [Git Best Practices](https://git-scm.com/book/en/v2)

## Questions?

Don't hesitate to ask questions! We're here to help and appreciate your interest in improving AI:CML.

---

Thank you for contributing to AI:CML! 🎉
