# Contributing to Claude Engineering Skills

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

---

## How to Contribute

### 1. Report Issues

Found a bug or have a suggestion?

- **Check existing issues** first to avoid duplicates
- **Use issue templates** when available
- **Provide context**: What were you trying to do? What happened? What did you expect?
- **Include examples**: Code snippets, error messages, screenshots

### 2. Suggest New Skills

Want to add a new skill area or reference?

**Before creating a new skill:**

- Check if it fits within an existing skill directory
- Ensure it follows the project's structure and style
- Consider if it's broadly applicable (not too specific)

**Structure for new skills:**

```
skill-name/
├── SKILL.md              # Main skill file
└── references/
    ├── pattern-one.md    # Reference document
    └── pattern-two.md    # Reference document
```

### 3. Improve Existing Skills

Enhancement ideas:

- Add more examples
- Clarify confusing sections
- Add common mistakes/anti-patterns
- Update for newer tool versions
- Fix typos or formatting

### 4. Add Code Examples

High-quality examples should:

- Be production-ready (not toy examples)
- Show best practices
- Include error handling
- Be well-commented
- Follow the skill's standards

---

## Pull Request Process

### Before Submitting

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/your-feature-name`
3. **Make your changes**
4. **Run pre-commit hooks**: `pre-commit run --all-files`
5. **Test your changes**: Ensure examples work and formatting is correct
6. **Commit with conventional commits**: `feat: add rust async patterns`

### Pre-commit Setup

```bash
# Install pre-commit hooks (run once after cloning)
pre-commit install

# Run hooks on all files
pre-commit run --all-files

# Update hooks to latest versions (run periodically)
pre-commit autoupdate
```

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**

- `feat`: New feature or skill
- `fix`: Bug fix or correction
- `docs`: Documentation changes
- `refactor`: Code restructuring without behavior change
- `style`: Formatting, whitespace
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**

```
feat(python): add asyncio best practices
fix(terraform): correct state locking example
docs(readme): update installation instructions
refactor(go): reorganize concurrency section
```

### PR Guidelines

- **One PR per feature/fix**: Keep changes focused
- **Clear title and description**: Explain what and why
- **Reference issues**: Use `Fixes #123` or `Relates to #456`
- **Add examples**: Show before/after if applicable

---

## Project Structure

```
dp-claude-engineering-skills/
├── agent-workflow/           # AI agent patterns
├── bash-shell-scripting/     # Shell scripting skills
├── cicd-github-actions/      # CI/CD patterns
├── cloud-platforms/          # AWS, GCP, Azure, Cloudflare
├── core-engineering/         # Git, code review, standards
├── database-postgresql/      # Database patterns
├── documentation-standards/  # Markdown, docs
├── go-rust-systems/          # Systems programming
├── infrastructure-iac/       # Terraform, Ansible, Docker
├── kubernetes-containers/    # K8s, Helm
├── mcp-development/          # MCP server development
├── python-development/       # Python patterns
├── security-testing/         # Security, testing
├── typescript-javascript/    # Frontend, Node.js
├── README.md                 # Main documentation
└── LICENSE                   # MIT License
```

---

## Code Quality Standards

### Skill File Standards

1. **Clear Structure**: Use SKILL.md as main file, references/ for details
2. **Markdown Formatting**: Use headers, code blocks, lists consistently
3. **Code Examples**: Test examples work correctly
4. **Language**: Clear, concise, professional
5. **Length**: Balance comprehensiveness with readability

### Code Example Standards

```markdown
# GOOD - Clear, complete, explained
\```python
def calculate_total(items: list[Item]) -> Decimal:
    """Calculate total with proper error handling."""
    if not items:
        raise ValueError("Items list cannot be empty")

    return sum(item.price for item in items)
\```

# BAD - No error handling
\```python
def calculate_total(items):
    return sum(item.price for item in items)
\```
```

---

## Testing Your Changes

### Validate Markdown

```bash
# Install markdownlint
npm install -g markdownlint-cli

# Check formatting
markdownlint **/*.md
```

### Validate Code Examples

Ensure all code examples:

- Use correct syntax for the language
- Include necessary imports
- Follow the skill's own standards
- Are complete enough to understand

---

## Areas for Contribution

### High Priority

- [ ] Java best practices
- [ ] C# / .NET patterns
- [ ] Ruby on Rails patterns
- [ ] PHP modern patterns
- [ ] More cloud platform patterns (Oracle Cloud, IBM Cloud)

### Medium Priority

- [ ] More testing patterns (property-based, contract, chaos)
- [ ] Performance benchmarking guides
- [ ] Database migration patterns
- [ ] CI/CD patterns for other platforms (GitLab, CircleCI, Jenkins)

---

## Questions?

- **GitHub Discussions**: For general questions and discussions
- **GitHub Issues**: For specific bugs or feature requests

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all.

### Our Standards

**Positive behavior includes:**

- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community

**Unacceptable behavior includes:**

- Trolling, insulting/derogatory comments, and personal attacks
- Public or private harassment
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to Claude Engineering Skills!**

Your contributions help developers worldwide write better, more maintainable code.
