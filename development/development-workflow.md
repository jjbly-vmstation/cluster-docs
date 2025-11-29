# Development Workflow

Development process for VMStation.

## Git Workflow

### Branch Naming

```
feature/<description>
fix/<issue>
docs/<topic>
```

### Commit Messages

```
type: brief description

Optional longer description

References: #issue
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Tests
- `chore`: Maintenance

## Making Changes

### 1. Create Branch

```bash
git checkout -b feature/my-feature
```

### 2. Make Changes

Edit files, add tests.

### 3. Test Locally

```bash
# Syntax check
ansible-playbook --syntax-check ansible/playbooks/*.yaml

# Dry run
./deploy.sh debian --check
```

### 4. Commit

```bash
git add .
git commit -m "feat: add new feature"
```

### 5. Push

```bash
git push origin feature/my-feature
```

### 6. Create PR

Open pull request on GitHub.

## Testing Changes

### Local Testing

```bash
# Syntax validation
./tests/test-syntax.sh

# Dry run deployment
./deploy.sh debian --check
./deploy.sh monitoring --check
```

### Cluster Testing

```bash
# Deploy changes
./deploy.sh debian

# Run validation
./tests/test-complete-validation.sh
```

## Code Review

### What to Check

- Code correctness
- Documentation updates
- Test coverage
- Breaking changes

### Approval

- One approval required
- CI must pass
- Merge when ready

## Release Process

1. Update version in README
2. Update changelog
3. Tag release
4. Push tag

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

## Related Documentation

- [Contributing](contributing.md)
- [Testing Guide](testing-guide.md)
- [Coding Standards](coding-standards.md)
