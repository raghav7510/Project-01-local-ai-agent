# Project 01 — Local AI Agent & Tool Automation

An experimental local AI-agent project using **Qwen3.5:4B**, **Ollama**, and **Odysseus**. The objective was to test whether a small locally hosted LLM could reason about tasks and reliably use real tools to interact with a computer and external services.

## Project question

> Can a small local LLM reliably reason about a task, produce executable tool calls, and complete multi-step workflows through an agent runtime?

## Stack

- Model: Qwen3.5:4B
- Inference: Ollama
- Agent interface: Odysseus
- Targets tested: Shell, Browser, Gmail
- Local endpoints: Odysseus `localhost:7000`; Ollama OpenAI-compatible endpoint `localhost:11434/v1`

## Architecture

```text
User instruction
      ↓
Odysseus Agent
      ↓
Qwen3.5:4B
      ↓
Structured tool call
      ↓
Tool execution
      ↓
Tool result
      ↓
Qwen continues reasoning
      ↓
Final response
```

## Experiments

### Local model
Qwen3.5:4B was installed and inspected locally with Ollama. The model was confirmed as a tool-capable local model.

### Tool support
Odysseus initially reported `supports_tools: null`. It was explicitly changed to `supports_tools: true` after confirming model tool support.

### Browser
The agent was asked to open Google, search for `RTX 4050 laptop GPU`, and return the first three results. The intent was understood, but reliable multi-step browser control was not achieved.

### Gmail
Gmail was configured through IMAP, with SMTP available for sending. The validation task was deliberately read-only: identify the latest email subject and sender without sending, deleting, or modifying anything. Integration succeeded; autonomous completion did not.

### Shell isolation test
After removing Gmail from the experiment, the agent was asked to execute `whoami`. The shell tool actually ran and returned `root`. This established that the local model → agent → native tool → execution-result path can work.

### Sequential shell test
A harder test requested `pwd` followed by `whoami` and required both results. The agent repeatedly described the intended actions, but reliable sequential execution did not occur.

## Results

| Experiment | Result |
|---|---|
| Local Qwen inference | PASS |
| Basic reasoning | PASS |
| Tool support configuration | PASS |
| Single shell tool | PASS — `whoami` returned `root` |
| Browser automation | PARTIAL/FAIL |
| Gmail setup | PASS |
| Gmail autonomous workflow | FAIL |
| Sequential shell tools | FAIL |

## Key technical findings

1. Tool intent and tool execution are separate layers.
2. Single-tool success does not prove reliable agentic behavior.
3. Integration is not equivalent to autonomous automation.
4. The experiments did not establish model size as the root cause because the 4B model successfully executed a native shell tool.
5. Controlled isolation was essential: replacing Gmail with `whoami` exposed the orchestration boundary.

## Security

Never commit Gmail app passwords, API keys, browser cookies/sessions, `.env` secrets, or other credentials. Shell access should be treated as privileged access. External integrations should use least privilege.

## Outcome

Project 01 was a successful engineering experiment, but not a fully successful autonomous-assistant build. It established a working local model-to-tool execution path and identified reliability problems in multi-step orchestration.

See [`report/Project_01_Complete_Technical_Report.md`](report/Project_01_Complete_Technical_Report.md) for the full written report and [`experiments/`](experiments/) for experiment records.
