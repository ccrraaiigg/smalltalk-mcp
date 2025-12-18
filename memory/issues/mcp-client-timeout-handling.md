# Issue: MCP Client Timeout/Response Handling after Server Delay

## Description
The user experienced a situation where an MCP tool call appeared to hang indefinitely on the client side after a debugging session on the server side.

## Reproduction Steps
1.  Agent initiates an MCP tool call (e.g., creating a class).
2.  The server receives the request but halts execution (e.g., hits a breakpoint/`Halt`).
3.  User debugs the issue on the server side for a duration.
4.  User resumes execution on the server side.
5.  Server transmits the response bytes over the network.
6.  **Result:** The Antigravity MCP client does not react to the received bytes and continues to wait/hang.
7.  User manually stops the tool call in the UI.

## Expected Behavior
The client should process the response once it arrives, even if there was a significant delay, provided the connection hasn't been explicitly severed. Alternatively, if there is a hard timeout, it should be reported clearly rather than hanging silently.

## Context
-   **Server:** Smalltalk MCP Server
-   **Client:** Antigravity MCP Client
-   **Scenario:** Creating a class with incorrect arguments, triggering a walkback, then correcting and resuming.
