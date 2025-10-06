# LiveMCPServer Exploration

## Complete Workflow for Creating New MCP Tools

1. **Define the implementation method** on the receiver class with parameter pragmas:
   ```smalltalk
   methodName: param1 with: param2
       <param: #param1 type: #param1Type description: 'description of param1'>
       <param: #param2 type: #param2Type description: 'description of param2'>
	   <result: #resultName type: #resultType description: 'description of method result'>
       "implementation"
   ```

The pragmas are used by the MCP framework to automatically generate
JSON schemas for the tools/list response.

It's okay for your method to have no explicit return value (i.e.,
return self), and have no "result" pragma.

2. **Register the tool** using the "createTool" MCP tool

3. **Reinitialize the server**:
   ```smalltalk
   LiveMCPServer initialize
   ```
