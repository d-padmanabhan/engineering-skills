# Context Management & QA Validation

## Context File Management

Context files are workspace-specific and should be in an `extras/` directory:

### Project Files

| File | Purpose |
|------|---------|
| `project-brief.md` | Project overview for stable, high-level grounding |
| `tasks.md` | Source of truth for current tasks |
| `active-context.md` | Current focus and working context |
| `progress.md` | Implementation status tracking |
| `reflect-*.md` | Review documents per significant task |
| `creative-*.md` | Design decisions for Level 3-4 tasks |

### Best Practices

1. Always check context files before starting work
2. Update context files as you progress
3. Don't skip phases for complex tasks
4. Preserve context between sessions using files
5. Be explicit about which phase you're in

## QA Validation System

**Purpose:** Technical validation to prevent implementation failures.

**When Required:**
- Mandatory before Implementation for Level 2+ tasks
- Can be called explicitly with "QA" command
- After Creative Phase completion (Level 3-4)

### Four-Point Validation Process

#### 1. Dependency Verification
- All required packages/tools installed
- Versions compatible with requirements
- No missing dependencies

#### 2. Configuration Validation
- Configuration files exist and are valid
- Syntax is correct (JSON, YAML, TOML)
- Platform compatibility verified
- Required settings present

#### 3. Environment Validation
- Build tools available
- Permissions sufficient
- Environment variables set
- Required ports available

#### 4. Minimal Build Test
- Build process works
- Core functionality testable
- No blocking errors

### QA Report Format

**Success:**
```
✅ QA VALIDATION STATUS
✓ Dependencies        All required packages installed
✓ Configurations      Format verified for platform
✓ Environment         Suitable for implementation
✓ Build Test          Core functionality verified

✓ VERIFIED - Clear to proceed to Implementation phase
```

**Failure:**
```
❌ QA VALIDATION FAILED

The following issues must be resolved:

1️⃣ DEPENDENCY ISSUES:
- [Description]
- [Recommended fix]

❌ IMPLEMENTATION BLOCKED until resolved.
```

## Creative Phase (Level 3-4 Tasks)

**Mandatory for:**
- Level 3: Complex multi-module features
- Level 4: Architectural changes
- Design decisions affecting multiple components

**Process:**
1. Entry Gate Verification - task complexity confirmed
2. Design Exploration - 2-4 alternatives with pros/cons
3. Exit Gate Verification - all decisions documented
4. Documentation - create `creative-*.md`
5. Proceed to QA Validation

**Implementation Block:**
```
⛔ IMPLEMENTATION BLOCKED
Creative phases MUST be completed before implementation.

Required Creative Phases:
- [ ] [Creative Phase 1]
- [ ] [Creative Phase 2]

🚫 This is a HARD BLOCK
```

## Platform Awareness

**Automatic Detection:**
- Operating system (Windows/macOS/Linux)
- Path separator format (`\` vs `/`)
- Shell environment (PowerShell/Bash/Zsh)

**Command Adaptation:**
- Windows: Use `dir`, `type`, backslash paths
- macOS/Linux: Use `ls`, `cat`, forward slash paths

## Common Workflow Violations

### Violation 1: Starting Without Planning
```diff
- BAD: User asks to refactor auth → AI immediately rewrites code
+ GOOD: AI analyzes current system, proposes options, waits for approval
```

### Violation 2: Adding Unplanned Features
```diff
- BAD: User asks for search bar → AI adds autocomplete, history, filters
+ GOOD: AI adds only basic search bar, suggests enhancements for next phase
```

### Violation 3: Not Stopping at Agreed Scope
```diff
- BAD: User asks to fix button → AI refactors entire CSS
+ GOOD: AI fixes only the button, notes other issues for discussion
```

