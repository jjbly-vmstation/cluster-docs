# Contributing

Guidelines for contributing to VMStation.

## Getting Started

1. Fork the repository
2. Clone your fork
3. Create a feature branch
4. Make your changes
5. Test your changes
6. Submit a pull request

## Development Environment

### Prerequisites

- Ansible 2.9+
- Python 3.8+
- kubectl
- Access to cluster (optional)

### Local Setup

```bash
git clone https://github.com/<your-fork>/vmstation.git
cd vmstation
```

## Making Changes

### Code Changes

1. Follow existing code style
2. Add comments where needed
3. Test changes locally
4. Run linting

### Documentation Changes

1. Follow markdown conventions
2. Update table of contents
3. Check for broken links
4. Verify examples work

## Testing

### Syntax Check

```bash
# Ansible playbooks
ansible-playbook --syntax-check ansible/playbooks/*.yaml

# Shell scripts
shellcheck scripts/*.sh
```

### Dry Run

```bash
./deploy.sh debian --check
```

### Validation

```bash
./tests/test-complete-validation.sh
```

## Pull Request Process

1. Update documentation
2. Add tests if applicable
3. Ensure CI passes
4. Request review

## Code of Conduct

- Be respectful
- Focus on constructive feedback
- Help others learn

## Questions?

Open an issue for discussion.
