# LiveMCPServer Exploration

## Class: LiveMCPServer

### Instance Methods
None defined directly on the class.

### Class Methods (Metaclass)
1. `addToolFrom:` - likely adds a tool from some specification
2. `endpoint` - probably returns the MCP endpoint
3. `initialize` - class initialization
4. `initializeResources` - initializes resources
5. `initializeTools` - initializes the tool set
6. `initializeToolsFrom:` - initializes tools from a source

## Source Code: initializeTools

```smalltalk
initializeTools
	"tools := superclass tools copy."
	tools := OrderedCollection new.
	
	tools addAll: {
		[LiveApplication activeInstance view]
			aiToolNamed: #applicationView_functions_focusView
			withDescription: 'Show and focus the named view.'
			forSelector: #focusView:.
		
		LiveScene
			aiToolNamed: #scene_properties_set_name
			withDescription: 'Set the name of a scene.'
			forSelector: #setNameOfScene:to:.
		
		[LiveSong activeInstance]
			aiToolNamed: #song_functions_createScene
			withDescription: 'Create a new scene.'
			forSelector: #createSceneAt:}.
	
	[LiveSong activeInstance observeScenes]
```

## Key Insights
1. Tools are stored in an OrderedCollection
2. Each tool is created with:
   - `aiToolNamed:` - the MCP tool name
   - `withDescription:` - the tool description
   - `forSelector:` - the Smalltalk selector to invoke
3. Tools can be created from:
   - A block that returns an object (e.g., `[LiveApplication activeInstance view]`)
   - A class directly (e.g., `LiveScene`)
4. The three current tools map to:
   - `applicationView_functions_focusView` → `LiveApplication activeInstance view focusView:`
   - `scene_properties_set_name` → `LiveScene setNameOfScene:to:`
   - `song_functions_createScene` → `LiveSong activeInstance createSceneAt:`


## Source Code: createToolNamed:describedBy:withReceiverComputation:andSelector:

```smalltalk
createToolNamed: name describedBy: description withReceiverComputation: closure andSelector: selector
	"Create an MCP tool with the given attributes."
	
	<param: #name type: #string description: 'the name of a new MCP tool'>
	<param: #description type: #string description: 'the description of a new MCP tool'>
	<param: #closure type: #string description: 'Smalltalk source code which evaluates to a receiver which performs a new MCP tool''s Smalltalk method'>
	<param: #selector type: #string description: 'the selector of the Smalltalk method which performs the behavior of a new MCP tool'>
	
	tools add: (
		[Compiler evaluate: closure]
			aiToolNamed: name
			withDescription: description
			forSelector: selector asSymbol).
	
	(Tether serverAt: '/mcp/ableton') notifyOfToolsListChange.
	^'Tool created.'
```

## Key Insights from createToolNamed:...
1. This is a method for dynamically creating tools at runtime!
2. It takes:
   - `name` - the MCP tool name (string)
   - `description` - the tool description (string)
   - `closure` - Smalltalk source code that evaluates to the receiver object (string)
   - `selector` - the Smalltalk selector to invoke (string, converted to symbol)
3. It compiles and evaluates the closure to get the receiver
4. It notifies the MCP server at '/mcp/ableton' of the tools list change
5. This is exactly what I need to livecode new tools!


## Complete Workflow for Creating New MCP Tools

1. **Define the implementation method** on the receiver class with parameter pragmas:
   ```smalltalk
   methodName: param1 with: param2
       <param: #param1 type: #string description: 'description of param1'>
       <param: #param2 type: #integer description: 'description of param2'>
       "implementation"
   ```

2. **Register the tool** using createToolNamed:describedBy:withReceiverComputation:andSelector:
   ```smalltalk
   LiveMCPServer 
       createToolNamed: 'tool_name'
       describedBy: 'Tool description.'
       withReceiverComputation: 'ReceiverClass' or '[ReceiverClass instance]'
       andSelector: 'methodName:with:'
   ```

3. **Reinitialize the server**:
   ```smalltalk
   LiveMCPServer initialize
   ```

The pragmas are used by the MCP framework to automatically generate JSON schemas for the tools/list response.

## Next: Examine existing tool methods
Let me look at the source of an existing tool method to see a complete example.
