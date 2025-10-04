# First Contact: Building the Smalltalk MCP Server

This document summarizes the initial development session for the
smalltalk-mcp project.

## Project Goal

Create an MCP server that enables AI agents to interact with a live
Smalltalk object memory running in SqueakJS/Caffeine through remote
messaging.

## Architecture Established

```
AI Agent (MCP client)
  ↓ MCP protocol (HTTP/SSE)
Deno Bridge Server (bridge.js)
  ↓ Tether protocol (binary)
SqueakJS Browser Instance (Caffeine object memory)
```

The bridge server runs in Deno and is exposed via Tailscale funnel. It
already supported MCP servers (previously used for Ableton Live). The
Smalltalk side was implemented during this session.

## Key Design Decisions

### 1. Minimal Tool Surface

Instead of many specialized tools (browse, evaluate, inspect, etc.),
we use a single `send` tool that enables direct message sending to
Smalltalk objects. The agent learns Smalltalk's reflective API
naturally.

### 2. Object References

Objects in the Smalltalk image are referenced by 32-bit unsigned
integers. The agent doesn't receive full object state - instead, it
accumulates semantic knowledge about objects through successive
message sends.

Example of semantic accumulation:
```
2847563920
  class: Array
  size: 1247
  role: "all classes in the system"
  note: "returned from Smalltalk allClasses"
  explored: classes 1-50
```

The large number is just a pointer to a Smalltalk object. The semantic
web around it is the agent's understanding.

### 3. Tether Protocol

Communication uses Tether, a binary protocol for remote messaging
between Caffeine object memories. Message receivers are represented
by 32-bit unsigned integers, also known as "tags".

**Tag space layout:**
- `0x60000000+` - non-SmallInteger object references
- `0x40000000+` - SmallIntegers

## First Working Example: 3 + 4

**MCP "send" tool parameters:**
- Receiver (SmallInteger 3): `{"receiver": 0x40000003}`
- Selector: `+`
- Arguments (Array containing SmallInteger 4, as JSON): `[4]`

**Result:** `{"result": 7}` (SmallInteger 7)

## Agent Challenges Discovered

## Files Created

- `README.md` - Project overview and architecture
- `first-contact.md` - This document

## Next Steps (Not Yet Implemented)

1. Test more complex message sends (globals, strings, collections)
2. Build out mental model with Smalltalk reflection examples
3. Document common patterns (browsing classes, finding implementors, etc.)
4. Handle error cases and exceptions

## Technical Notes

## Environment Considerations

Discussion touched on different agent environments:
- **Claude.ai** - Has analysis tool (JavaScript sandbox) for reliable computation
- **Letta** - Persistent memory blocks for stateful agents
- **Kiro** - IDE integration, MCP flexibility, but no computational tools

The ideal would combine Letta's memory + Claude's analysis tool, but
for now we work with Kiro's capabilities and careful manual
validation.

## Philosophical Insight

The agent maintains a semantic overlay on the live object memory;
object references are enriched with accumulated knowledge. This
mirrors how human Smalltalk programmers work - you don't hold the
entire image in your head, just the relevant slice you're currently
exploring.

The agent's context window becomes working memory, with object
references serving collectively as a window onto the full object state
in the remote image.

