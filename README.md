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

This guide assumes you are installing the plugin from a local repository (e.g., distributed by your team).

### Step 1️⃣ — Prepare the Plugin Directory

Ensure you have the plugin files available locally.
If you are cloning from a repository:

```bash
# Clone the plugin repository
git clone https://github.com/boosha-ai/figma-code-master.git
cd figma-code-master
```

### Step 2️⃣ — Configure Figma MCP Server (CRITICAL)

This plugin relies on the Figma Model Context Protocol (MCP) server. Each team member must configure this **once** on their machine.

**A. Add the Server**
Run this command in your terminal to register the Figma MCP server with Claude Code:

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

**B. Authenticate**
1. Open Claude Code: `claude`
2. Type `/mcp` and press Enter.
3. Locate **figma** in the list.
4. Select **Authenticate** (or ensure status is Connected).
5. Follow the browser prompt to authorize Figma access.

**C. Verify Connection**
Run `/doctor` inside Claude Code or check `/mcp` again. You should see a green indicator next to "figma".

### Step 3️⃣ — Install Plugin via Marketplace

Add the local directory as a marketplace to Claude Code, then install the plugin. This ensures it's available globally across all your projects.

1. **Add the Marketplace:**
   ```bash
   # Replace /path/to/figma-code-master with the absolute path to where you cloned the repo
   claude plugin marketplace add /path/to/figma-code-master
   ```
   *Expected output:* `Successfully added marketplace: figma-code-master`

2. **Install the Plugin:**
   ```bash
   claude plugin install figma-code-master@figma-code-master
   ```
   *Expected output:* `Successfully installed plugin: figma-code-master@figma-code-master`

3. **Verify Installation:**
   ```bash
   claude plugin list
   ```
   You should see `figma-code-master` in the active plugins list.

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

To update to the latest version distributed by your team:

```bash
# 1. Pull latest changes
cd figma-code-master
git pull origin main

# 2. Update the marketplace
claude plugin marketplace update figma-code-master

# 3. Update the plugin
claude plugin update figma-code-master@figma-code-master
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
