# Publishing Checklist

Recommended public repository:

```text
oomus-ehealth-campaignid-docs
```

## Before Publishing

- Confirm the documentation license with OOMUS.
- Review every Markdown file for secrets, credentials and private customer data.
- Confirm all screenshots or assets are anonymized before adding them to `assets/`.
- Keep internal governance, business, legal, SLA and risk documents outside this repository.

## GitHub Setup

1. Create a new public GitHub repository named `oomus-ehealth-campaignid-docs`.
2. Copy the contents of this `public-docs/` folder into the new repository root.
3. Push the initial commit.
4. Enable GitHub Pages if a static documentation site is required.
5. Use `SUMMARY.md` as the navigation source for a future Docusaurus, Mintlify or GitBook migration.

## Suggested Initial Commit

```bash
git add .
git commit -m "Initial public documentation"
git branch -M main
git remote add origin git@github.com:<org>/oomus-ehealth-campaignid-docs.git
git push -u origin main
```

