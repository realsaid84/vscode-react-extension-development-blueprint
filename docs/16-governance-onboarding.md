
# 16. Governance & Onboarding

### 13.1 Pull Request Requirements

- ✅ All tests pass
- ✅ Code coverage meets threshold (85%+)
- ✅ ESLint and Prettier checks pass
- ✅ SonarQube quality gate passes
- ✅ At least 2 approving reviews
- ✅ Conventional commit message format
- ✅ Updated documentation if needed

### 13.2 Developer Environment Setup

```bash
# Clone repository
git clone https://github.com/your-org/dapa.git
cd dapa

# Install dependencies
npm install

# Setup pre-commit hooks
npm run prepare

# Run type checking
npm run type-check

# Run tests
npm test

# Run linting
npm run lint

# Start development server
npm run dev
```

### 13.3 Documentation Standards

- README.md in each package with usage examples
- JSDoc comments for public APIs
- Storybook for component documentation
- ADRs (Architecture Decision Records) for major decisions

---