# First Contact: Building the Smalltalk MCP Server

This document summarizes the initial development session for the smalltalk-mcp project.

## Project Goal

Create an MCP server that enables AI agents to interact with a live Smalltalk object memory running in SqueakJS/Caffeine through remote messaging.

## Architecture Established

```
AI Agent (MCP client)
  ↓ MCP protocol (HTTP/SSE)
Deno Bridge Server (bridge.js)
  ↓ Tether protocol (binary)
SqueakJS Browser Instance (Caffeine object memory)
```

The bridge server runs in Deno and is exposed via Tailscale funnel. It already supported MCP servers (previously used for Ableton Live). The Smalltalk side was implemented during this session.

## Key Design Decisions

### 1. Minimal Tool Surface

Instead of many specialized tools (browse, evaluate, inspect, etc.), we use a single `send` tool that enables direct message sending to Smalltalk objects. The agent learns Smalltalk's reflective API naturally.

### 2. Object Proxies as Identity Hashes

Objects in the Smalltalk image are referenced by their identity hash (32-bit unsigned integer). The agent doesn't receive full object state - instead, it accumulates semantic knowledge about objects through successive message sends.

Example of semantic accumulation:
```
2847563920
  class: Array
  size: 1247
  role: "all classes in the system"
  note: "returned from Smalltalk allClasses"
  explored: classes 1-50
```

The hash is just a pointer. The semantic web around it is the agent's understanding.

### 3. Tether Protocol

Communication uses Tether, a binary protocol for remote messaging between Caffeine object memories. Objects are encoded as 32-bit words (big-endian) in hex.

**Tag space layout:**
- `0x70000000+` - Instructions
- `0x60000001+` - Object proxies (remote references)
- `0x40000000+` - SmallIntegers
- `0x20000000+` - Class tags and special values

**Key insight:** All values are represented in hex to avoid decimal-to-hex conversion errors. Each 32-bit word must be exactly 8 hex characters.

## First Working Example: 3 + 4

**Encoding:**
- Receiver (SmallInteger 3): `40000003`
- Selector: `+`
- Arguments (Array containing SmallInteger 4):
  - Array tag: `20000008`
  - Length: `00000001`
  - Element: `40000004`
  - Concatenated: `200000080000000140000004`

**Result:** `{"result": "40000007"}` (SmallInteger 7)

## Agent Challenges Discovered

### Hex String Transcription Errors

The agent made repeated errors when manually constructing hex strings, writing `400000004` (9 characters) instead of `40000004` (8 characters). This revealed that:

1. Agents don't have direct computational tools in Kiro (unlike Claude.ai's analysis tool)
2. The conversion/serialization step is error-prone
3. Manual validation is necessary

**Solution:** Created detailed encoding documentation with validation checklists in `tether-encoding.md`.

### The Humor Value

The project has an interesting philosophical dimension: agents are effectively performing coded behavior with no code in the traditional sense. They're doing computation through pure reasoning about encodings - constructing bytecode representations in their context window.

## Files Created

- `send.json` - MCP tool schema for the `send` tool
- `tether-encoding.md` - Complete reference for Tether encoding/decoding
- `README.md` - Project overview and architecture
- `first-contact.md` - This document

## Files Referenced

- `bridge.js` - Existing Deno MCP/Tether bridge server
- `tether.js` - JavaScript implementation of Tether protocol

## Next Steps (Not Yet Implemented)

1. Test more complex message sends (globals, strings, collections)
2. Build out mental model with Smalltalk reflection examples
3. Document common patterns (browsing classes, finding implementors, etc.)
4. Handle error cases and exceptions
5. Add support for negative integers (negativeNumberTag)

## Technical Notes

### Tether Encoding Rules for Agents

1. Every 32-bit word = exactly 8 hex characters
2. Use leading zeros to pad
3. Validate total character count (must be divisible by 8)
4. All responses wrapped in answerTag (`0x2000001D`)

### SmallInteger Encoding

Formula: `value + 0x40000000`

Examples:
- `0` → `40000000`
- `3` → `40000003`
- `7` → `40000007`

### Array Encoding

Format: `[arrayTag][length][element1][element2]...[elementN]`

Example for `[4]`:
```
20000008 00000001 40000004
```

## Environment Considerations

Discussion touched on different agent environments:
- **Claude.ai** - Has analysis tool (JavaScript sandbox) for reliable computation
- **Letta** - Persistent memory blocks for stateful agents
- **Kiro** - IDE integration, MCP flexibility, but no computational tools

The ideal would combine Letta's memory + Claude's analysis tool, but for now we work with Kiro's capabilities and careful manual validation.

## Philosophical Insight

The agent becomes a Tether encoder/decoder through pure reasoning. It maintains a semantic overlay on the live object memory, where identity hashes are enriched with accumulated knowledge. This mirrors how human Smalltalk programmers work - you don't hold the entire image in your head, just the relevant slice you're currently exploring.

The agent's context window becomes working memory, with object proxies serving as lightweight references to the full object state in the remote image.
