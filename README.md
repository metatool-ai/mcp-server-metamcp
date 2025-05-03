[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/metatool-ai-mcp-server-metamcp-badge.png)](https://mseep.ai/app/metatool-ai-mcp-server-metamcp)

# MetaMCP MCP Server

[https://metamcp.com](https://metamcp.com): The One MCP to manage all your MCPs

MetaMCP MCP Server is a proxy server that joins multiple MCP⁠ servers into one. It fetches tool/prompt/resource configurations from MetaMCP App⁠ and routes tool/prompt/resource requests to the correct underlying server.

[![smithery badge](https://smithery.ai/badge/@metatool-ai/mcp-server-metamcp)](https://smithery.ai/server/@metatool-ai/mcp-server-metamcp)

<a href="https://glama.ai/mcp/servers/0po36lc7i6">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/0po36lc7i6/badge" alt="MetaServer MCP server" />
</a>

MetaMCP App repo: https://github.com/metatool-ai/metatool-app

## Installation

### Installing via Smithery

Sometimes Smithery works (confirmed in Windsurf locally) but sometimes it is unstable because MetaMCP is special that it runs other MCPs on top of it. Please consider using manual installation if it doesn't work instead.

To install MetaMCP MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@metatool-ai/mcp-server-metamcp):

```bash
npx -y @smithery/cli install @metatool-ai/mcp-server-metamcp --client claude
```

### Manual Installation

```bash
export METAMCP_API_KEY=<env>
npx -y @metamcp/mcp-server-metamcp@latest
```

```json
{
  "mcpServers": {
    "MetaMCP": {
      "command": "npx",
      "args": ["-y", "@metamcp/mcp-server-metamcp@latest"],
      "env": {
        "METAMCP_API_KEY": "<your api key>"
      }
    }
  }
}
```

## Usage

### Using as a stdio server (default)

```bash
mcp-server-metamcp --metamcp-api-key <your-api-key>
```

### Using as an SSE server

```bash
mcp-server-metamcp --metamcp-api-key <your-api-key> --transport sse --port 12006
```

With the SSE transport option, the server will start an Express.js web server that listens for SSE connections on the `/sse` endpoint and accepts messages on the `/messages` endpoint.

### Using with Docker

When running the server inside a Docker container and connecting to services on the host machine, use the `--use-docker-host` option to automatically transform localhost URLs:

```bash
mcp-server-metamcp --metamcp-api-key <your-api-key> --transport sse --port 12006 --use-docker-host
```

This will transform any localhost or 127.0.0.1 URLs to `host.docker.internal`, allowing the container to properly connect to services running on the host.

### Command Line Options

```
Options:
  --metamcp-api-key <key>       API key for MetaMCP (can also be set via METAMCP_API_KEY env var)
  --metamcp-api-base-url <url>  Base URL for MetaMCP API (can also be set via METAMCP_API_BASE_URL env var)
  --report                      Fetch all MCPs, initialize clients, and report tools to MetaMCP API
  --transport <type>            Transport type to use (stdio or sse) (default: "stdio")
  --port <port>                 Port to use for SSE transport (default: "12006")
  --require-api-auth            Require API key in SSE URL path
  --use-docker-host             Transform localhost URLs to use host.docker.internal (can also be set via USE_DOCKER_HOST env var)
  -h, --help                    display help for command
```

## Environment Variables

- `METAMCP_API_KEY`: API key for MetaMCP
- `METAMCP_API_BASE_URL`: Base URL for MetaMCP API
- `USE_DOCKER_HOST`: When set to "true", transforms localhost URLs to host.docker.internal for Docker compatibility

## Development

```bash
# Install dependencies
npm install

# Build the application
npm run build

# Watch for changes
npm run watch
```

## Highlights

- Compatible with ANY MCP Client
- Multi-Workspaces layer enables you to switch to another set of MCP configs within one-click.
- GUI dynamic updates of MCP configs.
- Namespace isolation for joined MCPs.

## Architecture Overview

```mermaid
sequenceDiagram
    participant MCPClient as MCP Client (e.g. Claude Desktop)
    participant MetaMCP-mcp-server as MetaMCP MCP Server
    participant MetaMCPApp as MetaMCP App
    participant MCPServers as Installed MCP Servers in Metatool App

    MCPClient ->> MetaMCP-mcp-server: Request list tools
    MetaMCP-mcp-server ->> MetaMCPApp: Get tools configuration & status
    MetaMCPApp ->> MetaMCP-mcp-server: Return tools configuration & status

    loop For each listed MCP Server
        MetaMCP-mcp-server ->> MCPServers: Request list_tools
        MCPServers ->> MetaMCP-mcp-server: Return list of tools
    end

    MetaMCP-mcp-server ->> MetaMCP-mcp-server: Aggregate tool lists
    MetaMCP-mcp-server ->> MCPClient: Return aggregated list of tools

    MCPClient ->> MetaMCP-mcp-server: Call tool
    MetaMCP-mcp-server ->> MCPServers: call_tool to target MCP Server
    MCPServers ->> MetaMCP-mcp-server: Return tool response
    MetaMCP-mcp-server ->> MCPClient: Return tool response
```

## Credits

- Inspirations and some code (refactored in this project) from https://github.com/adamwattis/mcp-proxy-server/