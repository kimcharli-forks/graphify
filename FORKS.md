# Fork Information

This fork of `graphify` includes several security hardening improvements identified during a security audit on April 23, 2026.

## Security Audit Findings and Fixes

### 1. Stored XSS in Interactive HTML Visualization
- **Vulnerability**: Node IDs were injected directly into `onclick` attributes in the generated `graph.html` file. A malicious repository or document containing node IDs like `");alert(1)//` could execute arbitrary JavaScript in the user's browser.
- **Fix**: Added an `esc()` helper to the JavaScript template in `graphify/export.py` to properly escape node IDs and labels before injecting them into the DOM or attributes.

### 2. YAML Injection in Obsidian Export
- **Vulnerability**: The Obsidian vault export used double quotes to wrap metadata strings (like `source_file`) but did not escape internal double quotes. This allowed breaking out of the YAML field and injecting arbitrary YAML keys/values.
- **Fix**: Implemented a string escaping helper in `to_obsidian` (`graphify/export.py`) that escapes backslashes and double quotes.

### 3. Path Traversal in MCP Server and Path Queries
- **Vulnerability**: `validate_graph_path` had a logic flaw where it tried to "guess" the base directory from the input path, which could be bypassed. Additionally, the MCP server (`graphify/serve.py`) did not utilize this validation, relying only on a `Path.resolve()` check which is insufficient to prevent access to files outside the intended output directory.
- **Fix**: 
    - Tightened `validate_graph_path` in `graphify/security.py` to strictly enforce `graphify-out/` relative to CWD as the only allowed base.
    - Integrated `validate_graph_path` into the graph loading logic in `graphify/serve.py`.

### 4. SSRF and Cloud Metadata Protection
- **Improvement**: Enhanced `validate_url` in `graphify/security.py` to:
    - Explicitly block common cloud metadata hostnames (e.g., `metadata.google.internal`).
    - Improve private IP detection by resolving hostnames before checking `is_private`.
- **Note**: Acknowledged the remaining DNS rebinding risk in documentation, which is a limitation of using the standard `urllib` library without custom socket handling.

### 5. Command Injection in `clone` Command
- **Vulnerability**: The `graphify clone` command passed the `--branch` argument and repository identifiers directly to `git` and used them in filesystem paths without validation.
- **Fix**: Added strict regex validation for `owner`, `repo`, and `branch` names in `graphify/__main__.py`. Specifically prevented branch names from starting with `-` to avoid flag injection.

## Upstream
This fork tracks `safishamsi/graphify`.
