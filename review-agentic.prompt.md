# Agentic AI Review Framework

Domain-specific review checklist for agentic AI codebases. Loaded by the Code Review agent when the project involves autonomous agents, tool-calling systems, or multi-agent orchestration.

## 1. Agentic Guardrails & Safety (Critical)

- Inspect for unbounded autonomous loops (missing max-iteration limits).
- Look for unsafe tool execution (shell, API, or file access without constraints or human-approval gates).
- Check for prompt injection vulnerabilities through retrieved or external content.
- Ensure strict execution sandboxing and absence of hidden side-effects.
- Verify that tool outputs are validated/sanitized before being fed back to the agent.

## 2. Tool Invocation & Resilience

- Assess tool selection logic and prompt instructions for tools.
- Verify action validation (is tool input verified before execution?).
- Inspect retry loops and failure handling (how does the agent recover from a malformed JSON output?).
- Check for idempotency of tool calls where applicable.

## 3. State & Memory

- Review memory handling and potential memory contamination between conversational turns.
- Assess traceability (can you inspect the agent's chain-of-thought and tool outputs?).
- Check for proper cleanup of ephemeral state between sessions.
- Verify that long-term memory stores do not accumulate stale or contradictory entries.
