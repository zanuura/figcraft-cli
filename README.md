# FigCraft AI Agent Skill

An AI agent skill that teaches coding agents how to design with [FigCraft](https://github.com/storytold/figcraft)—an open-source, offline, Figma-like design tool written in Rust—using its built-in Model Context Protocol (MCP) server.

The skill and the MCP server are two distinct parts, and you need both:

| Component | What it does | Location |
|---|---|---|
| **Skill** (this repository) | Instructions: conventions, workflow, auto layout rules, best practices | `~/.agents/skills/figcraft/` |
| **MCP Server** (bundled with FigCraft) | Tools: `create_node`, `export`, `screenshot`, `set_style`… | Configured in your AI agent |

- **Without the skill**, an agent has the tools but must guess design conventions, coordinate systems, and layout hierarchies.
- **Without the MCP server**, an agent knows what to do conceptually but cannot interact with FigCraft.

---

## 1. Install FigCraft & FigCraft CLI

FigCraft provides both a desktop GUI application and a headless command-line interface (`figcraft-cli`). The CLI binary includes the MCP server.

### Option A: macOS

#### 1. Pre-built CLI Binary (Recommended)
1. Go to the [FigCraft Releases page](https://github.com/storytold/figcraft/releases).
2. Download `figcraft-cli-<version>-macos-universal.zip`.
3. Unzip and move the binary to your `$PATH`:
   ```bash
   unzip figcraft-cli-*-macos-universal.zip
   chmod +x figcraft-cli
   sudo mv figcraft-cli /usr/local/bin/
   ```
4. Verify installation:
   ```bash
   figcraft-cli --version
   ```

#### 2. Desktop GUI App (Optional)
If you want to view designs live in the GUI:
- **Via Homebrew:**
  ```bash
  brew install --cask figcraft
  ```
- **Via DMG:** Download `figcraft-<version>-macos-universal.dmg` from [Releases](https://github.com/storytold/figcraft/releases) and drag FigCraft to `/Applications`.

---

### Option B: Windows

#### 1. Installer (Recommended)
1. Download the Windows installer (`figcraft-<version>-windows-x64.msi` or `figcraft-<version>-windows-arm64.msi`) from [FigCraft Releases](https://github.com/storytold/figcraft/releases).
2. Run the installer. By default, it installs both the GUI app and the CLI to:
   ```text
   C:\Program Files\FigCraft\figcraft-cli.exe
   C:\Program Files\FigCraft\figcraft.exe
   ```
3. Add `C:\Program Files\FigCraft` to your system `PATH` environment variable so you can run `figcraft-cli` from any PowerShell or Command Prompt window.

#### 2. Portable ZIP
Alternatively, download `figcraft-<version>-windows-x64-portable.zip`, extract it to a folder of your choice (e.g. `C:\Tools\FigCraft`), and add that folder to your `PATH`.

---

### Option C: Linux

#### 1. Package Managers & Pre-built Packages
Download the package corresponding to your Linux distribution from [Releases](https://github.com/storytold/figcraft/releases):
- **Debian / Ubuntu:**
  ```bash
  sudo dpkg -i figcraft-<version>-linux-x86_64.deb
  # or for ARM64:
  # sudo dpkg -i figcraft-<version>-linux-aarch64.deb
  ```
- **Fedora / RHEL:**
  ```bash
  sudo rpm -i figcraft-<version>-linux-x86_64.rpm
  ```
- **Flatpak / AppImage:** Available on the releases page.

#### 2. Standalone Archive
1. Download `figcraft-<version>-linux-x86_64.tar.gz`.
2. Extract and place `figcraft-cli` in your executable path:
   ```bash
   tar -xzf figcraft-*.tar.gz
   sudo install -m 755 figcraft-cli /usr/local/bin/figcraft-cli
   ```

---

### Option D: Build from Source (Any Platform)

If pre-built binaries are not suitable or you prefer building from source, you can compile FigCraft using [Rust & Cargo](https://rustup.rs/):

```bash
git clone https://github.com/storytold/figcraft
cd figcraft
cargo build --release -p figcraft-cli
```

The resulting binary will be at:
- **macOS / Linux:** `target/release/figcraft-cli`
- **Windows:** `target\release\figcraft-cli.exe`

Copy the binary into a directory in your system `$PATH`.

---

## 2. Install the Skill

### Using the `skills` CLI:
```bash
npx skills add <your-username>/figcraft-skill -g
```

### Manual Installation:
Copy this skill directory into your AI agent's skill directory:

- **macOS / Linux:**
  ```bash
  mkdir -p ~/.agents/skills/figcraft
  cp -r . ~/.agents/skills/figcraft/
  ```

- **Windows (PowerShell):**
  ```powershell
  New-Item -ItemType Directory -Force -Path "$HOME\.agents\skills\figcraft"
  Copy-Item -Recurse * "$HOME\.agents\skills\figcraft\"
  ```

Verify the skill is recognized:
```bash
npx skills ls -g
```

---

## 3. Register the MCP Server

The command to start the MCP server is always:
```bash
figcraft-cli mcp
```

Configure your specific agent using the examples below:

### macOS / Linux

#### Standard JSON configuration (`mcpServers`)
Used by tools like Cursor, Claude Desktop, Windsurf, Cline, Roo-Code, etc.:
```json
{
  "mcpServers": {
    "figcraft": {
      "command": "figcraft-cli",
      "args": ["mcp"]
    }
  }
}
```
*(If `figcraft-cli` is not in your system `$PATH`, provide the absolute path, e.g. `/usr/local/bin/figcraft-cli`)*.

#### Claude Code (CLI)
```bash
claude mcp add figcraft -- figcraft-cli mcp
```

---

### Windows

#### Standard JSON configuration (`mcpServers`)
```json
{
  "mcpServers": {
    "figcraft": {
      "command": "C:\\Program Files\\FigCraft\\figcraft-cli.exe",
      "args": ["mcp"]
    }
  }
}
```

#### Claude Code (CLI)
```powershell
claude mcp add figcraft -- "C:\Program Files\FigCraft\figcraft-cli.exe" mcp
```

#### Oh My Pi (OMP)
Run `/mcp add`, choose *User level*, and enter:
```text
Name:      figcraft
Transport: stdio
Command:   C:\Program Files\FigCraft\figcraft-cli.exe
Arguments: mcp
```

---

## 4. Operational Modes: Headless vs. Live

### Headless Mode (Default)
In headless mode, the agent creates and operates on design documents entirely in memory:
- **Pros:** Fast, lightweight, works on servers and headless CI/CD environments without opening any GUI window.
- **Important:** Files are not persisted to disk automatically. Always instruct your agent where to save the `.figcraft` document or where to export PNG/SVG/PDF files.

### Live Mode (Attach to Desktop App)
Watch the AI design in real-time on your screen!

1. Start the FigCraft desktop application with the control port enabled:
   - **macOS / Linux:**
     ```bash
     figcraft --control 7979
     ```
   - **Windows:**
     ```powershell
     & "C:\Program Files\FigCraft\figcraft.exe" --control 7979
     ```
2. Configure the MCP server with the `--attach` flag:
   ```json
   {
     "mcpServers": {
       "figcraft": {
         "command": "figcraft-cli",
         "args": ["mcp", "--attach", "7979"]
       }
     }
   }
   ```
Every change made by the AI agent will instantly update in the open FigCraft window.

### Sandbox & Security Restrictions (Optional)
To restrict the directories that the agent is allowed to read from and write to, pass root boundary flags:
```text
figcraft-cli mcp --automation-read-root /path/to/designs --automation-write-root /path/to/output
```

---

## 5. Quick Test

Test your setup by prompting your AI agent:

> *"Use FigCraft to design a 375×812 mobile login screen with a logo header, email and password input fields, a primary blue 'Sign In' button, and social login options. Save the project to `./designs/login.figcraft` and export a preview PNG beside it."*

Check the output folder to verify the generated `.figcraft` project file and PNG render.

---

## Updating the MCP Reference

The reference file `figcraft-mcp.md` (or `references/mcp.md`) contains the tool schemas and command API derived from FigCraft's upstream documentation.

To update it with the latest upstream specifications:

- **macOS / Linux:**
  ```bash
  curl -sSL https://raw.githubusercontent.com/storytold/figcraft/HEAD/docs/mcp.md -o figcraft-mcp.md
  ```

- **Windows (PowerShell):**
  ```powershell
  Invoke-WebRequest https://raw.githubusercontent.com/storytold/figcraft/HEAD/docs/mcp.md -OutFile figcraft-mcp.md
  ```

---

## Credits & License

- Upstream documentation and MCP specifications © 2026 ArtCraft Team and FigCraft contributors, dual-licensed under MIT or Apache-2.0. See upstream [`LICENSE-MIT`](https://github.com/storytold/figcraft/blob/main/LICENSE-MIT) and [`LICENSE-APACHE`](https://github.com/storytold/figcraft/blob/main/LICENSE-APACHE).
- ArtCraft name and logo are trademarks of the ArtCraft Team.
- This repository is an independent community skill and is not officially affiliated with the FigCraft project.
