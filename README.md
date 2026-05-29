# Reverse Tools MCP Server

MCP Server for reverse engineering tools integration with AI coding assistants.

## Supported Tools

- **dnlib Backend**: Static analysis of .NET assemblies
- **x64dbg Backend**: (Planned) Debugger automation
- **Cheat Engine Backend**: (Planned) Memory scanning

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/7mengyu/nixiang_mcp.git
cd nixiang_mcp
```

### 2. Create virtual environment and install dependencies

```bash
python -m venv venv
# Windows
venv\Scripts\pip install -r requirements.txt
# Linux/macOS
./venv/bin/pip install -r requirements.txt
```

### 3. Download dnlib.dll

**Method 1: Using curl (Recommended)**

```bash
# Windows (PowerShell/CMD)
curl -L -o dnlib.nupkg "https://www.nuget.org/api/v2/package/dnlib"
# Extract (requires unzip)
unzip -o dnlib.nupkg -d dnlib-temp
cp dnlib-temp/lib/net45/dnlib.dll .
rm -rf dnlib-temp dnlib.nupkg

# Linux/macOS
curl -L -o dnlib.nupkg "https://www.nuget.org/api/v2/package/dnlib"
unzip -o dnlib.nupkg -d dnlib-temp
cp dnlib-temp/lib/net45/dnlib.dll .
rm -rf dnlib-temp dnlib.nupkg
```

**Method 2: Manual download**

1. Visit https://www.nuget.org/packages/dnlib
2. Click "Download package"
3. Rename `.nupkg` to `.zip`
4. Extract and copy `lib/net45/dnlib.dll` to project root

**Method 3: From GitHub releases**

1. Visit https://github.com/0xd4d/dnlib/releases
2. Download the latest release zip
3. Find and extract `dnlib.dll`

## Configuration

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "mcpServers": {
    "reverse-tools": {
      "command": "C:\\path\\to\\nixiang_mcp\\venv\\Scripts\\python",
      "args": ["-m", "src.server"],
      "cwd": "C:\\path\\to\\nixiang_mcp"
    }
  }
}
```

After configuration, **restart Claude Code** to load the MCP server.

## Usage

### Available Tools

| Tool | Description |
|------|-------------|
| `dnlib_set_path` | Initialize dnlib with path to dnlib.dll (must call first) |
| `dnlib_load_assembly` | Load a .NET assembly (.dll/.exe) |
| `dnlib_list_types` | List all types in assembly |
| `dnlib_get_type` | Get type details (methods, fields, properties) |
| `dnlib_search_types` | Search types by name pattern |
| `dnlib_search_methods` | Search methods by name pattern |
| `dnlib_decompile_method` | Decompile method to IL code |
| `dnlib_get_entry_point` | Get assembly entry point (Main) |
| `dnlib_list_resources` | List embedded resources |

### Step-by-Step Usage Guide

#### Step 1: Initialize dnlib

After restarting Claude Code, tell the AI to initialize dnlib:

```
初始化 dnlib，路径是 C:\Users\scydr\Desktop\123\456\reverse-tools-mcp\dnlib.dll
```

Or in English:

```
Initialize dnlib with path: C:\path\to\your\dnlib.dll
```

#### Step 2: Load Target Assembly

Load the .NET assembly you want to analyze:

```
加载程序集 D:\Games\SomeGame\Game.exe
```

Or:

```
Load assembly: D:\Games\SomeGame\Game.exe
```

The tool will return assembly info including name, version, and modules.

#### Step 3: Explore Types

List all types in the assembly:

```
列出所有类型
```

Or search for specific types:

```
搜索包含 "Player" 的类型
```

#### Step 4: Analyze a Class

Get detailed information about a specific class:

```
查看 PlayerManager 类的详细信息
```

This returns:
- Class name, namespace, base type
- Interfaces implemented
- All methods with signatures
- All fields with types
- All properties

#### Step 5: Decompile Methods

Decompile a method to IL code for detailed analysis:

```
反编译 PlayerManager.UpdatePlayer 方法
```

The output is IL assembly code showing the method's implementation.

#### Step 6: Find Entry Point

Locate the Main method:

```
获取程序入口点
```

### Complete Example Conversation

```
User: 初始化 dnlib，路径是 C:\Tools\dnlib.dll

AI: [Calls dnlib_set_path tool]
dnlib initialized successfully from: C:\Tools\dnlib.dll

User: 加载程序集 D:\Games\MyGame\MyGame.exe

AI: [Calls dnlib_load_assembly tool]
{
  "name": "MyGame",
  "full_name": "MyGame, Version=1.0.0.0",
  "version": "1.0.0.0",
  "modules": ["MyGame.exe"]
}

User: 搜索包含 "Health" 的类型

AI: [Calls dnlib_search_types tool]
[
  "MyGame.PlayerHealth",
  "MyGame.HealthSystem",
  "MyGame.UI.HealthBar"
]

User: 查看 MyGame.PlayerHealth 类的详细信息

AI: [Calls dnlib_get_type tool]
{
  "name": "PlayerHealth",
  "full_name": "MyGame.PlayerHealth",
  "namespace": "MyGame",
  "base_type": "System.Object",
  "methods": [
    {"name": "GetHealth", "return_type": "System.Int32", "is_static": false},
    {"name": "SetHealth", "return_type": "System.Void", "is_static": false},
    {"name": "TakeDamage", "return_type": "System.Void", "is_static": false}
  ],
  "fields": [
    {"name": "currentHealth", "type": "System.Int32", "is_static": false},
    {"name": "maxHealth", "type": "System.Int32", "is_static": false}
  ]
}

User: 反编译 PlayerHealth.SetHealth 方法

AI: [Calls dnlib_decompile_method tool]
; Method: System.Void MyGame.PlayerHealth.SetHealth(System.Int32)
; Token: 0x06000102

IL_0000: ldarg.0
IL_0001: ldarg.1
IL_0002: stfld System.Int32 MyGame.PlayerHealth::currentHealth
IL_0007: ret
```

### Tips for Game Reverse Engineering

1. **Find key classes**: Search for terms like "Player", "Health", "Money", "Inventory", "Weapon"
2. **Locate modifying methods**: Look for "Add", "Set", "Update", "Increase" method names
3. **Analyze IL code**: Simple methods like `SetHealth(value)` are easy to understand in IL
4. **Find entry points**: Entry point helps understand game initialization flow
5. **Use with other tools**: Combine with dnSpy for visual decompilation, then use MCP for quick searches

## Requirements

- Python 3.10+
- .NET Runtime (for dnlib backend via pythonnet)
- Windows OS (for pythonnet CLR interop)