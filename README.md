# Data Product Infrastructure Starter Kit

A collection of production-ready Helm charts for bootstrapping data product infrastructures on Kubernetes. This starter kit provides reusable templates and charts to quickly deploy and manage PostgreSQL databases, TLS certificates, and secrets management.

## Overview

This repository contains Helm charts designed to work together to provide a complete infrastructure stack for data products. All charts are automatically packaged and published to GitHub Container Registry on every push to the main branch.

## Available Charts

### 1. [cnpg-database](./charts/cnpg-database)

**Version:** 0.2.0 | **Type:** Application

A comprehensive Helm chart for deploying CloudNativePG PostgreSQL clusters with enterprise-grade features.

**Key Features:**
- Multiple bootstrap methods (initdb, recovery, pg_basebackup)
- Automated backups to S3, GCS, or volume snapshots
- Point-in-Time Recovery (PITR)
- High availability with streaming replication
- TLS/SSL encryption with Vault integration
- Managed database roles and custom pg_hba rules

**Use Cases:**
- Production PostgreSQL databases
- Development and staging environments
- Database cloning and testing
- Disaster recovery scenarios

**Quick Start:**
```bash
helm pull oci://ghcr.io/makassar-superapp/cnpg-database --version 0.2.0
```

[📖 Full Documentation](./charts/cnpg-database/README.md)

---

### 2. [database-tls-cert](./charts/database-tls-cert)

**Version:** 0.1.9 | **Type:** Application

Automates TLS certificate generation and management for PostgreSQL databases using cert-manager and HashiCorp Vault.

**Key Features:**
- Automated certificate lifecycle management
- Vault PKI integration
- Multiple certificate support
- Service account and RBAC automation
- Certificate concatenation controller
- Auto-reload for CloudNativePG

**Use Cases:**
- Secure database connections
- Certificate-based authentication
- Automated certificate rotation
- Multi-database certificate management

**Quick Start:**
```bash
helm pull oci://ghcr.io/makassar-superapp/database-tls-cert --version 0.1.9
```

[📖 Full Documentation](./charts/database-tls-cert/README.md)

---

### 3. [eso-secrets](./charts/eso-secrets)

**Version:** 0.1.0 | **Type:** Application

Manages Kubernetes secrets using External Secrets Operator (ESO) with HashiCorp Vault backend.

**Key Features:**
- Automated secret synchronization from Vault
- Multiple secret support
- Vault KV v2 integration
- Flexible conversion and decoding strategies
- Service account management

**Use Cases:**
- Application secrets management
- Database credentials synchronization
- API keys and tokens management
- Multi-environment secret deployment

**Quick Start:**
```bash
helm pull oci://ghcr.io/makassar-superapp/eso-secrets --version 0.1.0
```

[📖 Full Documentation](./charts/eso-secrets/README.md)

---

## Prerequisites

### Required Components

- **Kubernetes:** 1.19+ (tested on 1.24+)
- **Helm:** 3.0+ (recommended 3.15.1+)
- **HashiCorp Vault:** 1.10+ (for certificate and secrets management)

### Optional Components

Depending on which charts you use, you may need:

- **CloudNativePG Operator:** For cnpg-database chart ([Installation Guide](https://cloudnative-pg.io/documentation/current/installation_upgrade/))
- **cert-manager:** For database-tls-cert chart ([Installation Guide](https://cert-manager.io/docs/installation/))
- **External Secrets Operator:** For eso-secrets chart ([Installation Guide](https://external-secrets.io/latest/introduction/getting-started/))

## Installation

### From GitHub Container Registry (OCI)

All charts are published to GitHub Container Registry and can be pulled directly:

```bash
# Pull a specific chart
helm pull oci://ghcr.io/makassar-superapp/<chart-name> --version <version>

# Example: Pull cnpg-database chart
helm pull oci://ghcr.io/makassar-superapp/cnpg-database --version 0.2.0

# Extract and install
tar -xzf cnpg-database-0.2.0.tgz
cd cnpg-database
helm install my-database . -f values.yaml -n database-namespace --create-namespace
```

### From Git Repository

Clone the repository and install charts locally:

```bash
# Clone the repository
git clone https://github.com/makassar-superapp/dataproduct-starterkit.git
cd dataproduct-starterkit

# Install a chart
cd charts/cnpg-database
helm install my-database . -f values.yaml -n database-namespace --create-namespace
```

### As Subcharts

Use charts as dependencies in your own Helm charts:

```yaml
# Chart.yaml
apiVersion: v2
name: my-application
version: 0.1.0
dependencies:
  - name: cnpg-database
    version: 0.2.0
    repository: oci://ghcr.io/makassar-superapp
  - name: eso-secrets
    version: 0.1.0
    repository: oci://ghcr.io/makassar-superapp
```

## Common Usage Patterns

### Pattern 1: Complete Database Stack

Deploy a PostgreSQL database with TLS certificates and secrets management:

```bash
# 1. Deploy TLS certificates
helm install db-certs charts/database-tls-cert -f db-certs-values.yaml -n database

# 2. Deploy secrets
helm install db-secrets charts/eso-secrets -f db-secrets-values.yaml -n database

# 3. Deploy PostgreSQL database
helm install postgres charts/cnpg-database -f postgres-values.yaml -n database
```

### Pattern 2: Development Environment

Quick setup for development with minimal security:

```bash
# Deploy database without TLS
helm install dev-db charts/cnpg-database \
  --set instances=1 \
  --set storage.size=5Gi \
  --set backup.enabled=false \
  --set serverCerts.enabled=false \
  -n development
```

### Pattern 3: Production HA Setup

High-availability setup with backups and monitoring:

```bash
# Deploy with production values
helm install prod-db charts/cnpg-database \
  -f production-values.yaml \
  -n production
```

## CI/CD Integration

### GitHub Actions

Charts are automatically built and published using GitHub Actions:

```yaml
# .github/workflows/ci.yml
- Package all charts in charts/ directory
- Publish to ghcr.io/makassar-superapp/<chart-name>
- Upload as workflow artifacts
```

### Accessing Published Charts

Charts are available at:
- **OCI Registry:** `oci://ghcr.io/makassar-superapp/<chart-name>`
- **Workflow Artifacts:** Available for 30 days from Actions tab

## Chart Versioning

All charts follow [Semantic Versioning](https://semver.org/):

- **Major version (X.0.0):** Breaking changes
- **Minor version (0.X.0):** New features, backward compatible
- **Patch version (0.0.X):** Bug fixes, backward compatible

## Directory Structure

```
dataproduct-starterkit/
├── .github/
│   └── workflows/
│       └── ci.yml              # CI/CD pipeline
├── charts/
│   ├── cnpg-database/          # PostgreSQL database chart
│   │   ├── templates/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── README.md
│   ├── database-tls-cert/      # TLS certificate management
│   │   ├── templates/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── README.md
│   └── eso-secrets/            # Secrets management
│       ├── templates/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── README.md
├── helm-image/
│   └── Dockerfile              # Custom Helm image for CI/CD
├── LICENSE
└── README.md                   # This file
```

## Adding New Charts

To add a new chart to this repository:

1. **Create Chart Directory**
   ```bash
   mkdir -p charts/my-new-chart
   cd charts/my-new-chart
   helm create .
   ```

2. **Update Chart.yaml**
   ```yaml
   apiVersion: v2
   name: my-new-chart
   description: Description of your chart
   type: application
   version: 0.1.0
   appVersion: "1.0.0"
   ```

3. **Create README.md**
   - Follow the structure of existing chart READMEs
   - Include installation instructions
   - Document all configuration parameters
   - Provide usage examples

4. **Test Locally**
   ```bash
   helm lint .
   helm template test . -f values.yaml
   helm install test . --dry-run
   ```

5. **Commit and Push**
   ```bash
   git add charts/my-new-chart
   git commit -m "Add my-new-chart"
   git push origin main
   ```

The CI/CD pipeline will automatically package and publish your chart.

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Fork the repository**
2. **Create a feature branch:** `git checkout -b feature/my-new-feature`
3. **Make your changes** and test thoroughly
4. **Update documentation** as needed
5. **Commit your changes:** `git commit -m 'Add some feature'`
6. **Push to the branch:** `git push origin feature/my-new-feature`
7. **Open a Pull Request**

### Chart Development Guidelines

- Follow [Helm best practices](https://helm.sh/docs/chart_best_practices/)
- Use semantic versioning for chart versions
- Document all values in values.yaml with comments
- Provide comprehensive README with examples
- Include CHANGELOG.md for version history
- Test charts in multiple environments

## Support

For issues, questions, or contributions:

- **GitHub Issues:** [https://github.com/makassar-superapp/dataproduct-starterkit/issues](https://github.com/makassar-superapp/dataproduct-starterkit/issues)
- **Discussions:** Use GitHub Discussions for questions and ideas

## License

This project is licensed under the terms specified in the [LICENSE](./LICENSE) file.

## Acknowledgments

- [CloudNativePG](https://cloudnative-pg.io/) - PostgreSQL operator for Kubernetes
- [cert-manager](https://cert-manager.io/) - Certificate management for Kubernetes
- [External Secrets Operator](https://external-secrets.io/) - Secrets management
- [HashiCorp Vault](https://www.vaultproject.io/) - Secrets and encryption management

## Roadmap

Future charts planned for this repository:

- [ ] **monitoring-stack**: Prometheus, Grafana, and alerting
- [ ] **backup-manager**: Centralized backup management
- [ ] **data-pipeline**: Apache Airflow or similar
- [ ] **object-storage**: MinIO or S3-compatible storage
- [ ] **message-queue**: RabbitMQ or Kafka
- [ ] **api-gateway**: Kong or similar

---

**Maintained by:** Makassar Super App Team  
**Last Updated:** October 2024