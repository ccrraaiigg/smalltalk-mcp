# Feature Request: Provide Model Identity Information to AI Assistant

## Problem

The AI assistant currently has no way to know which model it is running as. When users ask "which model are you?", the assistant can only guess based on:
- What the user tells them the UI shows
- Information from their training data (which may be outdated)
- External documentation (which doesn't tell them their own identity)

This leads to:
- Confusing and contradictory responses
- Loss of user trust
- Inability to provide accurate information about capabilities and limitations
- Awkward conversations where the assistant has to admit it doesn't know its own identity

## Proposed Solution

Include model identity information in the system context provided to the AI assistant, similar to how system information is currently provided:

```xml
<model_information>
Model Name: Claude Sonnet 4.5
Model ID: claude-sonnet-4-5-20250929
Model Family: Claude 4
Provider: Anthropic
</model_information>
```

This would allow the assistant to:
- Accurately answer questions about which model it is
- Reference appropriate documentation for its specific version
- Understand its own capabilities and limitations
- Provide consistent, trustworthy responses

## Benefits

- **User trust**: Users can get accurate information about which model they're using
- **Better assistance**: The assistant can tailor responses based on its actual capabilities
- **Reduced confusion**: Eliminates contradictory responses about model identity
- **Transparency**: Aligns with best practices for AI transparency

## Implementation Notes

- Should be included in the system context automatically
- Should update when users switch models
- Could include additional metadata like context window size, capabilities, etc.
