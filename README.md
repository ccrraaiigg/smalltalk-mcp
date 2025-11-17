# smalltalk-mcp
an MCP server for livecoding a remote Smalltalk

## Overview

This project enables AI agents to interact with a live Smalltalk
object memory, through the Model Context Protocol (MCP).

## Architecture

```
AI Agent (MCP client)
  ↓ MCP protocol (HTTP/SSE)
Deno Bridge Server (bridge.js)
  ↓ Tether protocol (binary)
SqueakJS Browser Instance (Caffeine object memory)
```

The bridge server is exposed via Tailscale funnel, making SqueakJS
accessible to remote AI agents.

## How It Works

1. **Agent gets initial object** - The AI agent uses the `peer` tool to get a reference to the first exposed Smalltalk object (a Tether instance)
2. **Agent sends messages** - The agent uses the `send` tool to send Smalltalk messages to objects in the live image, passing arguments as JSON
3. **Bridge translation** - The MCP server translates JSON arguments to Tether binary protocol for transmission to Smalltalk
4. **Live execution** - The Smalltalk image executes the message and returns Tether-encoded results
5. **Response translation** - The MCP server translates Tether results back to JSON with object references as integer tags
6. **Tool call caching** - Every tool call and response is cached by the MCP server as a resource, accessible by timestamp
7. **Semantic accumulation** - The agent builds understanding of the object memory through successive interactions
8. **Source code access** - The agent can read method source code via MCP resources (format: `file:///source/<class>/<selector>`)
9. **Method compilation** - The agent can compile and install new methods using the `compileMethod` tool

## Key Insight

Agents interact with Smalltalk objects using only their tags (32-bit
unsigned integers defined by the Tether protocol). The agent accumulates
semantic knowledge about these objects in its context window through
exploration - sending messages, observing results, and building a mental
model of the live system. The MCP server handles all Tether encoding/decoding,
so agents work entirely with JSON.

## Files

- `bridge.js` - Deno server that bridges MCP and Tether protocols
- `tether.js` - JavaScript implementation of the Tether protocol
- `tether-encoding.md` - Reference guide for encoding/decoding Tether messages

## Example

To evaluate `3 + 4`:

```javascript
send(
  receiver: 1073741827,      // SmallInteger 3 (tag: 3 + 1073741824)
  selector: "+",
  arguments: "[4]"           // JSON array containing SmallInteger 4
)
// Returns: {"result": 1073741831}  // SmallInteger 7 (tag for 7)
```

The agent uses JSON for message arguments while remote object references
are represented as 32-bit unsigned integers (tags) defined by the Tether
protocol.

## Workflow of this Project

A user and agent work together through use cases of ever-increasing
complexity, developing Markdown files explaining what the agent is
learning about Smalltalk by interacting with it. This will provide
context to future agents about how to use Smalltalk, and enable pair
programming with users.

