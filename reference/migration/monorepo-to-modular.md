# Monorepo to Modular

Migrating from monorepo to modular repository structure.

## Overview

The VMStation project is transitioning from a single monorepo to a modular structure:

| Repository | Purpose |
|------------|---------|
| vmstation | Main code repository |
| cluster-docs | Documentation |

## Documentation Migration

### Source Repository

```
vmstation/
├── README.md
├── QUICK_START.md
├── TODO.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── TROUBLESHOOTING.md
│   ├── USAGE.md
│   └── ...
├── AI_AGENT_IMPLEMENTATION_REPORT.md
├── DEPLOYMENT_*.md
└── ...
```

### Target Repository

```
cluster-docs/
├── README.md
├── getting-started/
│   ├── README.md
│   ├── quick-start.md
│   ├── prerequisites.md
│   └── installation.md
├── architecture/
├── deployment/
├── operations/
├── troubleshooting/
├── components/
├── reference/
├── development/
├── roadmap/
└── migration/
```

## Migration Process

### 1. Create Target Structure

```bash
mkdir -p getting-started architecture deployment operations \
  troubleshooting components reference development roadmap migration
```

### 2. Migrate Content

Content was reorganized:
- Consolidated scattered docs
- Improved organization
- Added navigation
- Updated cross-references

### 3. Update Links

Update references from:
```markdown
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
```

To:
```markdown
[Architecture](architecture/README.md)
```

### 4. Verify Links

Check all links work:

```bash
# Find broken links (manual review)
grep -r '\[.*\](.*\.md)' --include='*.md' | grep -v http
```

## Content Mapping

| Source | Target |
|--------|--------|
| README.md | README.md |
| QUICK_START.md | getting-started/quick-start.md |
| TODO.md | roadmap/todo.md |
| docs/ARCHITECTURE.md | architecture/overview.md |
| docs/TROUBLESHOOTING.md | troubleshooting/README.md |
| docs/USAGE.md | getting-started/installation.md |
| AI_AGENT_IMPLEMENTATION_REPORT.md | development/ai-agent-implementation.md |
| DEPLOYMENT_*.md | deployment/deployment-reports/ |
| DNS_SYSLOG_KERBEROS_STATUS.md | components/infrastructure/ |
| GRAFANA_*.md | troubleshooting/grafana-fixes.md |
| KUBESPRAY_*.md | deployment/kubespray-deployment.md |

## Improvements Made

### Organization

- Logical hierarchy by topic
- Consistent structure
- Clear navigation

### Navigation

- Table of contents
- Cross-references
- Related links

### Formatting

- Consistent markdown style
- Code examples
- Tables for reference

### Content

- Updated procedures
- Added context
- Improved clarity

## Using Both Repositories

### Main Code

```bash
cd vmstation
./deploy.sh <command>
```

### Documentation

```bash
cd cluster-docs
# View documentation
```

Or via GitHub: https://github.com/jjbly-vmstation/cluster-docs

## Future Improvements

- [ ] Automated link checking
- [ ] Documentation site (MkDocs/Docusaurus)
- [ ] Version tagging
- [ ] Search functionality

## Related Documentation

- [Migration Guide](migration-guide.md)
- [Getting Started](../getting-started/README.md)
