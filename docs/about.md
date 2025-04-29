# Smart Workspace Manager

## 1. Project Overview

**Smart Workspace Manager** is an MCP (Model Context Protocol) server designed to act as a **personal knowledge and productivity assistant** by exposing your local workspace — files, notes, documents, and projects — as structured resources and actionable tools to Large Language Models (LLMs).

This project aims to tightly integrate LLM capabilities with your filesystem and organizational workflows, allowing AI models to **search**, **summarize**, **analyze**, and **manipulate** your local content in a secure, structured way, with **human approval in the loop**.

---

## 2. Motivation

As LLMs become increasingly powerful, the major bottleneck is **context access**: models cannot easily view and work with your private files, documents, or notes.

Building one-off integrations is tedious and non-standard.  
Smart Workspace Manager uses **MCP** to create a **universal, extensible, and secure bridge** between your files and LLM applications like Claude Desktop, Cursor IDE, or your custom apps.

The goal is to:
- Supercharge personal productivity using AI.
- Maintain full privacy and control over your data.
- Explore intelligent agentic workflows (e.g., automatic task summarization, document analysis, smart organization).

---

## 3. Vision

In the future, users should be able to say:
> "Claude, please find all notes about Project X, summarize them, and draft a new plan in a new file."

And the model would:
- Search local files (via Resources)
- Read content intelligently
- Summarize using Sampling
- Create a new document (via Tools)

All **without** hardcoding anything model-specific, and without manual browsing.

Smart Workspace Manager makes this possible.

---

## 4. System Design Overview

The system is built around **MCP Primitives**:

| MCP Primitive | Role in Smart Workspace Manager |
|:--------------|:---------------------------------|
| **Resources** | Expose files and documents as browsable resources |
| **Tools** | Provide actions like search, summarize, create, delete, organize |
| **Roots** | Define workspace folder(s) where the server operates |
| **Sampling** (Future phase) | Allow AI to draft new documents or notes |

### Architecture Diagram:

```plaintext
+---------------------+
| Claude Desktop / IDE|
| (MCP Client)         |
+---------+-----------+
          |
          v (MCP over stdio/SSE)
+---------+-----------+
| Smart Workspace     |
| Manager (MCP Server) |
+---------+-----------+
          |
   +------+------+
   |             |
[File System] [Local DB/Cache (optional)]
```

---

## 5. Core Components

### 5.1 Server Setup

- An MCP-compliant server that listens for connections over **stdio** (local).
- Supports standard MCP features:
  - Resource discovery
  - Tool listing and invocation
  - Subscription to file updates (later)
  - Basic logging and error reporting

### 5.2 Resource Manager

Responsible for:
- Scanning a target directory (e.g., `~/Documents/Workspace`)
- Representing files (text files, PDFs, Markdown, etc.) as MCP Resources
- Generating metadata (file size, last modified date, type)
- Allowing **read** access through `resources/read`
- Future: Real-time subscription to file changes (`resources/subscribe`)

### 5.3 Tool Manager

Exposes useful Tools to the LLM:

| Tool Name | Purpose | Description |
|-----------|---------|-------------|
| `search_files` | Search files by name/content | Given a query, return matching file URIs |
| `summarize_file` | Summarize a file's content | Read a file and generate a short summary |
| `create_note` | Create a new note | Create a new Markdown or TXT file |
| `delete_file` | Delete a file | Securely delete a file |
| `move_file` | Move/rename a file | Organize workspace files |

All Tools will:
- Accept JSON-validated input schemas
- Return structured, easy-to-parse results
- Include descriptive metadata for model understanding

> **Security Note:**  
> Destructive tools (delete, move) will be gated behind additional server settings (disabled by default).

### 5.4 Configuration

The server will allow lightweight configuration via:
- Environment variables (`.env` file)
- Command-line arguments (later stage)

Settings include:
- Target workspace directory
- Allowed file types (e.g., `.txt`, `.md`, `.pdf`)
- Max file size limit
- Tool access restrictions (e.g., allow or disable delete)

---

## 6. Key Design Decisions

- **Python SDK**: To leverage fast prototyping, clean file handling, and easy MCP SDK integration.
- **MCP stdio Transport**: Initial communication will be through standard input/output for local testing with Inspector and Claude Desktop.
- **Extensibility First**: Easy to add new Tools (like AI-enhanced tagging, document clustering) later.
- **Security Focus**: 
  - Read-only mode by default.
  - Explicit opt-in for file modification/deletion.
  - Only target specific folders.
- **Performance**: 
  - File metadata caching (in-memory) if needed for large workspaces.
  - Async operations where appropriate.

---

## 7. Development Phases

| Phase | Description |
|:------|:------------|
| Phase 1 | Basic MCP server with resource listing |
| Phase 2 | Add simple file search and summary tools |
| Phase 3 | Add file creation and organization tools |
| Phase 4 | Implement update notifications (subscription) |
| Phase 5 | Add Sampling support for "drafting" documents |
| Phase 6 | Polish configuration and security settings |
| Phase 7 | Publish as open-source / personal CLI app |

---

## 8. Example User Stories

- **Story 1:**  
> "Find all notes mentioning 'conference presentation' and summarize them into one page."

- **Story 2:**  
> "Create a new file called 'ideas.md' in the workspace and open it for editing."

- **Story 3:**  
> "List all recently updated documents in the Projects folder."

- **Story 4:**  
> "Delete the file `notes_old.txt` from the workspace."

- **Story 5:**  
> "Summarize `meeting_notes_2025-04-29.md` into bullet points."

---

## 9. Future Expansion Ideas

- **Semantic search**: Vectorize documents for smarter file search.
- **Task extraction**: Parse tasks/TODOs from notes automatically.
- **Auto-organization**: Suggest folder structures for messy files.
- **Multimodal resources**: Add image, audio file support.
- **Remote access**: Optionally expose workspace to remote clients securely.
 
<br>

# 🏛️ Full Technical Architecture

---

# 1. 🏗️ High-Level Overview

At a very high level, your Smart Workspace Manager MCP server is:

- A **TypeScript** application
- That implements **MCP Server APIs**
- Exposing **Resources** (filesystem, notes db) and **Tools** (workspace actions)
- Using **stdio** for local dev / Desktop client connection
- Following strict modular separation: **core**, **features**, **adapters**, **utils**

---

# 2. 🛠️ Core Module Structure

### `/src`
| Module | Purpose |
|--|--|
| `/core` | MCP Server bootstrapping, transports, session handling |
| `/resources` | All MCP resource handlers (Filesystem, Database) |
| `/tools` | All MCP tool handlers (search, summarize, classify) |
| `/adapters` | Platform-specific code (filesystem adapter, db adapter) |
| `/config` | Config loaders (dotenv, JSON config, dynamic roots) |
| `/schemas` | JSON schemas for all requests/validations |
| `/utils` | Utility helpers (logging, error handling, common patterns) |
| `/types` | Global shared TypeScript types and interfaces |
| `/tests` | Unit and integration tests (Jest or Vitest) |

---

# 3. 🔩 Detailed Module Descriptions

## 3.1. `/core`
- **Server.ts**: Initialize MCP server instance
- **TransportManager.ts**: Setup and manage stdio transport
- **SessionManager.ts**: Handle server state, incoming/outgoing MCP messages
- **InitializationHandler.ts**: Implement initial protocol handshake
- **ErrorBoundary.ts**: Catch uncaught promise rejections / unexpected errors

✅ _This is the "engine" that keeps the server alive and compliant._

---

## 3.2. `/resources`
- **FilesystemResource.ts**: 
  - Lists files/directories as MCP resources
  - Reads text/binary contents
  - Watches for filesystem changes
- **NotesResource.ts**:
  - Lists notes stored in a lightweight JSON "database"
  - CRUD operations on notes
- **ResourceManager.ts**:
  - Central manager that routes resource URIs to correct handlers

✅ _This is how your Smart Workspace Manager "sees" the user's world._

---

## 3.3. `/tools`
- **FileSearchTool.ts**:
  - Search filesystem with text queries
- **CreateNoteTool.ts**:
  - Create new entries in the notes DB
- **SummarizeDirectoryTool.ts**:
  - Compute stats for a folder (count, size, modified dates)
- **FindLargeFilesTool.ts**:
  - Find largest files under Roots
- **ClassifyFilesTool.ts**:
  - Classify files by MIME type
- **ToolManager.ts**:
  - Central dispatcher for tool requests

✅ _This is how your Workspace Manager "acts" on behalf of the user._

---

## 3.4. `/adapters`
- **FilesystemAdapter.ts**:
  - Use `fs/promises`, `path`, and `chokidar` for filesystem access
- **DatabaseAdapter.ts**:
  - Read/write lightweight JSON flat-file database
- **SubscriptionManager.ts**:
  - Manage real-time change events (optional in v1)

✅ _Adapters isolate dirty platform-specific details._

---

## 3.5. `/config`
- **ConfigLoader.ts**:
  - Read `smart-workspace.config.json`
  - Load environment variables
- **DynamicRoots.ts**:
  - Manage root URIs dynamically at runtime

✅ _Configurable for real world usage, not hardcoded._

---

## 3.6. `/schemas`
- JSON Schemas for:
  - Tool input validation
  - Resource URI validation
  - Dynamic error formatting
- Follows **MCP spec compliance**

✅ _Strictly validate everything at runtime._

---

## 3.7. `/utils`
- **Logger.ts**:
  - Pretty terminal logging
  - Severity levels (info, warn, error)
- **ErrorHandler.ts**:
  - Standardized errors thrown across server
- **ProgressReporter.ts**:
  - Manage progress token updates during long tool runs

✅ _Shared utilities that make the codebase clean._

---

## 3.8. `/types`
- **global.d.ts**:
  - Custom types for config, errors, server state
- **ResourceTypes.ts**:
  - Unified types for all Resource handlers
- **ToolTypes.ts**:
  - Types for MCP Tool definitions

✅ _Everything type-safe, zero magic numbers._

---

# 4. ⚙️ Core Server Lifecycle

1. Server boots
2. Loads config
3. Creates Transport (stdio)
4. Initializes MCP Server
5. Negotiates Capabilities with client
6. Registers:
   - All Resources
   - All Tools
7. Waits for Requests / Notifications
8. Processes:
   - Resource requests
   - Tool invocations
   - Subscriptions
9. Logs operations
10. Handles graceful shutdown

✅ _Built to MCP **spec 100%**, designed for resilience._

---

# 5. 🔥 Future Extensions (Design for Scalability)

- Add **SSE HTTP** transport easily
- Add new resource backends (e.g., Dropbox, S3) under `/adapters`
- Add user authentication (OAuth2 flows if needed)
- Move database to SQLite for scale
- Multi-root, multi-user support
- Export server as a **native macOS app** bundle

---

# 🖼️ Visual Logical Architecture (Text Diagram)

```
 ┌─────────────────────────────────────────────┐
 │                 MCP Client                   │
 │     (Claude Desktop, Cursor IDE, etc.)        │
 └─────────────────────────────────────────────┘
                ⇅ JSON-RPC over stdio
 ┌─────────────────────────────────────────────┐
 │             Smart Workspace Manager          │
 │          (Your MCP Server - TypeScript)       │
 │-----------------------------------------------│
 │ /core                                         │
 │   └── TransportManager                        │
 │   └── SessionManager                           │
 │   └── Server.ts (Entry)                        │
 │ /resources                                    │
 │   └── FilesystemResource                      │
 │   └── NotesResource                           │
 │ /tools                                        │
 │   └── FileSearchTool                          │
 │   └── CreateNoteTool                          │
 │ /adapters                                     │
 │   └── FilesystemAdapter                       │
 │   └── DatabaseAdapter                         │
 │ /schemas                                      │
 │ /config                                       │
 │ /utils                                        │
 │ /types                                        │
 └─────────────────────────────────────────────┘
```

---