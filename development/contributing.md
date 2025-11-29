# Contributing

Guidelines for contributing to VMStation.

## Getting Started

1. Fork the repository
2. Clone your fork
3. Create a feature branch
4. Make changes
5. Test thoroughly
6. Submit pull request

## Development Setup

### Prerequisites

- Ansible 2.9+
- Python 3.8+
- kubectl
- SSH access to test nodes

### Clone Repository

```bash
git clone https://github.com/YOUR_USERNAME/VMStation.git
cd VMStation
```

### Create Branch

```bash
git checkout -b feature/my-feature
```

## Making Changes

### Code Style

- Follow existing patterns
- Use descriptive names
- Add comments for complex logic
- Keep functions focused

### Ansible

- Use YAML format
- Idempotent tasks
- Descriptive task names
- Check mode support

### Shell Scripts

- Use bash
- Include error handling
- Add help messages
- Document options

## Testing

### Run Validation

```bash
# Check syntax
ansible-playbook playbook.yml --syntax-check

# Dry run
./deploy.sh <command> --check

# Full validation
./tests/test-complete-validation.sh
```

### Test Checklist

- [ ] Syntax validation passes
- [ ] Dry-run succeeds
- [ ] Fresh deployment works
- [ ] Idempotent (run twice)
- [ ] Documentation updated

## Pull Requests

### PR Guidelines

1. Clear title describing change
2. Description of what and why
3. Reference any issues
4. Include testing evidence
5. Update documentation

### Commit Messages

Format:
```
type: short description

Longer description if needed.

Fixes #123
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `refactor`: Code refactoring
- `test`: Test changes
- `chore`: Maintenance

### Example

```
feat: add Kubespray deployment support

Add scripts and roles for deploying Kubernetes using Kubespray
as an alternative to kubeadm.

- Add run-kubespray.sh wrapper script
- Add preflight-rhel10 role
- Update documentation

Closes #42
```

## Code Review

### Review Checklist

- [ ] Code follows style guidelines
- [ ] Logic is correct
- [ ] Tests pass
- [ ] Documentation updated
- [ ] No security issues
- [ ] Backward compatible

## Documentation

### Update Docs

When making changes:
- Update relevant docs
- Add examples if needed
- Update README if applicable

### Docs Location

- `docs/` - Detailed documentation
- `README.md` - Project overview
- Inline comments for complex code

## Questions?

1. Check documentation
2. Search existing issues
3. Open new issue for discussion

## Related Documentation

- [Development Workflow](development-workflow.md)
- [Testing Guide](testing-guide.md)
- [Coding Standards](coding-standards.md)
