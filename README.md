# OOMUS eHealth CampaignID Documentation — v4.2

Public documentation for OOMUS eHealth CampaignID, a GovTech enterprise SaaS platform for generating, distributing and verifying secure digital health cards for national public health campaigns.

This repository is intended for partners, frontend integrators, technical teams and pilot clients. It covers the full v4.2 platform: quota-based billing, real-time operations dashboard, campaign simulation engine, DHIS2 integration, Google Wallet, WhatsApp/SMS distribution and offline verification.

## Start Here

- [Product Requirements](./getting-started/product-requirements.md)
- [Technical Architecture](./architecture/technical-architecture.md)
- [API Reference](./reference/api-reference.md)
- [Billing & Quota Plans](./reference/billing-quota-plans.md)
- [Simulation Wizard Guide](./guides/simulation-wizard.md)
- [Generation Workflow](./guides/generation-workflow.md)
- [Deployment Runbook](./deployment/deployment-runbook.md)

## Documentation Map

| Section | Description |
| --- | --- |
| [Getting Started](./getting-started/) | Product scope and platform overview |
| [Guides](./guides/) | Campaign simulation, generation and integration guides |
| [Integrations](./integrations/) | DHIS2, Google Wallet, WhatsApp and SMS |
| [Reference](./reference/) | API reference, billing plans and database schema |
| [Architecture](./architecture/) | Technical architecture |
| [Security](./security/) | Cryptography, offline verification, data protection and responsible AI |
| [Deployment](./deployment/) | Deployment and operations runbooks |

## Public Scope

Included:

- Product and technical overview.
- Public API and integration documentation.
- Security model summaries.
- Deployment and monitoring guidance.
- Testing strategy.

Excluded:

- Business plan and internal financial documents.
- Legal contracts, full internal compliance files and SLA terms.
- Internal security policy details and risk register.
- Secrets, credentials, environment files and production exports.
- Client data, real DHIS2 exports and personally identifiable data.

## Repository Name

Recommended repository name:

```text
oomus-ehealth-campaignid-docs
```

## Publication

This documentation is written in Markdown and can be published directly on GitHub. It is also ready for a later migration to GitHub Pages, Docusaurus or Mintlify.

Before publishing, follow the [Publishing Checklist](./PUBLISHING.md). Contributions and security reporting are covered by [CONTRIBUTING.md](./CONTRIBUTING.md), [SECURITY.md](./SECURITY.md) and [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).
