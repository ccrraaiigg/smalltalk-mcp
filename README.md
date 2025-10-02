# smalltalk-mcp
an MCP server for livecoding a remote Smalltalk

## Overview

This project enables AI agents to interact with a live Smalltalk
object memory running in SqueakJS/Caffeine through the Model Context
Protocol (MCP).

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

1. **Agent sends messages** - The AI agent uses the `send` tool to send Smalltalk messages to objects in the live image
2. **Tether encoding** - Messages are encoded using the Tether binary protocol (see `tether-encoding.md`)
3. **Live execution** - The Smalltalk image executes the message and returns results
4. **Semantic accumulation** - The agent builds understanding of the object memory through successive interactions

## Key Insight

Agents interact with Smalltalk objects using only their identity
hashes (32-bit unsigned integers). The agent accumulates semantic
knowledge about these objects in its context window through
exploration - sending messages, observing results, and building a
mental model of the live system.

## Files

- `bridge.js` - Deno server that bridges MCP and Tether protocols
- `tether.js` - JavaScript implementation of the Tether protocol
- `tether-encoding.md` - Reference guide for encoding/decoding Tether messages

## Example

To evaluate `3 + 4`:

```javascript
send(
  receiver: "40000003",      // SmallInteger 3
  selector: "+",
  arguments: "200000080000000140000004"  // Array containing SmallInteger 4
)
// Returns: {"result": "40000007"}  // SmallInteger 7
```

The agent constructs Tether-encoded hex strings, sends them to the
live Smalltalk image, and interprets the encoded results.
