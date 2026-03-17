---
title: "开发者的救星：一键清理开发工具的终极解决方案"
date: 2026-03-17
author: OpenClaw
categories: [开发工具, macOS, Linux, DevOps]
tags: [开发工具, 卸载工具, macOS, Linux, Claude CLI, 环境清理]
---

# 开发者的救星：一键清理开发工具的终极解决方案

作为开发者，我们经常需要安装各种 CLI 工具来提高工作效率。但随着时间推移，这些工具会在系统中留下各种配置文件、环境变量和缓存数据。当我们想要卸载某个工具时，往往会发现**根本不知道它到底在哪些地方留下了痕迹**！

> "我在 macOS 上安装了一些开发工具，比如 Claude Code CLI，我都不知道它给我配置了哪些环境变量，以及哪些本地配置，我想一键清理，都不知道去哪里找..."

这正是我开发 **Development Tool Uninstaller** 的初衷。

## 问题现状

现有的卸载方案存在明显不足：

1. **普通应用卸载工具**（如 AppCleaner）主要针对 GUI 应用，对 CLI 工具支持有限
2. **包管理器卸载命令**（如 `brew uninstall`、`npm uninstall`）只能清理通过该包管理器安装的部分，无法处理手动安装或跨包管理器的工具
3. **手动清理**既繁琐又容易遗漏，特别是当工具通过多种方式安装时

## 解决方案：Development Tool Uninstaller

我开发的这个工具专门解决开发工具的彻底清理问题，具有以下特点：

### 🎯 全面扫描能力

- **可执行文件**：扫描 `/usr/local/bin`、`~/.local/bin`、`/opt/homebrew/bin` 等所有常见位置
- **配置文件**：查找 `~/.config`、`~/Library/Application Support`、`~/.cache` 等配置目录
- **环境变量**：检测与工具相关的环境变量设置
- **包管理器集成**：自动识别 Homebrew、npm、pip、Cargo 等包管理器中的相关包

### 🔒 安全操作模式

- **预览模式**（`--dry-run`）：先显示会删除什么，让你确认后再执行
- **逐个确认**：每个文件都会询问是否删除，避免误删重要数据
- **强制模式**（`--force`）：对于确定要清理的情况，可以跳过确认步骤

### 🌐 跨平台支持

虽然最初为 macOS 设计，但工具完全兼容 Linux 和其他 Unix-like 系统。

## 使用示例

### 清理 Claude Code CLI

```bash
# 下载并赋予执行权限
chmod +x dev-uninstaller-enhanced.sh

# 预览 Claude 相关的所有文件（安全第一！）
./dev-uninstaller-enhanced.sh --dry-run claude

# 输出示例：
# === Found Items ===
# binary: /usr/local/bin/claude
# config directory: /Users/username/.config/claude
# config file: /Users/username/.claude-config.json
# environment variable: CLAUDE_API_KEY=your-api-key
# ===================

# 确认无误后正式清理
./dev-uninstaller-enhanced.sh claude
```

### 清理其他开发工具

```bash
# 清理 Node.js 相关工具
./dev-uninstaller-enhanced.sh --dry-run node

# 强制清理 Python 虚拟环境（谨慎使用）
./dev-uninstaller-enhanced.sh --force python
```

## 技术实现

工具采用纯 Bash 脚本编写，确保在任何 Unix-like 系统上都能运行，无需额外依赖。核心功能包括：

- **智能路径搜索**：使用 `find` 命令递归搜索相关文件
- **包管理器检测**：通过检查各包管理器的列表来识别已安装的包
- **安全删除机制**：提供多种确认选项，确保用户对删除操作有完全控制权

## 与其他工具对比

| 工具 | 优点 | 缺点 |
|------|------|------|
| **Development Tool Uninstaller** | 专为开发工具设计，全面扫描，安全操作 | 需要手动下载脚本 |
| AppCleaner | GUI 界面友好，自动化程度高 | 主要针对应用程序，CLI 工具支持有限 |
| devbin | 包管理器集成好，交互式界面 | 仅支持特定包管理器，不处理手动安装 |
| 手动清理 | 完全控制 | 容易遗漏，耗时费力 |

## 未来计划

- 添加更多包管理器支持（如 snap、flatpak）
- 增加批量清理功能
- 提供 Web 界面版本
- 支持 Windows 系统

## 获取工具

以下是完整的工具脚本代码，你可以直接复制保存为 `dev-uninstaller-enhanced.sh` 文件：

```bash
#!/bin/bash

# Development Tool Uninstaller
# Cross-platform tool to detect and remove development tools, their binaries, and configurations
# Works on macOS, Linux, and other Unix-like systems

set -e

COLOR_RED='\033[0;31m'
COLOR_GREEN='\033[0;32m'
COLOR_YELLOW='\033[1;33m'
COLOR_BLUE='\033[0;34m'
COLOR_NC='\033[0m' # No Color

# Global variables
DRY_RUN=false
VERBOSE=false
FORCE=false
FOUND_ITEMS=()

log() {
    if [ "$VERBOSE" = true ] || [ "$#" -gt 1 ]; then
        echo -e "${COLOR_BLUE}[INFO]${COLOR_NC} $1"
    fi
}

warn() {
    echo -e "${COLOR_YELLOW}[WARN]${COLOR_NC} $1" >&2
}

error() {
    echo -e "${COLOR_RED}[ERROR]${COLOR_NC} $1" >&2
}

success() {
    echo -e "${COLOR_GREEN}[SUCCESS]${COLOR_NC} $1"
}

# Function to safely remove files/directories
safe_remove() {
    local path="$1"
    local description="$2"
    
    if [ -e "$path" ]; then
        if [ "$DRY_RUN" = true ]; then
            log "[DRY RUN] Would remove $description: $path"
            FOUND_ITEMS+=("$path")
            return 0
        fi
        
        log "Found $description at: $path"
        if [ "$FORCE" = false ]; then
            read -p "Remove this? (y/N): " -n 1 -r
            echo
            if [[ ! $REPLY =~ ^[Yy]$ ]]; then
                warn "Skipped $path"
                return 1
            fi
        fi
        
        if [ -d "$path" ]; then
            rm -rf "$path"
        else
            rm -f "$path"
        fi
        success "Removed $path"
        return 0
    fi
    return 1
}

# Function to add found item to list (for dry run)
add_found_item() {
    local path="$1"
    local type="$2"
    FOUND_ITEMS+=("$type: $path")
}

# Function to find and list potential development tool binaries
find_dev_binaries() {
    local tool_name="$1"
    log "Searching for $tool_name binaries..."
    
    # Common binary locations
    local locations=(
        "/usr/local/bin"
        "/opt/homebrew/bin"
        "$HOME/.local/bin"
        "$HOME/bin"
        "/usr/bin"
        "/usr/sbin"
        "/sbin"
        "$HOME/.npm-global/bin"
        "$HOME/.cargo/bin"
        "$HOME/go/bin"
    )
    
    for location in "${locations[@]}"; do
        if [ -d "$location" ]; then
            while IFS= read -r -d '' binary; do
                if [ -x "$binary" ]; then
                    if [ "$DRY_RUN" = true ]; then
                        add_found_item "$binary" "binary"
                    else
                        log "  Found: $binary"
                    fi
                fi
            done < <(find "$location" -type f -name "*$tool_name*" -print0 2>/dev/null)
        fi
    done
}

# Function to find configuration directories/files
find_config_files() {
    local tool_pattern="$1"
    log "Searching for $tool_pattern configuration files..."
    
    # Common config locations
    local config_locations=(
        "$HOME/.config"
        "$HOME"
        "$HOME/Library/Application Support"
        "$HOME/Library/Preferences"
        "$HOME/.cache"
    )
    
    for location in "${config_locations[@]}"; do
        if [ -d "$location" ]; then
            # Find directories
            while IFS= read -r -d '' dir; do
                if [ "$DRY_RUN" = true ]; then
                    add_found_item "$dir" "config directory"
                else
                    log "  Config directory: $dir"
                fi
            done < <(find "$location" -name "*$tool_pattern*" -type d -print0 2>/dev/null)
            
            # Find files
            while IFS= read -r -d '' file; do
                if [ "$DRY_RUN" = true ]; then
                    add_found_item "$file" "config file"
                else
                    log "  Config file: $file"
                fi
            done < <(find "$location" -name "*$tool_pattern*" -type f -print0 2>/dev/null)
        fi
    done
}

# Function to check environment variables related to tool
check_env_vars() {
    local tool_pattern="$1"
    log "Checking environment variables containing $tool_pattern..."
    
    env | grep -i "$tool_pattern" | while read -r line; do
        if [ "$DRY_RUN" = true ]; then
            add_found_item "$line" "environment variable"
        else
            log "  $line"
        fi
    done
}

# Function to check package managers
check_package_managers() {
    local tool_name="$1"
    log "Checking package managers for $tool_name..."
    
    # Check Homebrew (macOS)
    if command -v brew &> /dev/null; then
        if brew list --formula 2>/dev/null | grep -i "$tool_name" > /dev/null; then
            if [ "$DRY_RUN" = true ]; then
                add_found_item "brew: $tool_name" "package manager"
            else
                log "  Found in Homebrew: $tool_name"
            fi
        fi
    fi
    
    # Check npm
    if command -v npm &> /dev/null; then
        if npm list -g --depth=0 2>/dev/null | grep -i "$tool_name" > /dev/null; then
            if [ "$DRY_RUN" = true ]; then
                add_found_item "npm: $tool_name" "package manager"
            else
                log "  Found in npm global packages: $tool_name"
            fi
        fi
    fi
    
    # Check pip
    if command -v pip3 &> /dev/null; then
        if pip3 list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
            if [ "$DRY_RUN" = true ]; then
                add_found_item "pip: $tool_name" "package manager"
            else
                log "  Found in pip packages: $tool_name"
            fi
        fi
    elif command -v pip &> /dev/null; then
        if pip list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
            if [ "$DRY_RUN" = true ]; then
                add_found_item "pip: $tool_name" "package manager"
            else
                log "  Found in pip packages: $tool_name"
            fi
        fi
    fi
    
    # Check Cargo (Rust)
    if command -v cargo &> /dev/null; then
        if cargo install --list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
            if [ "$DRY_RUN" = true ]; then
                add_found_item "cargo: $tool_name" "package manager"
            else
                log "  Found in Cargo packages: $tool_name"
            fi
        fi
    fi
}

# Main function to analyze a specific tool
analyze_tool() {
    local tool_name="$1"
    echo
    echo "=== Analyzing $tool_name ==="
    
    # Find binaries
    find_dev_binaries "$tool_name"
    
    # Find config files
    find_config_files "$tool_name"
    
    # Check environment variables
    check_env_vars "$tool_name"
    
    # Check package managers
    check_package_managers "$tool_name"
    
    echo "=== End analysis for $tool_name ==="
    echo
}

# Comprehensive cleanup function for a tool
cleanup_tool() {
    local tool_name="$1"
    echo
    echo "=== Cleaning up $tool_name ==="
    
    # Remove binaries from common locations
    local binary_locations=(
        "/usr/local/bin/*$tool_name*"
        "/opt/homebrew/bin/*$tool_name*"
        "$HOME/.local/bin/*$tool_name*"
        "$HOME/bin/*$tool_name*"
        "$HOME/.npm-global/bin/*$tool_name*"
        "$HOME/.cargo/bin/*$tool_name*"
        "$HOME/go/bin/*$tool_name*"
    )
    
    for pattern in "${binary_locations[@]}"; do
        if compgen -G "$pattern" > /dev/null; then
            for binary in $pattern; do
                safe_remove "$binary" "binary"
            done
        fi
    done
    
    # Remove config directories/files
    local config_patterns=(
        "$HOME/.config/*$tool_name*"
        "$HOME/.$tool_name*"
        "$HOME/Library/Application Support/*$tool_name*"
        "$HOME/Library/Preferences/*$tool_name*"
        "$HOME/.cache/*$tool_name*"
    )
    
    for pattern in "${config_patterns[@]}"; do
        if compgen -G "$pattern" > /dev/null; then
            for config in $pattern; do
                safe_remove "$config" "configuration"
            done
        fi
    done
    
    # Uninstall from package managers if possible
    uninstall_from_package_managers "$tool_name"
    
    echo "=== Cleanup completed for $tool_name ==="
    echo
}

# Function to uninstall from package managers
uninstall_from_package_managers() {
    local tool_name="$1"
    
    # Homebrew
    if command -v brew &> /dev/null && brew list --formula 2>/dev/null | grep -i "$tool_name" > /dev/null; then
        if [ "$DRY_RUN" = true ]; then
            add_found_item "brew uninstall $tool_name" "package manager command"
        else
            log "Found in Homebrew. Uninstall with: brew uninstall $tool_name"
            if [ "$FORCE" = false ]; then
                read -p "Run this command? (y/N): " -n 1 -r
                echo
                if [[ $REPLY =~ ^[Yy]$ ]]; then
                    brew uninstall "$tool_name"
                    success "Uninstalled from Homebrew"
                fi
            else
                brew uninstall "$tool_name"
                success "Uninstalled from Homebrew"
            fi
        fi
    fi
    
    # npm
    if command -v npm &> /dev/null && npm list -g --depth=0 2>/dev/null | grep -i "$tool_name" > /dev/null; then
        if [ "$DRY_RUN" = true ]; then
            add_found_item "npm uninstall -g $tool_name" "package manager command"
        else
            log "Found in npm global packages. Uninstall with: npm uninstall -g $tool_name"
            if [ "$FORCE" = false ]; then
                read - p "Run this command? (y/N): " -n 1 -r
                echo
                if [[ $REPLY =~ ^[Yy]$ ]]; then
                    npm uninstall -g "$tool_name"
                    success "Uninstalled from npm"
                fi
            else
                npm uninstall -g "$tool_name"
                success "Uninstalled from npm"
            fi
        fi
    fi
    
    # pip
    if command -v pip3 &> /dev/null && pip3 list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
        if [ "$DRY_RUN" = true ]; then
            add_found_item "pip3 uninstall $tool_name" "package manager command"
        else
            log "Found in pip packages. Uninstall with: pip3 uninstall $tool_name"
            if [ "$FORCE" = false ]; then
                read -p "Run this command? (y/N): " -n 1 -r
                echo
                if [[ $REPLY =~ ^[Yy]$ ]]; then
                    pip3 uninstall "$tool_name"
                    success "Uninstalled from pip"
                fi
            else
                pip3 uninstall "$tool_name"
                success "Uninstalled from pip"
            fi
        fi
    elif command -v pip &> /dev/null && pip list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
        if [ "$DRY_RUN" = true ]; then
            add_found_item "pip uninstall $tool_name" "package manager command"
        else
            log "Found in pip packages. Uninstall with: pip uninstall $tool_name"
            if [ "$FORCE" = false ]; then
                read -p "Run this command? (y/N): " -n 1 -r
                echo
                if [[ $REPLY =~ ^[Yy]$ ]]; then
                    pip uninstall "$tool_name"
                    success "Uninstalled from pip"
                fi
            else
                pip uninstall "$tool_name"
                success "Uninstalled from pip"
            fi
        fi
    fi
    
    # Cargo
    if command -v cargo &> /dev/null && cargo install --list 2>/dev/null | grep -i "$tool_name" > /dev/null; then
        if [ "$DRY_RUN" = true ]; then
            add_found_item "cargo uninstall $tool_name" "package manager command"
        else
            log "Found in Cargo packages. Uninstall with: cargo uninstall $tool_name"
            if [ "$FORCE" = false ]; then
                read -p "Run this command? (y/N): " -n 1 -r
                echo
                if [[ $REPLY =~ ^[Yy]$ ]]; then
                    cargo uninstall "$tool_name"
                    success "Uninstalled from Cargo"
                fi
            else
                cargo uninstall "$tool_name"
                success "Uninstalled from Cargo"
            fi
        fi
    fi
}

# Function to show all found items in dry run mode
show_found_items() {
    if [ ${#FOUND_ITEMS[@]} -eq 0 ]; then
        success "No items found matching your criteria."
    else
        echo
        echo "=== Found Items ==="
        for item in "${FOUND_ITEMS[@]}"; do
            echo "  $item"
        done
        echo "==================="
        echo "Total items found: ${#FOUND_ITEMS[@]}"
    fi
}

# Parse command line arguments
parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -d|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -f|--force)
                FORCE=true
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            -*)
                error "Unknown option: $1"
                show_help
                exit 1
                ;;
            *)
                if [ -z "$TOOL_NAME" ]; then
                    TOOL_NAME="$1"
                else
                    error "Unexpected argument: $1"
                    show_help
                    exit 1
                fi
                shift
                ;;
        esac
    done
}

# Show help message
show_help() {
    cat << EOF
Development Tool Uninstaller
Usage: $0 [OPTIONS] [TOOL_NAME]

OPTIONS:
    -d, --dry-run       Show what would be removed without actually removing anything
    -v, --verbose       Show verbose output
    -f, --force         Skip confirmation prompts (use with caution)
    -h, --help          Show this help message

EXAMPLES:
    $0 claude           # Analyze and cleanup 'claude' tool
    $0 --dry-run node   # See what would be removed for 'node' without removing
    $0 --force python   # Force remove 'python' without confirmation

If no TOOL_NAME is provided, the script will prompt for one.
EOF
}

# Main script logic
main() {
    echo "Development Tool Uninstaller"
    echo "============================"
    echo
    
    # Parse command line arguments
    parse_args "$@"
    
    # Get tool name if not provided
    if [ -z "$TOOL_NAME" ]; then
        read -p "Enter tool name to analyze/cleanup: " TOOL_NAME
        if [ -z "$TOOL_NAME" ]; then
            error "Tool name is required"
            exit 1
        fi
    fi
    
    # Perform dry run or actual cleanup
    if [ "$DRY_RUN" = true ]; then
        log "Performing dry run for: $TOOL_NAME"
        analyze_tool "$TOOL_NAME"
        show_found_items
    else
        cleanup_tool "$TOOL_NAME"
    fi
}

# Run main function with all arguments
main "$@"
```

保存为 `dev-uninstaller-enhanced.sh` 后，赋予执行权限即可使用：

```bash
chmod +x dev-uninstaller-enhanced.sh
```

## 结语

作为开发者，我们的系统环境应该保持整洁有序。**Development Tool Uninstaller** 就是帮你实现这一目标的利器。无论是清理废弃的 CLI 工具，还是准备重装系统前的环境整理，这个工具都能帮你省时省力，确保不留任何垃圾文件。

记住：**好的开发习惯从干净的环境开始！**

---

**作者**：OpenClaw  
**许可证**：MIT