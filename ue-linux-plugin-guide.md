# How to Compile & Use an Unreal Engine Plugin on Linux

A comprehensive guide for installing and compiling UE plugins on Linux distributions, specifically tested on Fedora 42 with UE 5.4.

## Overview

This guide addresses the common challenge Linux users face when trying to install Unreal Engine plugins that are normally distributed through Epic Games' Fab marketplace. Since the Epic Launcher isn't officially supported on Linux, we need to compile plugins from source.

## Prerequisites

### System Requirements
- Linux distribution (tested on Fedora 42)
- Unreal Engine 5.4 (or compatible version)
- Build tools and development environment

### Install Required Build Tools

For **Fedora/RHEL-based** systems:
```bash
sudo dnf install clang cmake make mono-devel git
```

For **Ubuntu/Debian-based** systems:
```bash
sudo apt update
sudo apt install clang cmake make mono-devel git build-essential
```

UE on Linux uses clang + cmake for compilation.

## Step-by-Step Guide

### Step 1: Create or Convert to C++ Project

**Important**: Blueprint-only projects cannot compile C++ plugins. You need a C++ project.

#### If you have a Blueprint-only project:
1. Open your project in UE Editor
2. Go to **File** → **New C++ Class**
3. Select any class type (e.g., Actor) and create it
4. This will convert your project to a C++ project with a `Source/` folder

#### If starting fresh:
Create a new C++ project from the UE launcher.

### Step 2: Obtain the Plugin Source

1. Find the plugin's GitHub repository or source
2. Download or clone the plugin source code
3. Ensure you have the complete plugin folder structure:
   ```
   PluginName/
   ├── PluginName.uplugin
   ├── Source/
   │   └── [source files]
   └── [other plugin files]
   ```

### Step 3: Install the Plugin

1. Navigate to your UE project directory
2. Create a `Plugins/` folder if it doesn't exist:
   ```bash
   mkdir -p YourProject/Plugins/
   ```
3. Copy the entire plugin folder into `Plugins/`:
   ```bash
   cp -r PluginName/ YourProject/Plugins/
   ```

Your project structure should look like:
```
YourProject/
├── Content/
├── Source/
├── Plugins/
│   └── PluginName/
│       ├── PluginName.uplugin
│       ├── Source/
│       └── [other files]
└── YourProject.uproject
```

### Step 4: Compile the Plugin

#### Method 1: Through UE Editor (Recommended)
1. Open your UE project
2. UE will detect the new plugin and prompt: "Missing [PluginName] modules. Would you like to rebuild them now?"
3. Click **Yes**
4. UE will compile the plugin automatically
5. Wait for compilation to complete

#### Method 2: Command Line Build
If you built UE from source and have build scripts:

1. Navigate to your project directory
2. Regenerate project files:
   ```bash
   ./GenerateProjectFiles.sh
   ```
3. Build the project:
   ```bash
   make
   ```

### Step 5: Verify Installation

1. In UE Editor, go to **Edit** → **Plugins**
2. Search for your plugin name
3. Ensure it's enabled and shows as "Installed"
4. Test the plugin functionality according to its documentation

## Understanding Compilation Results

After successful compilation, you'll find these files in the plugin's `Binaries/Linux/` folder:

- `libUnrealEditor-PluginName.so` - Main plugin binary
- `libUnrealEditor-PluginName.debug` - Debug symbols
- `libUnrealEditor-PluginName.sym` - Symbol file
- `UnrealEditor.modules` - Module information

## Sharing Compiled Plugins

### For Binary Distribution (No Source Code)

To share a compiled plugin without source code:

1. Create a clean plugin folder with only:
   ```
   PluginName/
   ├── PluginName.uplugin
   └── Binaries/
       └── Linux/
           ├── libUnrealEditor-PluginName.so
           ├── libUnrealEditor-PluginName.debug
           ├── libUnrealEditor-PluginName.sym
           └── UnrealEditor.modules
   ```

2. Remove these folders (optional):
   - `Source/` (if you don't want to distribute source)
   - `Intermediate/` (build cache, safe to delete)

### Installing Pre-compiled Plugins

To use a pre-compiled plugin in a **Blueprint-only project**:

1. Copy the entire plugin folder to `YourProject/Plugins/`
2. Ensure it contains both `.uplugin` file and `Binaries/` folder
3. Open your project - UE will load the compiled plugin automatically
4. No compilation required

## Troubleshooting

### "Engine modules are out of date" Error
- **Cause**: Plugin needs compilation but UE Editor cannot compile while running
- **Solution**: Close UE Editor and follow the compilation steps above

### Plugin Not Appearing in Plugin List
- Verify the `.uplugin` file is present in the plugin folder
- Check that the plugin is in the correct `Plugins/` directory
- Ensure plugin compatibility with your UE version

### Compilation Fails
- Verify all build tools are installed
- Check that you have a C++ project (not Blueprint-only)
- Ensure sufficient disk space and memory
- Check UE and plugin version compatibility

### Missing Dependencies
Some plugins may require additional libraries:
```bash
# Common additional packages that might be needed
sudo dnf install libX11-devel libXrandr-devel libXinerama-devel libXcursor-devel
```

## Creating a Redistributable Plugin Package

To create a clean, shareable plugin package:

```bash
#!/bin/bash
# Script to create a binary-only plugin package

PLUGIN_NAME="YourPluginName"
SOURCE_PATH="YourProject/Plugins/$PLUGIN_NAME"
OUTPUT_PATH="./DistributionPlugins"

mkdir -p "$OUTPUT_PATH/$PLUGIN_NAME"

# Copy essential files
cp "$SOURCE_PATH/$PLUGIN_NAME.uplugin" "$OUTPUT_PATH/$PLUGIN_NAME/"
cp -r "$SOURCE_PATH/Binaries" "$OUTPUT_PATH/$PLUGIN_NAME/" 2>/dev/null || echo "No Binaries folder found"
cp -r "$SOURCE_PATH/Resources" "$OUTPUT_PATH/$PLUGIN_NAME/" 2>/dev/null || echo "No Resources folder found"

# Create archive
cd "$OUTPUT_PATH"
tar -czf "$PLUGIN_NAME-Linux-Binary.tar.gz" "$PLUGIN_NAME/"

echo "Plugin package created: $OUTPUT_PATH/$PLUGIN_NAME-Linux-Binary.tar.gz"
```

## Best Practices

1. **Version Compatibility**: Always ensure plugin and UE versions match
2. **Backup Projects**: Create backups before installing new plugins
3. **Documentation**: Keep notes of which plugins are installed and their versions
4. **Clean Builds**: Remove `Intermediate/` folders when sharing plugins
5. **Testing**: Test plugin functionality thoroughly after compilation

## Example: AutoSizeComments Plugin

Here's a real example using the AutoSizeComments plugin:

1. **Source**: https://github.com/AutoSizeComments/AutoSizeComments
2. **Target**: UE 5.4 on Fedora 42
3. **Process**:
   ```bash
   # Clone or download the plugin
   git clone https://github.com/AutoSizeComments/AutoSizeComments.git
   
   # Copy to your C++ project
   cp -r AutoSizeComments YourProject/Plugins/
   
   # Open UE project and compile when prompted
   # Plugin will be available in Edit → Plugins
   ```

## Conclusion

Compiling UE plugins on Linux requires a few extra steps compared to Windows, but the process is straightforward once you understand the requirements. The key points are:

- Use C++ projects for plugin compilation
- Ensure proper build tools are installed
- Follow the correct folder structure
- Understand the difference between source and binary distribution

This workflow enables Linux users to access the vast ecosystem of UE plugins despite not having official Epic Launcher support.

---

*This guide was created based on real-world experience compiling UE plugins on Linux. For updates and community contributions, please visit the GitHub repository.*