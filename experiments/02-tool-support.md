# Experiment 02 — Tool Support

Odysseus initially exposed `supports_tools: null`. The setting was explicitly changed to `supports_tools: true` after the selected model was confirmed to support tools.

## Result
PASS — configuration enabled tool-capable operation.

## Caveat
A configuration flag does not prove successful execution. Later experiments distinguish configuration from actual tool invocation and multi-step continuation.
