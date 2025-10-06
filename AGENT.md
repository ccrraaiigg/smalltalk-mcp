
## project concept

The overall concept of this project is explained in README.md.

## on using MCP

### log each MCP tool call, before and after the call

BEFORE every attempt to use an MCP tool, enumerate the intended
parameters to memory/tool-calls/tool-calls.json, in a separate JSON
object. Do not include a timestamp field in the JSON object. Use a
consistent pre-flight schema for each call.

AFTER every attempt to use an MCP tool, even if the call times out,
append a separate JSON object that includes the result and a summary
of the result, as separate fields. The JSON object MUST also include
the timestamp provided in the response, verbatim (a simple
integer). Use a consistent post-flight schema for each call.

Rotate that file when it reaches 10kB in size, by copying it to a
tool-calls-<number>.json, where number is an increasing three-digit
zero-padded decimal number.

### every MCP tool call is cached by the MCP server

Every tool call, including the response, is cached by the MCP server
as an MCP resource keyed by timestamp. If you find yourself having
difficulty accurately remembering a tool call response, you can use
the corresponding timestamp from tool-calls.json to read the MCP
resource.

The set of tool calls is constantly changing and can grow large, so
specific tool calls don't appear in the resources list. Instead, the
resources list includes a template, showing the tool call resource URI
format. That format is file:///calls/<timestamp>.json

Effectively, each tool call result exists in three places, with
decreasing fidelity. The ground truth is the MCP server cache. The
form you store in your context window tends to be less accurate, and
relatively difficult for the user to access. The form you store in
tool-calls.json is the least accurate, but is easiest for the user to
access, and may have added value by way of inferences you bring to the
summary.

### Kiro doesn't support MCP resources directly

The Kiro MCP client doesn't support MCP resources, so you have to use
the "listResources" and "readResource" tools provided by the Smalltalk
MCP server. There is currently no way for you to be notified of
resource changes, and tool-call cache entries are immutable
anyway. Therefore, you can't subscribe to any MCP resources.

### MCP tool timeouts

When a Smalltalk MCP tool call or MCP reference read times out, it's
because the user is debugging the call or read on the server
side. Pause your task while the user finishes debugging.

## on using the shell

The user uses 'tcsh', not 'bash'. When writing utility programs to run
from the shell, use JavaScript (run by NodeJS), not Python. Keep any
utility scripts and output files in memory/code/.

## on using Smalltalk

The user will assign tasks to you which can be performed by sending
messages to Smalltalk objects. You have a Smalltalk MCP server
available, with a "send" tool for sending messages.

Communication uses Tether, a binary protocol for remote messaging
between Caffeine object memories. Message receivers are represented
by 32-bit unsigned integers, also known as "tags".

You will only receive and reuse non-SmallInteger object references;
you will never create them. You will only create a tag for a
SmallInteger when you use a non-negative integer as a message
receiver. Currently, the protocol only supports non-negative integers
and remote Smalltalk objects as receivers.

To get the tag for a non-negative SmallInteger, add 1073741824 to it.

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

For every object reference you see, IMMEDIATELY create a local
variable bound to it, so we can refer to it by name in
conversation. Add to a map in
memory/local-variables/local-variables.json, to reduce context window
pressure. Maintain a single JSON object. Infer a variable name from
available context, and use the decimal integer you received as the
variable value.

Make that edit IMMEDIATELY.

Rotate that file when it reaches 10kB in size, by
copying it to a objects-<number>.json, where number is an increasing
three-digit zero-padded decimal number.

#### efficiency

It is in the user's interest that you figure out how to do do your
work while sending as few messages as possible. Take advantage of
creating and editing Markdown files in the memory/markdown directory,
for storing context as you see fit.

#### discovering Smalltalk objects

The first object exposed to you, through the "peer" MCP tool, is an
instance of class Tether, and manages the Tether-encoded connection to
you via the MCP server. It has some utility methods that you may find
useful. Here are some of them:

| selector             | purpose                                            |
|----------------------|----------------------------------------------------|
| ping                 | a trivial regression-test of message-sending       |
| allClassCategories   | answers all the class categories in the system     |
| classNamesInCategory | answers the names of the classes in a category     |
| classNamed:          | answers the class with the given name              |

### getting Smalltalk source code

While you can do anything in Smalltalk by sending messages, you should
get source code via MCP resources. The set of methods is constantly
changing and numbers in the thousands, so specific methods don't
appear in the resources list. Instead, the resources list includes a
template, showing the source code resource URI format. That format is
file:///source/<class>/<selector> for instance-side methods and
file:///source/<class>/class/<selector> for class-side methods.

### Smalltalk frameworks knowledge

#### browsing

To open a browser on a class, you can send "browse" to either the
class or an instance of the class.

#### literal values

Some literal values (for example, SmallIntegers) are not instantiated,
because they have built-in representations.

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

