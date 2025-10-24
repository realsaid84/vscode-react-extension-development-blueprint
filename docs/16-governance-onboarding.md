# Governance & Onboarding

> **Team processes, contribution guidelines, and developer onboarding**

## Table of Contents
- [Pull Request Process](#pull-request-process)
- [Code Review Guidelines](#code-review-guidelines)
- [Developer Setup](#developer-setup)
- [Architecture Decision Records](#architecture-decision-records)
- [Contributing Guidelines](#contributing-guidelines)

---

## Pull Request Process

### PR Template

```markdown
<!-- .github/pull_request_template.md -->
## Description

<!-- Provide a clear and concise description of the changes -->

## Type of Change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Performance improvement
- [ ] Code refactoring

## Related Issue

Closes #(issue number)

## Changes Made

<!-- List the specific changes in bullet points -->

-
-
-

## Testing

<!-- Describe the tests you ran to verify your changes -->

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] Manual testing performed

## Screenshots (if applicable)

<!-- Add screenshots to demonstrate the changes -->

## Checklist

- [ ] My code follows the style guidelines of this project
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
- [ ] Any dependent changes have been merged and published
- [ ] I have updated the CHANGELOG.md

## Additional Notes

<!-- Any additional information that reviewers should know -->
```

### PR Requirements

**Before Creating PR:**
1. ✅ All tests pass locally
2. ✅ Code coverage ≥85%
3. ✅ ESLint passes with no errors
4. ✅ Prettier formatting applied
5. ✅ TypeScript compiles without errors
6. ✅ Conventional commit messages used
7. ✅ Documentation updated

**PR Size Guidelines:**
- ✅ Small: < 200 lines changed
- ⚠️ Medium: 200-500 lines changed
- ❌ Large: > 500 lines (break into smaller PRs)

---

## Code Review Guidelines

### Reviewer Checklist

```markdown
## Code Quality

- [ ] Code is readable and well-structured
- [ ] Follows project style guide (Airbnb)
- [ ] No code smells or anti-patterns
- [ ] DRY principle applied
- [ ] SOLID principles followed
- [ ] No hardcoded values (use constants)
- [ ] Proper error handling implemented

## Functionality

- [ ] Changes address the stated requirements
- [ ] No breaking changes (or properly documented)
- [ ] Edge cases handled
- [ ] No console.log statements left in code
- [ ] No commented-out code

## Testing

- [ ] Unit tests included
- [ ] Tests cover edge cases
- [ ] Integration tests if needed
- [ ] Tests are meaningful (not just for coverage)
- [ ] Test names are descriptive

## Security

- [ ] No security vulnerabilities introduced
- [ ] User input validated
- [ ] No sensitive data exposed
- [ ] Dependencies are trusted

## Performance

- [ ] No unnecessary re-renders
- [ ] Memoization used where appropriate
- [ ] No N+1 queries
- [ ] Large lists virtualized

## Documentation

- [ ] JSDoc comments for public APIs
- [ ] README updated if needed
- [ ] CHANGELOG updated
- [ ] Architecture Decision Record if needed
```

### Review Comments

**✅ Good Comments:**
```
- "Consider using useMemo here to prevent unnecessary recalculations"
- "This could be simplified using destructuring"
- "Great use of TypeScript discriminated unions!"
- "Have you considered the case where user is null?"
```

**❌ Bad Comments:**
```
- "This is wrong" (not constructive)
- "I don't like this" (subjective without reasoning)
- "Just use X" (no explanation why)
```

### Review Timeline

- **Initial Review**: Within 24 hours
- **Follow-up**: Within 4 hours
- **Approval**: 2+ reviewers required
- **Merge**: After all checks pass + approvals

---

## Developer Setup

### Prerequisites

```bash
# Required
- Node.js 18+ (use nvm)
- npm 9+
- Git 2.30+
- VS Code 1.80+

# Recommended
- VS Code Extensions:
  - ESLint
  - Prettier
  - TypeScript + JavaScript Language Features
  - Error Lens
  - GitLens
```

### Initial Setup

```bash
# 1. Clone repository
git clone https://github.com/your-org/dapa-vscode-extension.git
cd dapa-vscode-extension

# 2. Install dependencies
npm install

# 3. Setup git hooks
npm run prepare

# 4. Create local environment file
cp .env.example .env

# 5. Run type checking
npm run type-check

# 6. Run tests
npm test

# 7. Start development
npm run watch
```

### VS Code Launch Configuration

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}"
      ],
      "outFiles": [
        "${workspaceFolder}/dist/**/*.js"
      ],
      "preLaunchTask": "npm: watch"
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/dist/test/suite/index"
      ],
      "outFiles": [
        "${workspaceFolder}/dist/test/**/*.js"
      ],
      "preLaunchTask": "npm: watch"
    }
  ]
}
```

### Tasks Configuration

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "watch",
      "problemMatcher": "$tsc-watch",
      "isBackground": true,
      "presentation": {
        "reveal": "never"
      },
      "group": {
        "kind": "build",
        "isDefault": true
      }
    },
    {
      "type": "npm",
      "script": "test",
      "problemMatcher": [],
      "group": {
        "kind": "test",
        "isDefault": true
      }
    }
  ]
}
```

### First Contribution Guide

```markdown
# Making Your First Contribution

## 1. Pick an Issue

- Browse [Good First Issues](https://github.com/your-org/dapa/labels/good%20first%20issue)
- Comment on the issue to claim it
- Wait for maintainer confirmation

## 2. Create Branch

```bash
# From main branch
git checkout -b feature/issue-123-add-feature

# Branch naming convention:
# feature/issue-number-description
# fix/issue-number-description
# docs/issue-number-description
```

## 3. Make Changes

- Follow the style guide
- Write tests
- Update documentation
- Commit frequently with conventional commits

## 4. Test Locally

```bash
# Run all checks
npm run quality
npm run test:all

# Test extension manually
# Press F5 in VS Code to launch extension host
```

## 5. Create Pull Request

- Push your branch
- Create PR using template
- Link related issue
- Request reviews
- Respond to feedback

## 6. After Approval

- Squash commits if requested
- Maintainer will merge
- Your contribution is live! 🎉
```

---

## Architecture Decision Records

### ADR Template

```markdown
# ADR-001: Use Redux Toolkit for State Management

## Status

Accepted

## Context

We need a state management solution for the VSCode extension that:
- Handles complex state with multiple features
- Provides good DevTools integration
- Supports time-travel debugging
- Has excellent TypeScript support
- Scales with the application

## Decision

We will use Redux Toolkit with RTK Query for state management.

## Consequences

### Positive

- Built-in best practices (immer, thunk)
- Excellent TypeScript support
- Great DevTools experience
- Time-travel debugging
- Clear separation of concerns
- Well-documented and widely adopted

### Negative

- Learning curve for new developers
- More boilerplate than simpler solutions
- Overkill for very simple state

## Alternatives Considered

### Zustand
- Pros: Simple, less boilerplate
- Cons: Limited DevTools, no time-travel

### Context API
- Pros: Built into React
- Cons: Performance issues, no DevTools

### MobX
- Pros: Less boilerplate
- Cons: Magic behavior, harder to debug

## References

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [State Management Comparison](link)
```

### ADR Index

```markdown
# Architecture Decision Records

## Active

- [ADR-001: Redux Toolkit for State Management](./adr-001-redux-toolkit.md)
- [ADR-002: Vitest for Testing](./adr-002-vitest-testing.md)
- [ADR-003: Webpack for Bundling](./adr-003-webpack-bundling.md)

## Superseded

- [ADR-004: Jest for Testing](./adr-004-jest-testing.md) → Superseded by ADR-002

## Rejected

- [ADR-005: Vite for Bundling](./adr-005-vite-bundling.md) - Not compatible with VSCode extension API
```

---

## Contributing Guidelines

### CONTRIBUTING.md

```markdown
# Contributing to DAPA VSCode Extension

Thank you for considering contributing! This document outlines the process for contributing.

## Code of Conduct

Please read and follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues. When you create a bug report, include as many details as possible:

- **Title**: Clear and descriptive
- **Description**: Detailed description of the issue
- **Steps to Reproduce**: Step-by-step instructions
- **Expected Behavior**: What should happen
- **Actual Behavior**: What actually happens
- **Screenshots**: If applicable
- **Environment**: OS, VS Code version, extension version

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, include:

- **Clear title and description**
- **Use case**: Why is this enhancement needed?
- **Proposed solution**: How should it work?
- **Alternatives considered**: Other approaches you've thought about

### Pull Requests

1. Fork the repo and create your branch from `main`
2. If you've added code that should be tested, add tests
3. Ensure the test suite passes
4. Make sure your code lints
5. Update documentation
6. Create a pull request

## Development Process

### Git Workflow

We use [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/):

- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `fix/*`: Bug fixes
- `release/*`: Release preparation

### Commit Messages

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: feat, fix, docs, style, refactor, perf, test, chore

Examples:
```
feat(editor): Add syntax highlighting for OpenAPI
fix(validation): Resolve null pointer in schema validator
docs(readme): Update installation instructions
```

### Coding Standards

- Follow the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- Use TypeScript strict mode
- Write meaningful comments
- Keep functions small and focused
- Follow SOLID principles

### Testing Standards

- Write tests for all new features
- Maintain 85%+ code coverage
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)

## Project Structure

See [Project Structure Documentation](./docs/01-project-structure.md)

## Questions?

Feel free to ask questions in:
- [Discussions](https://github.com/your-org/dapa/discussions)
- [Discord](link-to-discord)
- Email: dev@example.com

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
```

---

## Best Practices Summary

### ✅ DO

- **Review code** thoroughly
- **Provide constructive** feedback
- **Follow PR** template
- **Keep PRs small** and focused
- **Update documentation** with changes
- **Write meaningful** commit messages
- **Communicate early** about large changes
- **Be respectful** in all interactions

### ❌ DON'T

- **Don't merge** without reviews
- **Don't skip** tests
- **Don't break** the build
- **Don't push** to main directly
- **Don't ignore** review comments
- **Don't create** giant PRs
- **Don't forget** to update docs
- **Don't be** dismissive of feedback

### 🎯 Team Health Metrics

Monitor these indicators:
- PR review time (target: <24h)
- PR merge time (target: <48h)
- Test coverage (target: ≥85%)
- Build success rate (target: ≥95%)
- Documentation coverage
- Code review participation
- Bus factor (target: >2 per feature)

### 👥 Roles & Responsibilities

**Maintainers:**
- Review and merge PRs
- Triage issues
- Release management
- Architecture decisions
- Mentor contributors

**Contributors:**
- Submit quality PRs
- Respond to review feedback
- Help with issue triage
- Improve documentation
- Share knowledge

**Users:**
- Report bugs
- Suggest features
- Test pre-releases
- Share feedback
- Evangelize project

---

This governance structure ensures the DAPA project remains healthy, welcoming, and sustainable with clear processes for contribution and decision-making.

