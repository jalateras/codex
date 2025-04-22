# OpenAI Codex CLI – Project Overview

This repository contains **Codex CLI**, a terminal‑based agentic coding assistant.  
It wraps OpenAI models to let you converse with – and **direct** – a local
codebase using natural‑language instructions. The tool can read & edit files,
run shell commands in a sandbox, and iteratively refine code under your
guidance. In short: a lightweight, scriptable pair‑programming companion that
fits inside your terminal.

The project is intentionally modular. The diagram and sections below give a
bird’s‑eye view of the main building blocks and how they interact.

```text
┌────────────────────────────────────┐
│            Terminal UI             │  ← React + Ink components
└────────────────────────────────────┘
               │
               ▼  user input / streaming output
┌────────────────────────────────────┐
│           CLI Entrypoint           │  src/cli.tsx / cli_singlepass.tsx
└────────────────────────────────────┘
               │ initiates / controls
               ▼
┌────────────────────────────────────┐
│            AgentLoop               │  utils/agent/agent-loop.ts
└────────────────────────────────────┘
      │                 │
      │ tool‑calls      │ streamed text
      ▼                 ▼
┌────────────────┐  ┌────────────────────┐
│ Patch engine   │  │  Shell executor    │
│  (apply‑patch) │  │  (exec + sandbox)  │
└────────────────┘  └────────────────────┘
      │                 │
      └───────┬─────────┘
              ▼
     Local working directory
```

## 1. User Interface (codex‑cli/src/components)

Codex presents a **React‑Ink** TUI composed of several overlays:

- `chat/` – scrolling message history, multiline editor, typeahead etc.
- `help‑overlay.tsx`, `history‑overlay.tsx`, `model‑overlay.tsx` – transient
  panels toggled by hotkeys.
- `approval‑mode‑overlay.tsx` – collects user confirmation when Codex proposes
  edits or commands that are not auto‑approved.

All rendering lives in the terminal; no browser is required.

## 2. CLI Entrypoints

- **Interactive mode** – `src/cli.tsx` bundles the Ink application and starts
  an `AgentLoop` instance.
- **Single‑pass/CI mode** – `src/cli_singlepass.tsx` executes one instruction
  non‑interactively (useful for scripting or pipelines).

The build script (`build.mjs`) uses **esbuild** to transpile & bundle these
entrypoints into `dist/cli.js` which is published to npm and exposed via the
`codex` binary.

## 3. Agent Engine (src/utils/agent)

`AgentLoop` orchestrates the conversation with the OpenAI API:

1. Streams model responses.
2. Detects **tool calls** encoded in the messages.
3. Routes recognised calls to dedicated handlers.
4. Sends the handler’s output back to the model so it can continue the chain.

Supported tools / handlers:

- **Shell execution** – `handle-exec-command.ts` spawns commands via `exec.ts`.
  Execution occurs inside a platform‑specific **sandbox**
  (`sandbox/`):
  - macOS – Seatbelt profiles.
  - Linux – (future) seccomp; currently relies on process isolation.
- **Patch application** – `apply‑patch.ts` validates and applies unified diff
  snippets to the working tree. Safety rails prevent destructive edits unless
  explicitly allowed.

An **approval policy** (defined in `approvals.ts`) governs whether patches or
commands execute automatically, require a quick diff review, or must be
rejected outright.

## 4. Utilities & Helpers

- `parse‑apply‑patch.ts`, `format‑command.ts` – diff parsing and pretty‑printing.
- `text‑buffer.ts` – CRLF‑aware in‑memory text manipulation used by the editor.
- `utils/` – configuration loader, token estimators, session tracking, etc.

## 5. Examples (codex‑cli/examples)

Self‑contained folders demonstrating Codex in action:

- `camerascii` – converts webcam frames to ASCII art.
- `impossible‑pong` – AI‑assisted game implementation.
- `prompt‑analyzer` – clusters & visualises prompt datasets.

Each example ships with a `task.yaml` describing the objective so Codex can
replay or re‑build the artefact from scratch.

## 6. Tests (codex‑cli/tests)

The project leans heavily on **Vitest** for unit & integration tests covering
agent error states, diff logic, multiline editor behaviour, etc. The fixtures
under `__fixtures__/` provide sample files for patching scenarios.

## 7. Build, Release & Tooling

- `npm run build` – esbuild bundle with tree‑shaking & optional dev‑time
  source‑maps.
- Prettier + ESLint enforce style.
- Git hooks (via `pre‑commit.yaml` if present) gate commits.
- `release` script bumps the version to a timestamp (YYMMDDHHmm) and publishes
  to npm.

---

### How the pieces fit together

1. User types a message; the Ink front‑end forwards it to **AgentLoop**.
2. AgentLoop sends the prompt (plus context) to the OpenAI Chat Completions
   endpoint.
3. As the response streams in, UI prints text chunks immediately.
4. When the model emits a `function_call`:
   • If it’s `exec`, AgentLoop asks `handle‑exec‑command` to run it in the
   sandbox.
   • If it’s `applyPatch`, AgentLoop parses the diff, optionally shows it to
   the user, then writes it to disk.
5. The handler’s JSON result is fed back to the model so it can continue the
   chain (think tools loop).
6. The cycle repeats until the model returns a final assistant message.

## 8. Extending Codex

- Add a new tool by implementing a handler in `utils/agent/` and wiring it into
  `AgentLoop`’s `toolHandlers` map.
- New UI overlays plug into `src/components` – all Ink/React code.
- Rules for auto‑approval live in `approvals.ts`.

## 9. Key Files Cheat‑Sheet

| Area               | Path                                               |
| ------------------ | -------------------------------------------------- |
| CLI entrypoint     | `codex-cli/src/cli.tsx`                            |
| Singlepass CLI     | `codex-cli/src/cli_singlepass.tsx`                 |
| Ink UI root        | `codex-cli/src/app.tsx`                            |
| Agent loop         | `codex-cli/src/utils/agent/agent-loop.ts`          |
| Patch application  | `codex-cli/src/utils/agent/apply-patch.ts`         |
| Shell exec handler | `codex-cli/src/utils/agent/handle-exec-command.ts` |
| Sandboxing helpers | `codex-cli/src/utils/agent/sandbox/`               |
| Approval policies  | `codex-cli/src/approvals.ts`                       |
| Build script       | `codex-cli/build.mjs`                              |

---

For detailed usage, configuration options and development workflow, refer to
`README.md`. This document is meant to be the quick mental map of what lives
where and how data flows when you run `codex`.
