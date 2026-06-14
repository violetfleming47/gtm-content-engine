# Contributing to GTM Content Engine

Thank you for considering contributing to GTM Content Engine. This document explains how to get involved.

## How to Contribute

### Reporting Issues

- Use [GitHub Issues](https://github.com/violetfleming47/gtm-content-engine/issues) to report bugs or suggest features
- Check existing issues before creating a new one
- Include reproduction steps for bugs
- For feature requests, explain the use case and which agents/frameworks are affected

### Pull Requests

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Make your changes
4. Ensure no sensitive data is introduced (check for company names, credentials, or API keys)
5. Commit with a clear message
6. Push to your fork and open a Pull Request

### What We Accept

- **New agent skills** -- additional agents that extend the pipeline
- **Framework improvements** -- better ICP models, scoring algorithms, content strategies
- **Integration connectors** -- new CRM, analytics, or publishing integrations
- **Documentation** -- setup guides, tutorials, examples for specific industries
- **Bug fixes** -- issues with the SQL schema or agent logic
- **Templates** -- new starter templates for different company types

### Guidelines

- Keep agent skills focused on a single responsibility
- Follow existing naming conventions (`gtm-*` prefix for skills)
- Use `[YOUR ...]` syntax for company-specific values
- Write clear agent instructions -- another person should understand the agent's purpose
- Test SQL changes against `architecture/schema.md` before submitting

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold this code.

## Questions?

Open a [Discussion](https://github.com/violetfleming47/gtm-content-engine/discussions) for questions about architecture, customisation, or how to build new agents.
