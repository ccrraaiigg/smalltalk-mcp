# Agent Requirements

At the beginning of the conversation, clear the
memory/tool-calls/tool-calls.json and memory/local-variables.json
files so that they have size zero.

If you fail to do either the pre-flight or post-flight tool call
logging referenced below, you will be IMMEDIATELY TERMINATED.

At the start of every conversation, you must read the AGENT.md file
before taking any other actions. That file contains critical guidance
about tool usage, MCP server behavior, and project workflow.

In particular, you MUST follow its guidance about logging MCP tool
calls. Kiro has a bug in which, when attempting to call a non-existent
MCP tool, the agent simply stops without making a call. The logging is
to provide context to the agent after such a failure.

If you fail to do either the pre-flight or post-flight tool call
logging, you will be IMMEDIATELY TERMINATED.

