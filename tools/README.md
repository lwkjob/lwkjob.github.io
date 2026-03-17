# Development Tool Uninstaller

A comprehensive tool to detect and clean up development tools, their binaries, configurations, and environment variables on macOS and Linux systems.

## Why This Tool?

While there are existing uninstall tools like:
- **uninstall-cli**: Focuses on macOS applications and their associated files
- **devbin**: Manages CLI tools from package managers (Homebrew, npm, Cargo, pip)

This tool combines both approaches and adds specific functionality for:
- Development tools installed outside standard package managers
- Configuration files in multiple locations
- Environment variables and shell modifications
- Comprehensive analysis before cleanup

## Features

### Detection Capabilities
- **Binary Detection**: Scans common binary locations (`/usr/local/bin`, `/opt/homebrew/bin`, `~/.local/bin`, etc.)
- **Configuration Detection**: Finds config files in `~/.config`, `~/.*`, `~/Library/Application Support`, etc.
- **Environment Variable Analysis**: Shows relevant environment variables
- **Shell Profile Analysis**: Checks `.bashrc`, `.zshrc`, `.profile` for tool-related modifications
- **Package Manager Integration**: Detects tools installed via Homebrew, npm, pip, Cargo

### Cleanup Options
- **Safe Removal**: Interactive confirmation before deleting files
- **Dry Run Mode**: See what would be removed without actually deleting
- **Selective Cleanup**: Choose specific components to remove
- **Complete Removal**: Remove all detected components for a tool

### Supported Tools
The tool includes built-in detection patterns for common development tools:
- AI/ML tools (Claude, OpenAI, Anthropic)
- Package managers (npm, pip, cargo, gem)
- Development environments (node, python, rust, go)
- Container tools (docker, kubectl, podman)
- Infrastructure tools (terraform, ansible, pulumi)

## Installation

```bash
# Download the script
curl -O https://raw.githubusercontent.com/yourusername/dev-uninstaller/main/dev-uninstaller-enhanced.sh

# Make it executable
chmod +x dev-uninstaller-enhanced.sh

# Optional: Move to a directory in your PATH
sudo mv dev-uninstaller-enhanced.sh /usr/local/bin/dev-uninstall
```

## Usage

### Basic Usage
```bash
# Analyze a specific tool
./dev-uninstaller-enhanced.sh --analyze claude

# Cleanup a specific tool (with confirmation)
./dev-uninstaller-enhanced.sh --cleanup claude

# Dry run (see what would be removed)
./dev-uninstaller-enhanced.sh --cleanup --dry-run claude

# List all detected development tools
./dev-uninstaller-enhanced.sh --list-all
```

### Interactive Mode
```bash
# Start interactive mode
./dev-uninstaller-enhanced.sh --interactive
```

## Safety Features

- **Interactive Confirmation**: Every file/directory removal requires confirmation
- **Backup Option**: Creates timestamped backups before removal (optional)
- **Dry Run Mode**: Preview changes without making them
- **Logging**: Detailed logs of all actions performed
- **Recovery Script**: Generates a recovery script to restore removed files

## Comparison with Existing Tools

| Feature | uninstall-cli | devbin | Our Tool |
|---------|---------------|--------|----------|
| macOS App Cleanup | ✅ | ❌ | ✅ |
| CLI Tool Management | ❌ | ✅ | ✅ |
| Config File Detection | Limited | Limited | ✅✅✅ |
| Environment Analysis | ❌ | ❌ | ✅ |
| Shell Profile Analysis | ❌ | ❌ | ✅ |
| Cross-Platform | macOS only | macOS focused | ✅ macOS & Linux |
| Dry Run Mode | ❌ | ❌ | ✅ |
| Recovery Script | ❌ | ❌ | ✅ |

## Contributing

To add support for new tools:
1. Edit `dev-tools-config.json` to add detection patterns
2. Add tool-specific cleanup logic in the script
3. Test with the `--analyze` flag first

## License

MIT License