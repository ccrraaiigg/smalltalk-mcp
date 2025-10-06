# Ableton MCP Project Summary

## Overview
The ableton-mcp project is developing MCP tools for Ableton Live 12.3,
providing complete coverage of the Live Object Model (LOM) API. The
project uses the same Tether/Caffeine technology as the Smalltalk MCP
server.

## Current State
- **Status**: Full tool schema coverage complete (as of Oct 1, 2025)
- **Phase**: Now implementing tool functions through use-case discussions with agents
- **Architecture**: MCP server webapp that communicates with Max for Live via its built-in NodeJS server

## Key Components

### Tool Schemas
- Located in `tools/` directory as JSON files
- Each file corresponds to a class in the Ableton Live Object Model
- Schemas define:
  - Children getters: `<class>_children_get_<childName>`
  - Property getters: `<class>_properties_get_<propertyName>`
  - Property setters: `<class>_properties_set_<propertyName>`
  - Functions: `<class>_functions_<functionName>`
  - Summary function for each class

### Classes Covered (partial list from tasks.md)
- Application, ApplicationView
- Chain, ChainMixerDevice
- Clip, Clip.View, ClipSlot
- CompressorDevice, ControlSurface
- And many more...

## Implementation Approach

### Canonical Paths
Objects can be referenced by canonical paths instead of ID lookups:
- First scene: `live_set scenes 0`
- Second scene: `live_set scenes 1`
- First track: `live_set tracks 0`

### Tool Organization
Tools are ordered within each class:
1. Children getters
2. Properties getters, then setters
3. Functions

### Naming Conventions
- Underscores only for namespace delineation
- camelCase for child/property/function names
- Tool descriptions: complete sentences with capitalization and period
- Child/property/return descriptions: phrases without capitalization or period

## Development Workflow
1. Agent checks which MCP tools are actually available
2. If tool is missing, agent suggests implementation referencing JSON schema
3. User implements the tool function
4. Testing is required before declaring success

## Technology Stack
- JavaScript (vanilla, no TypeScript)
- No bundling (no webpack)
- Variables declared at function start
- Single-line if statements without braces

## Currently Implemented Tools (from connected Ableton MCP server)

1. **mcp_ableton_applicationView_functions_focusView**
   - Show and focus a named view
   - Parameter: viewName (string)

2. **mcp_ableton_scene_properties_set_name**
   - Set the name of a scene
   - Parameters: scene (object ID or canonical path), name (string)

3. **mcp_ableton_song_functions_createScene**
   - Create a new scene
   - Parameter: index (integer, -1 to insert at end)

## Goal for This Session
Enable livecoding of MCP tool functions directly in Smalltalk,
augmenting file-based programming capabilities. This would allow rapid
iteration on tool implementations without file editing and server
restarts.
