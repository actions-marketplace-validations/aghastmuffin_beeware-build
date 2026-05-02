# beeware-build
> [!WARNING]
> By default this action activates on release (when you version your files with a tag), mostly due to compute resources (MacOS-latest takes up 10x compute minutes, so it's very taxing on your credits), because of this it attaches the content to the versioned release, this action will require some changing if you want it to be on: push.

[Build](https://briefcase.beeware.org/en/v0.3.26/reference/commands/build/) your [beeware](https://beeware.org/) apps to MacOS, iOS, Android, Windows, and Linux with one action.
I am a highschooler, so this project (as with all of my others) isn't the primary focus of my attention. I use this personally, so it works for my needs, I just wanted to share it to hopefully make developing cross-platform just that little bit easier. 

# A Couple Things to Know
## 📖 Usage
 
### Basic Example
 
```yaml
name: Build My App
on: [push]
 
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - platform: macOS
            os: macos-latest
          - platform: Windows
            os: windows-latest
          - platform: Linux
            os: ubuntu-latest
          - platform: Android
            os: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build ${{ matrix.platform }} Package
        uses: YOUR_USERNAME/beeware-builder-action@v1
        with:
          platform: ${{ matrix.platform }}
      
      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.platform }}-app
          path: dist/*
```
 
### Advanced: Create Release on Tag
 
```yaml
name: Release
on:
  push:
    tags:
      - 'v*'
 
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - platform: macOS
            os: macos-latest
          - platform: Windows
            os: windows-latest
          - platform: Linux
            os: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Package
        id: build
        uses: YOUR_USERNAME/beeware-builder-action@v1
        with:
          platform: ${{ matrix.platform }}
          python-version: '3.11'
      
      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          files: ${{ steps.build.outputs.artifact-path }}
```
 
### Custom Configuration
 
```yaml
- uses: YOUR_USERNAME/beeware-builder-action@v1
  with:
    # Target platform (required)
    platform: 'macOS'
    
    # Python version (optional, default: 3.11)
    python-version: '3.12'
    
    # Briefcase version (optional, default: latest)
    briefcase-version: '0.3.17'
    
    # Output directory (optional, default: dist)
    output-path: 'build/output'
```
 
## 📥 Inputs
 
| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `platform` | ✅ Yes | - | Target platform: `macOS`, `Windows`, `Linux`, or `Android` |
| `python-version` | ❌ No | `3.11` | Python version to use for building |
| `briefcase-version` | ❌ No | `latest` | Specific Briefcase version to install |
| `output-path` | ❌ No | `dist` | Directory where built packages are saved |
 
## 📤 Outputs
 
| Output | Description |
|--------|-------------|
| `artifact-path` | Full path to the built package file (e.g., `dist/MyApp-1.0.0.dmg`) |
| `artifact-name` | Filename of the built package (e.g., `MyApp-1.0.0.dmg`) |
 
## 🛠️ Requirements
 
Your repository must contain:
- `pyproject.toml` with Briefcase configuration
- `src/` directory with your BeeWare app code
- Standard BeeWare/Toga project structure
## 🎯 Supported Platforms
 
| Platform | Runner OS | Output Format | Notes |
|----------|-----------|---------------|-------|
| macOS | `macos-latest` | `.dmg` | Universal binary support |
| Windows | `windows-latest` | `.msi` | Requires WiX Toolset (auto-installed) |
| Linux | `ubuntu-latest` | `.AppImage` | Compatible with most distros |
| Android | `ubuntu-latest` | `.apk` | Unsigned APK (sign separately) |


# Troubleshooting

<details>
<summary><b>Build fails on Linux with missing dependencies</b></summary>
The action automatically installs required system libraries. If you need additional packages:
 
```yaml
- name: Install Extra Dependencies
  run: sudo apt-get install -y your-package
  
- uses: YOUR_USERNAME/beeware-builder-action@v1
  with:
    platform: Linux
```
</details>
<details>
<summary><b>Android build hangs or fails</b></summary>
Ensure Java 17 is available. The action installs it automatically, but if you're using a custom Java setup:
 
```yaml
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '17'
```
</details>
<details>
<summary><b>Artifact path is empty</b></summary>
Check that your `pyproject.toml` has correct Briefcase configuration:
 
```toml
[tool.briefcase.app.myapp]
formal_name = "My App"
bundle = "com.example"
```
</details>
