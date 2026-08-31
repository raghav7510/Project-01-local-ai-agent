# Project 01 — Complete Technical Report

## 1. Executive summary

Project 01 investigated a local AI-agent stack composed of Qwen3.5:4B, Ollama, and Odysseus. The central question was whether a relatively small local model could move from natural-language reasoning into reliable real-world tool execution.

The experiment produced a mixed result. Local inference and basic reasoning worked. Native single-tool execution was demonstrated with a shell `whoami` command returning `root`. However, browser automation, Gmail task completion, and sequential multi-tool execution were not reliable enough to qualify as autonomous agent behavior.

The most important result was diagnostic: the project isolated the difference between **tool awareness**, **tool invocation**, **tool execution**, and **multi-step orchestration**.

## 2. Objectives

- Run a capable LLM locally.
- Connect the model to an agent runtime.
- Enable structured tool use.
- Test a browser workflow.
- Test a read-only Gmail workflow.
- Isolate tool execution using a native shell command.
- Test sequential tool calls.
- Document failure modes rather than hiding them.

## 3. Technology stack

### Qwen3.5:4B

A compact local model selected for experimentation with reasoning and tool use.

### Ollama

Used as the local model serving/runtime layer.

Example commands:

```powershell
ollama pull qwen3.5:4b
ollama run qwen3.5:4b
ollama show qwen3.5:4b
```

### Odysseus

Used as the agent-facing interface/orchestration layer. The local configuration exposed an endpoint on port 7000, while Ollama exposed an OpenAI-compatible endpoint on port 11434.

## 4. System architecture

```text
                    ┌─────────────────────┐
                    │    User request     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │     Odysseus        │
                    │ Agent/orchestration │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Qwen3.5:4B      │
                    │   local inference   │
                    └──────────┬──────────┘
                               │
                         tool decision
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Structured tool     │
                    │ call / execution    │
                    └──────────┬──────────┘
                               │
                 ┌─────────────┼──────────────┐
                 ▼             ▼              ▼
              Shell         Browser         Gmail
                 │             │              │
                 └─────────────┼──────────────┘
                               ▼
                         Tool result
                               │
                               ▼
                       Agent continuation
```

## 5. Experiment A — Local inference

The model was installed and inspected locally through Ollama. The objective was to establish a stable local baseline before adding external integrations.

**Outcome: PASS.**

This established that the project did not depend on a hosted inference API for the core model.

## 6. Experiment B — Tool capability configuration

Odysseus initially exposed tool support as `supports_tools: null`. The setting was explicitly changed to `supports_tools: true` after confirming that the selected model reports tool capabilities.

**Outcome: PASS.**

Important distinction: enabling a tool-support flag does not guarantee correct execution. It only establishes that the orchestration layer is configured to permit tool use.

## 7. Experiment C — Browser automation

### Task

Open Google, search for `RTX 4050 laptop GPU`, and return the first three results.

### Expected pipeline

1. Open browser.
2. Navigate to Google.
3. Enter query.
4. Submit search.
5. Inspect results.
6. Extract first three results.
7. Return them to the user.

### Result

**PARTIAL/FAIL.**

The model could represent the objective and discuss the intended sequence, but browser state and continuation were not reliable enough for repeatable completion.

### Lesson

Browser automation introduces statefulness and observation requirements that are significantly harder than producing a textual answer.

## 8. Experiment D — Gmail integration

### Task

Check Gmail and return the subject and sender of the latest email. The task was explicitly read-only and prohibited sending, deleting, or modifying messages.

### Integration

Gmail was configured through IMAP, with SMTP available for sending when required.

### Result

**Integration: PASS. Autonomous task completion: FAIL.**

The important observation is that a successfully configured connector does not imply successful autonomous use by the agent.

## 9. Experiment E — Shell isolation

To determine whether Gmail/browser integrations were the source of the problem, they were removed from the active test.

### Task

```text
Use the shell tool to run whoami. Return only the result.
```

### Result

**PASS.** The shell tool executed and returned:

```text
root
```

This was the most useful diagnostic experiment because it proved that the local model and orchestration stack can cross the tool-execution boundary.

## 10. Experiment F — Sequential tool calls

### Task

Execute:

```text
pwd
whoami
```

and return both results.

### Result

**FAIL for reliable multi-step execution.**

The agent repeatedly communicated the intended actions, but the actual sequence did not complete consistently.

### Interpretation

The failure points toward orchestration/continuation/state handling rather than a complete inability to use tools.

## 11. Results matrix

| Test | Result | Interpretation |
|---|---|---|
| Local model | PASS | Local inference works |
| Basic reasoning | PASS | Model responds normally |
| Tool support flag | PASS | Agent configured for tools |
| Single shell call | PASS | Actual tool execution demonstrated |
| Browser workflow | PARTIAL/FAIL | Stateful automation unreliable |
| Gmail configuration | PASS | Connector configured |
| Gmail autonomous workflow | FAIL | Connector did not translate into reliable completion |
| Sequential shell workflow | FAIL | Multi-step continuation unreliable |

## 12. Root-cause analysis

The evidence does not justify claiming that the 4B model is simply "too small". If model incapability were the only explanation, the successful `whoami` execution would be difficult to explain.

The more defensible conclusion is that reliable agents require a larger control loop around the model:

- explicit tool schemas;
- deterministic tool-call parsing;
- execution state;
- tool-result injection;
- continuation after tool results;
- retry/error handling;
- termination criteria;
- state synchronization for browser tasks;
- permission boundaries for external accounts.

## 13. Security analysis

The experiment deliberately avoided destructive actions. Gmail validation was read-only.

Credentials must never be stored in source control. In particular, the repository must not contain:

- Gmail app passwords;
- API keys;
- access tokens;
- cookies or browser sessions;
- `.env` files containing secrets;
- personal authentication material.

Shell tools should be sandboxed or permission-restricted in any production-grade agent.

## 14. Final conclusion

Project 01 succeeded as an engineering investigation, not as a finished autonomous-agent product.

The project demonstrated:

- local model inference;
- model-to-agent connectivity;
- tool capability configuration;
- real native shell execution;
- successful isolation of an orchestration problem.

It also demonstrated that multi-step autonomous execution remains substantially harder than single-turn reasoning or isolated tool calls.

That distinction is the primary technical outcome of the project.
