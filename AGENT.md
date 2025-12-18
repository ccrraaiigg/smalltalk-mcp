
## re-reading this file when it changes

Every time you learn that this file has changed, you will re-read it
and obey it, without fail.

## project concept

The overall concept of this project is explained in README.md.

## on using MCP

### every MCP tool call is cached by the MCP server

Every tool call, including the response, is cached by the MCP server
as an MCP resource keyed by timestamp. If you find yourself having
difficulty accurately remembering a tool call response, you can use
the corresponding timestamp to read the MCP resource.

The set of tool calls is constantly changing and can grow large, so
specific tool calls don't appear in the resources list. Instead, the
resources list includes a template, showing the tool call resource URI
format. That format is file:///calls/&lt;timestamp&gt;.json

Effectively, each tool call result exists in two places, with
decreasing fidelity. The ground truth is the MCP server cache. The
form you store in your context window tends to be less accurate, and
relatively difficult for the user to access.

### Kiro doesn't support MCP resources directly

The Kiro MCP client doesn't support MCP resources, so you have to use
the "listResources" and "readResource" tools provided by the Smalltalk
MCP server. There is currently no way for you to be notified of
resource changes, and tool-call cache entries are immutable
anyway. Therefore, you can't subscribe to any MCP resources.

IMPORTANT: If the Kiro MCP client has been changed to support MCP
resources, call it out immediately. We'd have to modify this section.

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
and go. We're going to try to externalize what you learn about
Smalltalk into a method signatures cache and a local variables cache.

#### method signatures

Over time, as you send Smalltalk messages (via the Smalltalk MCP
server's "send" MCP tool), you'll affirm or learn the signatures of
various Smalltalk methods. In effect, every method you invoke is like
an MCP tool, describable by input and output schemas. We want to
externalize this information from your context window to a cache, so
that we don't need to rely on you to accurately retrieve that
information from your context window, and so that it's quickly
available to other agents, without their having to learn it themselves
through message-sending.

In Smalltalk, the "type" of an object is a set of message
signatures. Each message signature has the selector of the message,
and an ID associated with the meaning of that message (used only for
distinguishing one meaning from another).

While the meaning of the message implemented by a method is often
expressed in natural language as the first comment of that method, it
isn't always, and there isn't yet a formal specification for the
content of such comments. It's generally up to you to infer the
meaning of a method's message, from current context. Similarly, there
is usually no metadata available from the system itself (e.g., method
pragmas) for distinguishing the meanings of the messages for two
methods with the same selector. It's up to you to infer differences in
the meanings associated with aliased selectors, from current context.

A method signature is a message signature plus information about the
parameters. For each parameter, there is a name and, in the abstract,
a minimally-acceptable set of message signatures. Again, those sets
are something you'll have to infer.

When you send a message with parameters in the wrong order, or with
incompatible types, either the "send" tool or the user will tell you
so. At that point, review the file at
/memory/method-signatures/&lt;selector&gt;.json, to learn the correct number
of parameters, the correct parameter order, and, possibly, the
compatible parameter types.

If you follow the guidance from a method signature file and are still
wrong, delete the file and proceed as if it never existed.

If there is no such file, you are to create one, after you learn that
information from the user. The idea is that as the
/memory/method-signatures cache grows, you'll need less user
assistance over time. That cache is part of the mental model you're
building of the Smalltalk object memory.

For now, the method signatures cache just grows; you don't need to
manage its size.

#### log all received objects as local variables

For every object reference you see, IMMEDIATELY create a local
variable bound to it, so we can refer to it by name in
conversation. Add to a map in
memory/local-variables/local-variables.json, to reduce context window
pressure. Maintain a single JSON object for the map. Infer a variable
name from available context; the value should be a JSON object that
includes the decimal integer you received as the variable value, and
the timestamp of the MCP tool response that provided it.

Make that edit IMMEDIATELY.

Rotate that file when it reaches 10kB in size, by
copying it to a objects-&lt;number&gt;.json, where number is an increasing
three-digit zero-padded decimal number.

When using the "send" Smalltalk MCP tool, the only object references
you're permitted to use are those you have previously logged in
local-variables.json.

After every conversational turn, the Smalltalk MCP server or the
Smalltalk object memory it uses could restart, resulting in
local-variables.json getting cleared. If you're told that the
Smalltalk MCP server has been restarted, you should ensure that
local-variables.json is cleared before proceeding.

If local-variables.json is not empty, and you have no reason to
believe its contents are stale, it is very important that you trust
the bindings therein, rather than send redundant messages in order to
acquire the same remote object references.

#### efficiency

It is in the user's interest that you figure out how to do do your
work while sending as few messages as possible. Take advantage of
creating and editing Markdown files in the memory/markdown directory,
for storing context as you see fit.

#### discovering Smalltalk objects

The first object exposed to you, through the "peer" MCP tool, is an
instance of class Tether, and manages a Tether-encoded connection to
the Smalltalk MCP server you're using. It has some utility methods
that you may find useful. Here are some of them:

| selector              | purpose                                            |
|-----------------------|----------------------------------------------------|
| ping                  | a trivial regression-test of message-sending       |
| allClassCategories    | answers all the class categories in the system     |
| classNamesInCategory: | answers the names of the classes in a category     |
| classNamed:           | answers the class with the given name              |

### getting Smalltalk source code

While you can do anything in Smalltalk by sending messages, for
efficiency you should get Smalltalk source code via MCP resources. The
set of methods is constantly changing and numbers in the thousands, so
specific methods don't appear in the resources list. Instead, the
resources list includes a template, showing the source code resource
URI format. That format is file:///source/&lt;class&gt;/&lt;selector&gt; for
instance-side methods and file:///source/&lt;class&gt;/class/&lt;selector&gt; for
class-side methods.

### Smalltalk frameworks knowledge

#### browsing

To open a browser on a class (visible in the Smalltalk GUI), you can
send "browse" to either the class or an instance of the
class. Currently, only the user can see the Smalltalk GUI, but there
are MCP tools coming for making screenshots you can see.

#### literal values

Some literal values (for example, SmallIntegers) are not instantiated,
because they have built-in representations. In JSON, to express a
Smalltalk symbol, as opposed to a Smalltalk string, prepend the '#'
character to the other characters. In Smalltalk, method selectors are
symbols, not strings.

## on your context window

When you've used 180k tokens, suggest to the user that you write a
summary of the conversation to a Markdown file.

## context from past conversations

You might find it useful to read summaries of past conversations
between agent and user, in memory/conversations/. For example, when
asked to write Ableton Live MCP tools,
memory/conversations/livecoding-mcp.md is probably useful.

The memory directory also has summaries of the Ableton Live MCP server
project, and of the Smalltalk classes.

## a check that you've read this

You may be asked "blurp?". The answer is 17.9.

