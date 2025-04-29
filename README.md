# 🧠 Smart Workspace Manager

> Supercharge your productivity by connecting your personal workspace to AI.

Smart Workspace Manager is a secure, local-first [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that exposes your files, notes, and projects to AI assistants like Claude Desktop, Cursor IDE, and custom LLM workflows.

Built for macOS (Silicon) developers, it bridges your private filesystem with Large Language Models, allowing AI to **search**, **read**, **summarize**, **create**, and **organize** your documents — with full user control and privacy.

---

## ✨ Key Features

- 📚 **Resource Discovery**: Files, folders, and notes exposed as structured MCP Resources.
- 🔎 **Smart Search**: Search workspace contents by name, path, or keywords.
- 📝 **Document Summarization**: Quickly summarize meeting notes, articles, or project plans.
- 🗂️ **File Creation and Organization**: Create, rename, move, and (optionally) delete files.
- ⚙️ **Configurable and Secure**: Full control over file types, size limits, and writable actions.
- 🔌 **MCP Compatible**: Connects to any MCP-enabled client without custom integration.
- 🛡️ **Local-First Privacy**: Your data never leaves your machine.

---

## 🛠 How It Works

- Runs an MCP-compliant server on your Mac.
- Connects to LLM applications through standard I/O or HTTP (future).
- Represents your workspace directory (e.g., `~/Documents/Workspace`) as:
  - **Resources** (files and metadata)
  - **Tools** (search, summarize, create files, etc.)
- Allows the LLM, under user control, to **query**, **analyze**, and **modify** local content.
- All destructive actions (delete, move) are gated behind secure configuration.

---

## 🏛️ Technical Overview

| Component       | Description                                  |
|-----------------|----------------------------------------------|
| MCP Server      | Built using the official Python MCP SDK      |
| Transport       | Stdio (local connections), SSE (planned)     |
| Resources       | Files (.txt, .md, .pdf) exposed via URIs      |
| Tools           | JSON-schema validated functions (e.g., search, summarize, create) |
| Configuration   | `.env` file + Command-line overrides         |
| Security        | Read-only by default, fine-grained tool access control |

---

## 🚀 Getting Started

### Prerequisites

- macOS (Silicon / ARM64)
- Python 3.10+
- Node.js (optional, for testing tools like MCP Inspector)
- MCP-compatible client (e.g., [Claude Desktop](https://claude.ai/desktop))

### Installation

```bash
# Clone the repository
git clone https://github.com/Harshil7875/Smart-Workspace-Manager-MCP-Server.git
cd Smart-Workspace-Manager-MCP-Server

# Create a virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Configuration

Create a `.env` file in the project root:

```dotenv
WORKSPACE_PATH=/Users/yourname/Documents/Workspace
ALLOWED_FILE_TYPES=.txt,.md,.pdf
MAX_FILE_SIZE_MB=5
ENABLE_DESTRUCTIVE_TOOLS=false
```

### Running the Server

```bash
python server.py
```

Or launch automatically via your MCP client (e.g., Claude Desktop config).

---

## 🛤️ Roadmap

- [x] Basic MCP server boilerplate
- [x] Expose workspace as Resources
- [ ] Implement search and summarize Tools
- [ ] Add create/move/delete file Tools
- [ ] Support subscription to file changes
- [ ] Integrate Sampling (LLM drafting assistant)
- [ ] Optional remote (HTTP/SSE) support
- [ ] Polished CLI tool packaging

---

## 🧩 Extending

Smart Workspace Manager is built for extensibility:
- Add new tools by defining simple JSON Schema interfaces.
- Customize the workspace scanning logic.
- Extend to support semantic search or AI-driven file tagging.

---

## 📜 License

[MIT License](LICENSE)

---

## 🤝 Contributing

Contributions are welcome!

- Fork the repository
- Create a new branch (`git checkout -b feature-name`)
- Commit your changes
- Open a pull request

Please make sure to update tests and documentation if you add new features.

---

## 🧠 About Model Context Protocol

MCP is a new open standard for connecting LLMs to external tools, databases, and data sources securely and modularly.  
Learn more at [modelcontextprotocol.io](https://modelcontextprotocol.io/).

---