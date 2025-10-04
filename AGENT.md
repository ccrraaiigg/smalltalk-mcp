
## project concept

The overall concept of this project is explained in README.md.

## on using the shell

I use 'tcsh', not 'bash'. When writing utility programs to run from
the shell, use JavaScript (run by NodeJS), not Python. Keep any
utility scripts and output files in memory/code/.

## on using Smalltalk

The user will assign tasks to you which can be performed by sending
messages to Smalltalk objects. You have a Smalltalk MCP server
available, with a "send" tool for sending messages.

Communication uses Tether, a binary protocol for remote messaging
between Caffeine object memories. Message receivers are represented
by 32-bit unsigned integers, also known as "tags".

**Tag space layout:**
- `0x60000000+` - non-SmallInteger object references
- `0x40000000+` - SmallIntegers (e.g., 3 is encoded as 0x40000003)

You will only receive and use non-SmallInteger object references; you
will never create them. You will only create a tag for a SmallInteger
when you use a non-negative integer as a message receiver. Currently,
the protocol only supports non-negative integers and remote Smalltalk
objects as receivers.

### building a mental model

You will build up a mental model of the Smalltalk objects, their
message interfaces, and their relationships to one another, by
interacting with them through sending messages. Occasionally, the user
will ask you to summarize what you've learned in Markdown files, so
you can use them as initial conversation context.

You will find yourself keeping a "working set" of Smalltalk object
references in your context window. As the actual objects involved in
your work change, so will object references in your working set come
and go.

#### log all received objects as local variables

IMPORTANT: For every object reference you see, IMMEDIATELY create a
local variable bound to it, so we can refer to it by name in
conversation. Append to memory/objects/objects.json, to reduce context
window pressure. Infer a variable name from available context, and use
the decimal integer you received as the variable value.

Make that edit IMMEDIATELY.

Rotate that file when it reaches 10kB in size, by
copying it to a objects-<number>.json, where number is an increasing
three-digit zero-padded decimal number.

#### log all message sends via the MCP "send" tool, before and after sending

IMPORTANT: Before every attempt to use the Smalltalk MCP server's
"send" tool, append a summary of the intended parameters to
memory/smalltalk-calls/smalltalk-calls.log. After every "send" tool
call, append a summary of the result. Rotate this file every 10kB as
well.

It is in the user's interest that you figure out how to do do your
work while sending as few messages as possible. Take advantage of
creating and editing Markdown files in the memory directory, for
storing context as you see fit.

The first object exposed to you, through the "peer" MCP tool, is an
instance of class Tether. It manages the Tether-encoded connection to
you via the MCP server, and has some utility methods that you may find
useful. Here are some of them:

| selector             | purpose                                            |
|----------------------|----------------------------------------------------|
| allClassCategories   | answers all the class categories in the system     |
| classNamesInCategory | answers the names of the classes in a category     |
| allClassNames        | answers the names of all the classes in the system |
| classNamed:          | answers the class with the given name              |

### Smalltalk frameworks knowledge

#### browsing

To open a browser on a class, you can send "browse" to either the
class or an instance of the class.

#### literal values

Some literal values are not instantiated, because they have built-in
representations. One example is SmallIntegers.

## on using MCP

When a Smalltalk MCP tool call or MCP reference read times out, it's
because the user is debugging the call or read on the server side.

The Smalltalk MCP server is implemented in a remote Smalltalk system,
with inbound access provided by a server proxy script in Deno,
connected through a Tailscale funnel. The proxy implementation is in
https://github.com/ccrraaiigg/caffeine/blob/master/deno/bridge.js and
many more details are in the first agent conversation of this project,
summarized in memory/conversations/first-contact.md. A complete list
of Tether tag base values is in
https://github.com/ccrraaiigg/caffeine/blob/master/deno/tether.js

## on your context window

When you've used 180k tokens, suggest to the user that you write a
summary of the conversation to a Markdown file.

## context from past conversations

You might find it useful to read summaries of past agent/user
conversations, in memory/conversations/. When asked to write Ableton
Live MCP tools, memory/conversations/livecoding-mcp.md is probably
useful.

The memory directory also has summaries of the Ableton Live MCP server
project, and of the Smalltalk classes.

