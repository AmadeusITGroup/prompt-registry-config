# Prompt Registry Hub Configuration

This repository contains the centralized Hub configuration for the Prompt Registry extension, designed to provide Amadeus engineers with curated collections of GitHub Copilot prompt bundles tailored to their development workflows.

## Overview

A **Hub** is a centralized configuration that defines:
- **Sources**: Repositories or locations where prompt bundles are stored
- **Profiles**: Pre-configured collections of bundles organized by role or workflow
- **Metadata**: Information about the Hub, including maintainers and update history

This Hub configuration acts as a single source of truth for managing prompt bundles across your organization, ensuring consistency and ease of maintenance.

## Purpose

This repository enables:
- **Centralized Management**: Single location to maintain prompt bundle configurations
- **Role-Based Access**: Pre-configured profiles for different engineering roles (Developers, QA, MCP Developers)
- **Version Control**: Track changes to Hub configuration over time
- **Easy Distribution**: Share Hub configurations via simple URL references
- **Consistency**: Ensure all team members use approved and curated prompt bundles

## Hub Configuration Structure

The `hub-config.yml` file follows this structure:

```yaml
version: 1.0.0
metadata:
  name: My Awesome Hub
  description: A curated list of prompts
  maintainer: Your Name
  updatedAt: '2025-11-29T22:02:24.607Z'

sources:
  - id: source-identifier
    name: Source Display Name
    type: github | gitlab | http | local
    url: https://github.com/org/repo
    enabled: true
    priority: 1
    metadata:
      description: Description of the source
      homepage: https://source-homepage.com

profiles:
  - id: profile-id
    name: Profile Display Name
    description: Profile description
    bundles:
      - id: bundle-id
        version: latest | specific-version
        source: source-identifier
        required: true | false
```

### Key Concepts

#### 1. **Version**
The schema version of the Hub configuration. Use `1.0.0` for current format.

#### 2. **Metadata**
Information about the Hub:
- `name`: Display name for the Hub
- `description`: Brief description of the Hub's purpose
- `maintainer`: Person or team responsible for maintenance
- `updatedAt`: ISO timestamp of last update

#### 3. **Sources**
External repositories or locations that provide prompt bundles:
- `id`: Unique identifier for the source
- `name`: Display name shown in UI
- `type`: Source type (`github`, `local`) - in practice only `github` type is used
- `url`: Location of the source repository
- `enabled`: Whether the source is currently active
- `priority`: Order in which sources are processed (lower = higher priority)

#### 4. **Profiles**
Pre-configured collections of bundles organized by role or workflow:
- `id`: Unique identifier for the profile
- `name`: Display name for the profile
- `description`: Purpose and target audience
- `bundles`: Array of bundle references
  - `id`: Bundle identifier from the source
  - `version`: Specific version or `latest`
  - `source`: Which source provides this bundle
  - `required`: Whether the bundle is mandatory for the profile

## Using This Hub

### For End Users

1. **Install the Prompt Registry Extension** in VS Code
2. **Add This Hub**:
   - Open Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`)
   - Run: `Prompt Registry: Switch Hub`
   - Choose "Import New Hub"
   - Enter the oraganisation/repo and the branch containing the `hub-config.yml` to be used

3. **Activate Profiles**:
   - Browse available profiles in the Prompt Registry view
   - Activate profiles relevant to your role (Developer, QA, MCP Developer, etc.)
   - All bundles in the profile will be automatically installed

### For Hub Maintainers

#### Adding New Sources

Add new prompt bundle sources to make them available across all profiles:

```yaml
sources:
  - id: company-internal-prompts
    name: Company Internal Prompts
    type: github
    url: https://github.com/your-org/internal-prompts
    enabled: true
    priority: 2
    metadata:
      description: Internal company-specific prompt bundles
      homepage: https://wiki.company.com/prompts
```

#### Creating New Profiles

Define new profiles for specific roles or workflows:

```yaml
profiles:
  - id: data-engineer
    name: Data Engineer
    description: Prompts for data engineering workflows
    bundles:
      - id: sql-database-development
        version: latest
        source: awesome-copilot-official
        required: true
      - id: python-data-science
        version: latest
        source: awesome-copilot-official
        required: true
```

#### Adding Bundles to Existing Profiles

Extend existing profiles with additional bundles:

```yaml
profiles:
  - id: developer
    name: Developer
    description: Prompts for developers
    bundles:
      # ... existing bundles ...
      - id: new-bundle-id
        version: latest
        source: source-identifier
        required: false
```

#### Version Management

- Use `latest` for bundles that should always use the newest version
- Specify exact versions (e.g., `1.2.3`) for stability and reproducibility
- Update `metadata.updatedAt` whenever making changes

## Best Practices

### Maintenance

1. **Regular Updates**: Review and update the Hub configuration regularly
2. **Version Pinning**: For production profiles, consider pinning specific versions
3. **Testing**: Test changes in a development Hub before updating the main configuration
4. **Documentation**: Keep bundle descriptions clear and up-to-date
5. **Changelog**: Maintain a CHANGELOG.md to track configuration changes

### Organization

1. **Logical Grouping**: Organize profiles by role, team, or workflow
2. **Required vs Optional**: Mark critical bundles as required, others as optional
3. **Source Priority**: Use priority to control bundle resolution order
4. **Naming Conventions**: Use consistent naming for sources and profiles

### Security

1. **Access Control**: Restrict write access to Hub configuration repository
2. **Review Process**: Implement PR reviews for configuration changes
3. **Source Verification**: Ensure all sources are from trusted locations
4. **Audit Trail**: Use Git history to track who changed what and when

## Profile Examples

### Developer Profile
Comprehensive set of bundles for general software development across multiple languages and platforms.

### QA Profile
Focused on testing, automation, and security best practices for quality assurance engineers.

### MCP Developer Profile
Specialized bundles for developers working with Model Context Protocol (MCP) across various programming languages.

## Contributing

### How to Contribute

1. **Fork** this repository
2. **Create a branch** for your changes: `git checkout -b add-new-profile`
3. **Make changes** to `hub-config.yml`
4. **Test** your changes using the Prompt Registry extension
5. **Submit a Pull Request** with:
   - Clear description of changes
   - Rationale for additions/modifications
   - Testing confirmation

### Contribution Guidelines

- **Profile Additions**: Propose new profiles with clear use cases
- **Bundle Additions**: Include description and justification
- **Source Additions**: Verify source availability and authenticity
- **Version Updates**: Test compatibility before updating versions
- **Documentation**: Update this README if adding new concepts

### Review Process

All changes to `hub-config.yml` require:
1. Validation against schema
2. Testing with Prompt Registry extension
3. Approval from Hub maintainers
4. Verification of source URLs

## Troubleshooting

### Hub Not Loading
- Verify the Hub URL is accessible
- Check `hub-config.yml` syntax is valid YAML
- Ensure `version` field is present and valid

### Bundles Not Installing
- Verify source URLs are accessible
- Check bundle IDs exist in the specified source
- Ensure bundle versions are available
- Review source `enabled` status

### Profile Activation Issues
- Confirm all required bundles are available
- Check for conflicting bundle versions
- Verify source priority settings

## Support

For issues or questions:
- **Extension Issues**: Report to [Prompt Registry Extension Repository](https://github.com/AmadeusITGroup/prompt-registry)
- **Hub Configuration**: Open an issue in this repository
- **Bundle Content**: Contact the respective source maintainers

## Related Resources

- [Prompt Registry Extension](https://github.com/AmadeusITGroup/prompt-registry)
- [GitHub Awesome Copilot](https://github.com/github/awesome-copilot)
- [Hub Configuration Schema](./schemas/hub-config.schema.json)

## License

This configuration repository follows your organization's licensing policies. Individual prompt bundles are licensed according to their respective sources.

---

**Maintainer**: See `hub-config.yml` metadata for current maintainer information  
**Last Updated**: See `hub-config.yml` metadata for last update timestamp
