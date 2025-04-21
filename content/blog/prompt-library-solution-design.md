+++
 title = "Prompt Library MCP Server Solution Design"
 description = "Solution design for the Cozy Coder prompt library MCP server."
 date = 2025-04-21T18:14:00+00:00
 updated = 2025-04-21T18:14:00+00:00
 draft = false
 template = "blog/page.html"

 [taxonomies]
 authors = ["jehrhardt"]

 [extra]
 lead = "The goal of Cozy Coder is to let developers define their own prompt libraries for more efficient development with LLMs. To make this possible we need an integration of the prompts into a variety of AI coding agents, which are available in different IDEs and editors."
 +++

## Context

Model Context Protocol is a hot topic right now, as it allows coding agents to integrate additional functionality into the context of an LLM. MCP defines three types of capabilities:

- Resources
- Prompts
- Tools

Most MCP servers provide tools to allow LLMs the usage of external systems.

### MCP Support in IDEs

- Github Copilot (VS Code only)
- Jetbrains AI (currently EAP)
- Claude desktop
- Claude code
- Cursor
- Windsurf
- Zed (currently Preview)

In most cases, MCP servers are set up in a JSON config and many use the stdio based protocol.

Some IDEs and editors like VS Code and Zed offer the possibility to add MCPs via extensions.

Github Copilot and Cursor currently support tools only.

### MCP Frameworks

MCP frameworks are available for different programming languages:

- Python
- Typescript
- Rust

### Managing Prompts

The MCP allows LLMs to fetch and use prompts, but it can also provide tools to let an LLM create or modify external data. That way an agent could be used to create prompts that can later be used to utilize LLMs more efficiently.

### Future Capabilities

In the future, Cozy Coder will provide the possibility to share prompts within a team. To enable this Automerge CRDTs will be used for synchronization between different devices and team members.

## Goals / Non-goals

### Goals

- Create an MCP server that provides custom prompts to coding agents
- Store the prompts in an Automerge document for future synchronization and collaboration
- Provide tools via MCP to create and modify prompts

### Non-goals
- Cloud synchronization
- Team collaboration
- Extensions for specific IDEs or editors
- Installation via package managers

## Solution Design

The Cozy Coder application provides a simple CLI with the following commands:

- `cozycoder` - default command that show the help
- `cozycoder serve` - starts the MCP server on stdio
- `cozycoder prompt` - access to prompts via CLI (e.g. list or delete prompts)

### MCP Server API

The MCP server provides the following capabilities:

- `prompt/list` - lists all prompts stored in the server
- `prompt/get` - returns the named prompt
- `save-prompt` - tool to save a new or existing prompt
- `delete-prompt` - tool to delete existing prompt
