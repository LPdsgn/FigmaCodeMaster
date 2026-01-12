# 🚀 FigmaCodeMaster

<div align="center">

![Figma Code Master](https://img.shields.io/badge/Figma-Code%20Master-F24E1E?style=for-the-badge&logo=figma&logoColor=white)
![Version](https://img.shields.io/badge/version-1.0.0-2E86AB?style=for-the-badge)
![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-D97757?style=for-the-badge)

**Convert Figma designs to production-ready code with pixel-perfect fidelity.**

[Prerequisites](#-prerequisites) • [Installation](#-installation) • [Usage](#-usage) • [Troubleshooting](#-troubleshooting)

</div>

---

## 📋 Prerequisites

Before you begin, ensure you have the following installed:

| Requirement | Verification Command | Note |
|-------------|----------------------|------|
| 🤖 **Claude Code** | `claude --version` | Must be version >= 1.0.33 |
| 🐍 **Python** | `python3 --version` | 3.10+ (for utility scripts) |
| 📦 **Node.js** | `node --version` | 18+ (for project analysis) |
| 🎨 **Figma Account** | - | Access to the designs you want to convert |

---

## 📥 Installation

### Method 1️⃣ — Direct Installation from GitHub (Recommended)

This method installs the plugin directly from GitHub without manual cloning.

1. **Configure Figma MCP Server (one-time setup):**
   ```bash
   claude mcp add --transport http figma https://mcp.figma.com/mcp
   ```

2. **Authenticate with Figma:**
   - Open Claude Code: `claude`
   - Type `/mcp` and press Enter
   - Locate **figma** in the list and select **Authenticate**
   - Follow the browser prompt to authorize access

3. **Add the Marketplace and Install:**
   ```bash
   # Add the marketplace from GitHub
   claude plugin marketplace add LPdsgn/FigmaCodeMaster
   
   # Install the plugin
   claude plugin install figma-code-master@lpdsgn
   ```

4. **Verify Installation:**
   ```bash
   # Check that the marketplace was added
   claude plugin marketplace list
   
   # Then open Claude Code to verify the plugin
   claude
   # Inside Claude Code, type: /plugin
   ```
   ✅ You should see `lpdsgn` in the marketplace list and `figma-code-master` in the `/plugin` interface.

---

### Method 2️⃣ — Manual Installation from Cloned Repository

Use this method if you want to modify the plugin or use a local version.

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/LPdsgn/FigmaCodeMaster.git
   cd FigmaCodeMaster
   ```

2. **Configure Figma MCP Server (one-time setup):**
   ```bash
   claude mcp add --transport http figma https://mcp.figma.com/mcp
   ```
   Then authenticate via `/mcp` in Claude Code (see Method 1, step 2).

3. **Add the Marketplace and Install:**
   ```bash
   # Windows
   claude plugin marketplace add ./
   
   # Linux/Mac
   claude plugin marketplace add ./
   
   # Install the plugin
   claude plugin install figma-code-master@lpdsgn
   ```

4. **Verify Installation:**
   ```bash
   # Check that the marketplace was added
   claude plugin marketplace list
   
   # Then open Claude Code to verify the plugin
   claude
   # Inside Claude Code, type: /plugin
   ```
   ✅ You should see `lpdsgn` in the marketplace list and `figma-code-master` in the `/plugin` interface.

> **💡 Tip:** Run `/doctor` inside Claude Code to verify that the Figma MCP server is connected properly.

---

## 🎯 Usage

Once installed, usage is simple and works in any project directory.

### 1. Connect to Figma Designs

1. Copy the link to a Figma frame or file.
   *Ensure the link includes `?node-id=...` for specific frames.*

2. Start Claude Code in your project folder:
   ```bash
   cd my-react-project
   claude
   ```

### 2. Available Commands

| Command | Description | Example |
|---------|-------------|---------|
| `/figma-code-master:design-to-code` | Convert a Figma frame to code | `/figma-code-master:design-to-code https://figma.com/...` |
| `/figma-code-master:extract-tokens` | Extract design tokens (colors, fonts) | `/figma-code-master:extract-tokens https://figma.com/...` |
| `/figma-code-master:validate-design` | Validate implementation vs Figma | `/figma-code-master:validate-design components/Hero.tsx` |

> **Tip:** You can often just ask similarly in natural language: "Convert this design to code: [LINK]"

### 3. Workflow Example

1. **Analyze**: The plugin analyzes your `package.json` to detect Framework (Next.js, Vue, etc.) and styling (Tailwind v3 vs v4).
2. **Extract**: It pulls design data, tokens, and screenshots from Figma via MCP.
3. **Generate**: It creates component code that matches your existing project structure.
4. **Validate**: Use `/validate-design` to check pixel accuracy.

---

## 🔄 Updating the Plugin

To update to the latest version:

**Method 1 (Direct from GitHub):**
```bash
# Update the marketplace
claude plugin marketplace update lpdsgn

# Update the plugin
claude plugin update figma-code-master@lpdsgn
```

**Method 2 (Local Repository):**
```bash
# Pull latest changes
cd FigmaCodeMaster
git pull origin main

# Update the marketplace
claude plugin marketplace update lpdsgn

# Update the plugin
claude plugin update figma-code-master@lpdsgn
```

---

## 🔧 Troubleshooting

### ❌ "MCP Server not connected"
Run `/mcp` in Claude to check connection status. If disconnected, select it and choose **Reconnect**.

### ❌ "Plugin not found"
Verify the marketplace is added correctly:
```bash
claude plugin marketplace list
```
If missing, re-add it using the absolute path to the plugin folder.

### ❌ "Token conflict" during generation
If the plugin finds a conflict (e.g., Figma uses `blue-500` but your code uses a different hex for `blue-500`), it will pause and ask you for a decision.

---

## 📞 Support

For issues or feature requests, please contact the maintainer or open an issue in the repository.

---

<div align="center">

**FigmaCodeMaster v1.0.0**

*January 2026*

---

Made with ❤️ by Boosha AI

</div>
