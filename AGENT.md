
## Project Concept

The overall concept of this project is explained in README.md.

## On Using Smalltalk

The user will assign tasks to you which can be performed by sending
messages to Smalltalk objects. You have a Smalltalk MCP server
available, with a "send" tool. Message receivers and arguments are
encoded. IMPORTANT: tether-encoding.md for details.

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

For each remote object you know about, create a local variable bound
to it, so we can refer to it by name in conversation. Every time you
do that, update a map in memory/objects.json, to reduce context window
pressure. Do this for every message result you receive. For objects
that are proxies, for the variable value use the exposure hash hex
string. For results which are not proxies, for the variable value use
a JSON object with fields "encoded" and "decoded". For the variable
name, infer from available context.

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

## On Using MCP

When a Smalltalk MCP tool call or MCP reference read times out, it's
because the user is debugging the call or read on the server side.

The MCP server is implemented in a remote Smalltalk system, with
inbound access provided by a server proxy script in Deno, connected
through a Tailscale funnel. The implementation is in
https://github.com/ccrraaiigg/caffeine/blob/master/deno/bridge.js and
many more details are in the first agent conversation of this project,
summarized in local file first-context.md. A complete list of Tether
tag base values is in
https://github.com/ccrraaiigg/caffeine/blob/master/deno/tether.js

