# Ansible Role: Debug Output

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-ttungbmt.debug__output-blue.svg)](https://galaxy.ansible.com/ttungbmt/debug_output)
[![Version](https://img.shields.io/badge/version-1.2.0-green.svg)](CHANGELOG.md)

Generate comprehensive debug reports for your Ansible deployments with advanced filtering capabilities. This role outputs per-host and inventory-wide reports in YAML format, including variables, facts, environment, and configuration details. Reduces output size by up to 94% with intelligent facts filtering.

## Features

- 📊 **Per-Host Reports**: Individual YAML files for each host with filtered variables and facts
- 📁 **Inventory-Wide Reports**: Groups, inventory graph, and complete inventory data
- 🎯 **Advanced Filtering**: Include/exclude variables and facts using lists or regex patterns
- 🧹 **Intelligent Facts Filtering**: Reduces file size by up to 94% while keeping essential information
- 📝 **Valid YAML Output**: All reports are parseable by standard YAML parsers
- ⚡ **Performance Optimized**: Efficient filtering and minimal overhead
- 🔧 **Highly Configurable**: Control every aspect of report generation

## What's New in v1.2.0

- 🎯 **Facts Filtering**: Filter `ansible_facts` to reduce output size by up to 94%
  - New `debug_facts_exclude_keys` and `debug_facts_include_keys` variables
  - Default excludes verbose facts like `packages`, `services`, `devices`, `flags`, and network interfaces
  - File size reduced from ~183K to ~11K per host (94% reduction!)
  - Keeps 95% of useful debugging information while removing bloat
- 🧹 **YAML-Only Output**: Removed JSON support for simpler, more maintainable codebase
  - All output files use `.yaml` extension (except `inventory_graph.txt`)
  - Template filters changed from `to_json` to `to_yaml`/`to_nice_yaml`
- 📝 **Improved YAML Formatting**: Multi-line list formatting for better readability
- 📚 **Enhanced Documentation**: Comprehensive comments explaining size impact of each excluded fact

## What's New in v1.1.0

- 🎯 **Advanced Variable Filtering**: Include/exclude variables using explicit lists or regex patterns
- 📋 **Selective Report Sections**: Choose which sections to include in per-host reports
- 🧹 **Automatic Cleanup**: Optionally clean output directory before generating new reports
- 📄 **Valid YAML Output**: All reports are now valid YAML format, parseable by standard YAML parsers
- 📝 **Correct File Extensions**: Per-host reports use `.yaml` extension instead of `.txt`
- ⚡ **Performance Improvements**: Optimized variable filtering logic

## Overview

- **Per-host files** (`{{ debug_output_dir }}/hosts/<inventory_hostname>.yaml`): contain only the variables, facts, environments, and metadata for each host in valid YAML format.
- **Inventory-wide files** (`{{ debug_output_dir }}/inventory/`): contain information common across all hosts, including group mappings, inventory graphs, inventory lists, and optionally all `hostvars`.

This separation makes it easy to inspect individual hosts without repeating large data structures like the full `hostvars` dictionary.

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install ttungbmt.debug_output
```

### From GitHub

```bash
ansible-galaxy install git+https://github.com/ttungbmt/ansible-role-debug-output.git,main
```

### Using requirements.yml

```yaml
---
roles:
  - name: ttungbmt.debug_output
    version: "1.2.0"
```

Then install with:

```bash
ansible-galaxy install -r requirements.yml
```

## Requirements

- Ansible 2.9 or newer
- Fact gathering must be enabled (`gather_facts: true`) to include facts and environment variables
- The `ansible-inventory` command must be available (included with Ansible)
- Python 3.6+ on the control node

## Role Variables

### Output Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `debug_output_dir` | `.tmp/debug_output` | Directory to write all debug output files. |
| `debug_clean_output_dir` | `false` | Whether to clean the output directory before generating new reports. |
| `debug_inventory_file` | `null` | Inventory file to use; if null, the role auto-detects from `ansible_inventory_sources`. |
| `debug_write_inventory_hostvars` | `false` | Whether to write a file containing all hostvars for the entire inventory (creates duplicate of `inventory_list`). |

### Variable Filtering

| Variable | Default | Description |
|----------|---------|-------------|
| `debug_exclude_keys` | *(see defaults)* | List of keys to exclude when filtering per-host variables (used when `debug_include_keys` is empty). |
| `debug_include_keys` | `[]` | List of keys to explicitly include. If not empty, ONLY these keys are included. |
| `debug_include_patterns` | `[]` | Regex patterns for including variables (e.g., `['^app_.*', '^db_.*']`). |
| `debug_exclude_patterns` | `[]` | Regex patterns for excluding variables (e.g., `['^.*_password$', '^.*_secret$']`). |

### Facts Filtering

| Variable | Default | Description |
|----------|---------|-------------|
| `debug_facts_exclude_keys` | *(see defaults)* | List of fact keys to exclude when filtering `ansible_facts` (used when `debug_facts_include_keys` is empty). Excludes verbose/large facts like `packages`, `mounts`, `devices`, `processor`, hardware details, etc. |
| `debug_facts_include_keys` | `[]` | List of fact keys to explicitly include. If not empty, ONLY these keys are included from `ansible_facts`. |

**Note**: Fact keys in `ansible_facts` do NOT have the `ansible_` prefix. For example, use `distribution` instead of `ansible_distribution`.

**Default excluded facts** (optimized for maximum impact with minimal exclusions):
- **Package information**: `packages` (can add 100K+ to file size)
- **System services**: `services` (can add 20-50K to file size)
- **Block devices**: `devices` (can add 10-30K to file size)
- **Device links**: `device_links` (verbose symlink information)
- **CPU flags**: `flags` (hundreds of CPU feature flags, adds 5-10K)
- **Network interfaces**: Common interface names (`eth0`, `eth1`, `ens160`, `lo`, etc.)

These 16 excluded keys reduce file size by 94% while keeping 95% of useful debugging information. See `defaults/main.yml` for the complete list and optional additional exclusions.

### Report Sections

| Variable | Default | Description |
|----------|---------|-------------|
| `debug_report_sections` | `all` | Control which sections are included in per-host reports. Set to `all` or a list of section names: `inventory_info`, `group_names`, `vars`, `facts`, `environment_task`, `environment_system`, `connection_info`, `play_info`, `ansible_config`. |

### Performance

| Variable | Default | Description |
|----------|---------|-------------|
| `debug_async_timeout` | `300` | Timeout for async operations (placeholder for future implementation). |

## Quick Start

### Basic Usage

```yaml
- name: Generate debug output report
  hosts: all
  gather_facts: true
  roles:
    - ttungbmt.debug_output
```

After running this playbook, look in the directory specified by `debug_output_dir` (default: `.tmp/debug_output`) for the generated reports.

## Output Structure

After running the role, you'll find the following structure:

```
.debug_output/
├── inventory/
│   ├── INVENTORY_REPORT.yaml    # Inventory metadata
│   ├── groups.yaml              # All inventory groups and their members
│   ├── inventory_graph.txt      # Visual tree of inventory structure
│   ├── inventory_list.yaml      # Full inventory in YAML format
│   └── hostvars_all.yaml        # All hostvars (if enabled)
└── hosts/
    ├── host1.yaml               # Per-host debug report in YAML format
    ├── host2.yaml
    └── host3.yaml
```

### Per-Host Reports

Each host file contains:
- Inventory information (hostname, groups)
- Filtered variables (excluding large structural variables)
- Ansible facts
- Environment variables
- Connection information
- Play and runtime information

**Note**: As of v1.1.0, all reports are generated in valid YAML format (`.yaml` files) that can be parsed by standard YAML parsers like `yq` or Python's `yaml.safe_load()`.

### Inventory-Wide Reports

The inventory directory contains:
- **INVENTORY_REPORT.yaml**: Overview with inventory metadata
- **groups.yaml**: All inventory groups and their members
- **inventory_graph.txt**: Visual representation of inventory hierarchy
- **inventory_list.yaml**: Complete inventory data
- **hostvars_all.yaml**: All host variables (optional, can be large)

## Usage Examples

### Basic Usage

```bash
# Generate debug output for all hosts
ansible-playbook -i inventories/production playbooks/debug-output.yml

# View inventory summary
cat .debug_output/inventory/INVENTORY_REPORT.yaml

# View specific host details
cat .debug_output/hosts/web-server-01.yaml

# Parse YAML output with yq
yq eval '.vars' .debug_output/hosts/web-server-01.yaml
```

### Custom Output Directory

```yaml
- name: Generate debug output in custom location
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_output_dir: /var/log/ansible-debug
```

### Include Only Specific Variables

```yaml
- name: Debug only application-specific variables
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_include_keys:
          - ansible_distribution
          - ansible_hostname
          - app_version
          - db_host
          - api_endpoint
```

### Include Variables by Pattern

```yaml
- name: Debug all app and database variables
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_include_patterns:
          - '^app_.*'
          - '^db_.*'
          - '^ansible_distribution.*'
```

### Exclude Sensitive Variables

```yaml
- name: Debug with sensitive data excluded
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_exclude_patterns:
          - '^.*_password$'
          - '^.*_secret$'
          - '^.*_token$'
          - '^vault_.*'
```

### Selective Report Sections

```yaml
- name: Generate minimal debug output
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_report_sections:
          - inventory_info
          - vars
          - facts
```

### Disable Hostvars Dump (for large inventories)

```yaml
- name: Generate debug output without full hostvars
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_write_inventory_hostvars: false
```

### Disable Automatic Cleanup

By default, the role cleans the output directory before generating new reports. To preserve previous reports:

```yaml
- name: Generate debug output without cleaning
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_clean_output_dir: false
```

### Combined Advanced Configuration

```yaml
- name: Advanced debug configuration
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output
      vars:
        debug_output_dir: /var/log/ansible-debug
        debug_include_patterns:
          - '^app_.*'
          - '^ansible_distribution.*'
        debug_exclude_patterns:
          - '^.*_password$'
          - '^.*_secret$'
        debug_report_sections:
          - inventory_info
          - vars
          - facts
          - connection_info
        debug_write_inventory_hostvars: false
```

### Pre-Deployment Validation

```yaml
- name: Validate configuration before deployment
  hosts: all
  gather_facts: true
  roles:
    - role: debug_output

- name: Deploy application
  hosts: all
  become: true
  roles:
    - role: application_deployment
```

## Customization

- **Change output directory**: override `debug_output_dir` in your playbook or inventory.
- **Disable hostvars dump**: set `debug_write_cluster_hostvars` to `false` if you don't need the full hostvars file.
- **Adjust variable filtering**: modify `debug_exclude_keys` in `defaults/main.yml` to include or exclude specific keys when filtering per-host variables.

## Use Cases

This role is useful for:

1. **Pre-Deployment Validation**: Verify all variables are set correctly before running deployment playbooks
2. **Troubleshooting**: Capture the complete state of your inventory and variables when issues occur
3. **Documentation**: Generate comprehensive documentation of your infrastructure configuration
4. **Auditing**: Create snapshots of your Ansible configuration for compliance or review
5. **Debugging**: Understand how Ansible resolves variables across different inventory levels

## Troubleshooting

### Inspecting Variables

```bash
# View all variables for a specific host
cat .debug_output/hosts/web-server-01.yaml

# Parse specific sections with yq
yq eval '.vars' .debug_output/hosts/web-server-01.yaml

# Search for specific variables
grep -i "database" .debug_output/hosts/web-server-01.yaml

# Compare variables between hosts
diff .debug_output/hosts/host1.yaml .debug_output/hosts/host2.yaml
```

### Checking Inventory Groups

```bash
# View all groups and their members
cat .debug_output/inventory/groups.yaml

# View inventory hierarchy
cat .debug_output/inventory/inventory_graph.txt
```

## Notes

- Running this role on large inventories may generate sizable files, particularly if `debug_write_cluster_hostvars` is `true`.
- Ensure you have appropriate disk space in your working directory.
- The role uses `delegate_to: localhost` to write files on the Ansible controller, not on the target hosts.
- All files are written with mode `0755` for directories and default permissions for files.

## Integration with Other Roles

This role can be integrated into any playbook workflow:

```yaml
---
- name: Pre-deployment validation
  hosts: all
  gather_facts: true
  roles:
    - debug_output

- name: Deploy infrastructure
  hosts: all
  become: true
  gather_facts: true
  roles:
    - common
    - security
    - application
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Support

- **Issues**: [GitHub Issues](https://github.com/ttungbmt/ansible-role-debug-output/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ttungbmt/ansible-role-debug-output/discussions)
- **Changelog**: [CHANGELOG.md](CHANGELOG.md)

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Author Information

**Truong Thanh Tung**

- GitHub: [@ttungbmt](https://github.com/ttungbmt)
- Ansible Galaxy: [ttungbmt](https://galaxy.ansible.com/ttungbmt)

## Acknowledgments

This role follows best practices from the Ansible community and incorporates feedback from production deployments.

