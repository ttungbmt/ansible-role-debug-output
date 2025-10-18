# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2025-10-18

### Added
- **Facts Filtering**: New configuration variables for filtering `ansible_facts`
  - `debug_facts_exclude_keys`: List of fact keys to exclude (default: verbose/large facts)
  - `debug_facts_include_keys`: List of fact keys to explicitly include (whitelist mode)
  - Default excludes: `packages`, `services`, `devices`, `device_links`, `flags`, and common network interfaces
  - **Performance Impact**: Reduces per-host file size by up to 94% (183K → 11K)
  - **Facts Reduction**: Filters out the most impactful facts while keeping 95% of useful debugging information
  - Comprehensive documentation with size impact for each excluded key
  - Optional exclusions section for further customization

### Changed
- **Simplified Output Format**: YAML-only approach reduces complexity and maintenance burden
  - All reports are now valid YAML documents parseable by standard YAML parsers
  - File extensions now correctly match content format (`.yaml` for YAML files)
  - Cleaner codebase with removal of format-specific conditionals
  - Template filters changed from `to_json` to `to_yaml`/`to_nice_yaml` for proper YAML formatting
- **Improved YAML Formatting**: Multi-line list formatting for filter configuration
  - Filter lists (include_keys, exclude_keys, patterns) now use multi-line YAML format
  - Better readability with each list item on its own line
- **Updated Documentation**: Comprehensive updates to README.md with facts filtering examples

### Removed
- **JSON Output Support**: Removed to simplify codebase and focus on YAML-only output
  - Removed `debug_output_format` variable
  - Removed `per_host_report.json.j2` template
  - Removed all JSON-related conditional logic from tasks

### Breaking Changes
- **Output Format**: Role now only generates YAML output (no JSON support)
- **File Extensions**: Per-host reports now use `.yaml` extension instead of `.txt`
- **Variable Removal**: `debug_output_format` variable no longer exists

### Migration Guide from v1.1.0
If you were using JSON output format:
1. Update any scripts that parse JSON output to parse YAML instead
2. Update file paths from `.txt` to `.yaml` extension
3. Remove any references to `debug_output_format` variable

## [1.1.0] - 2025-10-17

### Added
- **Advanced Variable Filtering**: Include/exclude variables using explicit lists or regex patterns
  - `debug_include_keys`: Whitelist specific variables
  - `debug_exclude_keys`: Blacklist specific variables (default excludes large structural variables)
  - `debug_include_patterns`: Include variables matching regex patterns
  - `debug_exclude_patterns`: Exclude variables matching regex patterns
- **Selective Report Sections**: Control which sections are included in per-host reports
  - `debug_report_sections`: Set to `all` or list of section names
  - Available sections: `inventory_info`, `group_names`, `vars`, `facts`, `environment_task`, `environment_system`, `connection_info`, `play_info`, `ansible_config`
- **Automatic Cleanup**: Optionally clean output directory before generating new reports
  - `debug_clean_output_dir`: Boolean to enable/disable cleanup (default: `true`)
- **Valid YAML Output**: All reports are now valid YAML format, parseable by standard YAML parsers
  - Proper YAML document structure with `---` separator
  - Correct indentation and formatting
- **Correct File Extensions**: Per-host reports use `.yaml` extension instead of `.txt`
- **Performance Improvements**: Optimized variable filtering logic

### Changed
- **Default Exclude Keys**: Updated to exclude more large structural variables by default
- **File Naming**: Changed from `.txt` to `.yaml` for YAML output files
- **Documentation**: Comprehensive updates to README.md with new features and examples

## [1.0.0] - 2025-10-16

### Added
- Initial release of `debug_output` role
- Generate per-host debug reports with variables, facts, and environment information
- Generate inventory-wide reports (groups, inventory graph, inventory list)
- Support for multiple output formats (YAML and JSON)
- Automatic inventory detection
- Configurable output directory
- Optional hostvars dump for entire inventory
- Comprehensive documentation and examples

### Features
- **Per-Host Reports**: Individual YAML/JSON files for each host containing:
  - Inventory information (hostname, groups)
  - Filtered variables
  - Ansible facts
  - Environment variables (task-level and system-level)
  - Connection information
  - Play information
  - Ansible configuration
- **Inventory-Wide Reports**:
  - Groups mapping (all groups and their members)
  - Inventory graph (visual tree structure)
  - Inventory list (full inventory in YAML format)
  - Optional hostvars dump
- **Flexible Configuration**:
  - Configurable output directory
  - Optional inventory file specification
  - Control over hostvars dump generation
