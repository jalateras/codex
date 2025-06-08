# Codex Agent Internals

## Table of Contents
- [Introduction](#introduction)
- [Agent Loop (`agent-loop.ts`)](#agent-loop-agent-loopts)
  - [Core Responsibilities](#core-responsibilities)
  - [Initialization](#initialization)
  - [Main Run Cycle (`run` method)](#main-run-cycle-run-method)
  - [Event Handling and Streaming](#event-handling-and-streaming)
  - [Cancellation and Termination](#cancellation-and-termination)
- [Tool Call Processing](#tool-call-processing)
  - [Shell Commands (`shell` and `local_shell` tools)](#shell-commands-shell-and-local_shell-tools)
  - [File Modifications (`apply_patch`)](#file-modifications-apply_patch)
- [Approval Mechanism (`approvals.ts`)](#approval-mechanism-approvalsts)
  - [Approval Policies](#approval-policies)
  - [Safety Assessment (`canAutoApprove`)](#safety-assessment-canautoapprove)
  - [Handling `apply_patch` Approvals](#handling-apply_patch-approvals)
- [Role of `AGENTS.md` Files](#role-of-agentsmd-files)
  - [Providing Context and Instructions](#providing-context-and-instructions)
  - [Loading Mechanism](#loading-mechanism)
- [State Management](#state-management)
  - [Session ID](#session-id)
  - [Response Storage & Transcript](#response-storage--transcript)
  - [Pending Aborts](#pending-aborts)

## Introduction
This document describes the internal workings of the Codex agent, primarily focusing on the logic within `codex-cli/src/utils/agent/`. It details how the agent processes user prompts, interacts with AI models, handles tool calls (like shell commands and file patches), manages user approvals, and utilizes `AGENTS.md` files for contextual instructions.

## Agent Loop (`agent-loop.ts`)
The `AgentLoop` class is the heart of the Codex agent, orchestrating the interaction between the user, the AI model, and the various tools Codex can use.

### Core Responsibilities
-   Managing the conversation flow with the AI model (OpenAI API or compatible providers).
-   Sending user prompts and contextual information (like `AGENTS.md` content and previous interactions) to the model.
-   Receiving and processing responses from the model, which can include text, reasoning steps, and tool calls.
-   Handling the execution of tool calls, including obtaining user approval and applying sandboxing.
-   Streaming updates to the UI.

### Initialization
The `AgentLoop` is initialized with:
-   AI model details (model name, provider).
-   User-provided instructions (from prompts or `AGENTS.md`).
-   The current `ApprovalPolicy`.
-   Configuration settings (`AppConfig`), including API keys and timeout settings.
-   Callbacks for UI updates (`onItem`, `onLoading`), command confirmation (`getCommandConfirmation`), and tracking the last response ID.
-   Information about whether to disable server-side response storage (affecting how conversation history is managed).
-   A unique `sessionId` is generated or retrieved.

It sets up an OpenAI client instance (or AzureOpenAI client) with appropriate headers, timeouts, and proxy configuration if specified.

### Main Run Cycle (`run` method)
The `run` method is the entry point for each turn of interaction.
1.  **Input Processing**:
    -   It takes an array of `ResponseInputItem` (user messages, tool outputs from previous steps).
    -   If there are pending tool calls that were aborted in a previous cancellation, it generates dummy "aborted" outputs for them to satisfy the AI model's expectations.
    -   Constructs the `turnInput` to be sent to the AI. If server-side response storage is disabled, the entire local `transcript` is included; otherwise, only the new input items are sent along with `previousResponseId`.
2.  **AI Model Interaction**:
    -   Sends the `turnInput`, model name, instructions (merged from system prefix, model-specific instructions, and user/`AGENTS.md` instructions), and tool definitions to the AI model provider via `this.oai.responses.create()` or `responsesCreateViaChatCompletions()`.
    -   The request is made in streaming mode (`stream: true`).
    -   It includes retry logic for transient network errors, rate limits (with exponential backoff), and server-side errors.
3.  **Response Streaming**:
    -   Asynchronously iterates over events from the AI model's response stream.
    -   `response.output_item.done`: Processes individual items like text messages, reasoning steps, or tool calls. Reasoning items are annotated with duration. Tool calls are added to a `pendingAborts` set.
    -   `response.completed`: When the full response is received, it processes all output items. If the response status is "completed" or "requires_action", it triggers `processEventsWithoutStreaming`. The `lastResponseId` is updated.
4.  **Loop Continuation**:
    -   The `turnInput` for the next iteration is populated by the results of `processEventsWithoutStreaming` (typically tool call outputs).
    -   The loop continues as long as `turnInput` is not empty (i.e., there are tool calls to respond to or further actions indicated by the model).
    -   If response storage is disabled, logic ensures the loop terminates if only the transcript is resent without new actionable items.

### Event Handling and Streaming
-   The `stageItem` method handles incoming items from the AI. It uses a small `setTimeout` delay to ensure smooth UI rendering and allow for cancellation before an item is actually displayed via the `onItem` callback.
-   It avoids duplicating items already processed using `alreadyStagedItemIds`.
-   If response storage is disabled, it maintains a local `transcript` by appending non-system messages.

### Cancellation and Termination
-   **`cancel()`**: Aborts the current stream and any in-progress tool calls (via `execAbortController`). It sets a `canceled` flag to prevent further processing in the current `run` cycle. If no tool calls were pending, `lastResponseId` is cleared to start a fresh turn.
-   **`terminate()`**: A hard stop that aborts all operations and marks the instance as unusable.

## Tool Call Processing

### Shell Commands (`shell` and `local_shell` tools)
-   When the AI model emits a `function_call` with `name: "shell"` (or `local_shell_call` for `codex-*` models), the `handleFunctionCall` (or `handleLocalShellCall`) method is invoked.
-   It parses the arguments (command, workdir, timeout) from the tool call.
-   It calls `handleExecCommand` (from `codex-cli/src/utils/agent/handle-exec-command.ts`, not read in this session but its role is clear from context).
    -   `handleExecCommand` is responsible for:
        -   Determining the appropriate sandboxing strategy based on the OS and configuration (`SandboxType`).
        -   Calling `exec` (from `codex-cli/src/utils/agent/exec.ts`).
        -   `exec.ts` then delegates to platform-specific execution functions: `rawExec`, `execWithSeatbelt` (macOS), or `execWithLandlock` (Linux).
        -   Obtaining user confirmation via `getCommandConfirmation` if the command is not auto-approved by the current `ApprovalPolicy`.
-   The output (stdout, stderr, exit code) from the command execution is then formatted as a JSON string and returned as a `function_call_output` or `local_shell_call_output` in the next turn's input to the AI.

### File Modifications (`apply_patch`)
-   The `apply_patch` tool is a specialized way to edit files, invoked via the `shell` tool.
-   The agent expects a specific format for patch content, as detailed in `codex-cli/src/utils/agent/apply-patch.ts` (e.g., `*** Begin Patch [...] *** End Patch`).
-   `execApplyPatch` function within `exec.ts` handles these:
    -   It parses the patch text.
    -   Uses `process_patch` (from `apply-patch.ts`) which takes functions for reading, writing, and deleting files.
    -   `process_patch` internally uses `text_to_patch` to parse the diff format and `patch_to_commit` to prepare changes, then `apply_commit` to perform file operations.
    -   The result ("Done!" or an error message) is returned as the output of the `apply_patch` shell command.
-   The `applyPatchToolInstructions` string in `agent-loop.ts` provides detailed instructions to the AI model on how to format these `apply_patch` calls.

## Approval Mechanism (`approvals.ts`)
The `approvals.ts` module determines whether a command (shell or `apply_patch`) can be auto-approved or requires user confirmation.

### Approval Policies
-   **`suggest`**: Only "known safe" read-only commands are auto-approved. All file modifications or other commands require user review.
-   **`auto-edit`**: Auto-approves safe read-only commands and `apply_patch` commands that are constrained to writable paths defined by the user/system. Other shell commands require review.
-   **`full-auto`**: All commands are auto-approved but are expected to run in a sandbox (for shell commands) or are checked against writable paths (for patches).

### Safety Assessment (`canAutoApprove`)
-   This is the main function that takes a command, working directory, policy, and writable roots.
-   **For shell commands**:
    -   It checks against a list of "known safe" commands (`isSafeCommand`): `cd`, `ls`, `pwd`, `cat`, `rg`, `grep`, safe `git` subcommands, etc.
    -   If the command is `bash -lc "..."`, it tries to parse the inner command.
        -   If the inner command is an `apply_patch` heredoc, it delegates to `canAutoApproveApplyPatch`.
        -   Otherwise, it parses the shell expression and checks if all segments are safe commands and all operators are in a safe list (`SAFE_SHELL_OPERATORS`).
    -   If not deemed safe by the above, and policy is `full-auto`, it's approved to run in a sandbox. Otherwise, it returns `ask-user`.
-   **For `apply_patch` commands**:
    -   Delegates to `canAutoApproveApplyPatch`.

### Handling `apply_patch` Approvals
-   `canAutoApproveApplyPatch` checks the policy:
    -   `suggest`: Always asks the user.
    -   `auto-edit` and `full-auto`: Approves if `isWritePatchConstrainedToWritablePaths` returns true. This function checks if all files being added, modified, or deleted by the patch fall within the configured `writableRoots`.
    -   If `full-auto` and not constrained to writable paths (which should be rare if paths are absolute), it might still auto-approve to run (though `apply_patch` itself isn't sandboxed like shell commands, the check acts as a guard). Typically, it would fall back to `ask-user` if paths are not writable.

## Role of `AGENTS.md` Files
(Based on `README.md` and `codex-rs/core/src/project_doc.rs` as `codex-cli/AGENTS.md` was not found, but the principle is general)
-   **Providing Context and Instructions**: `AGENTS.md` files allow users to provide persistent custom instructions, context, or guidance to the AI agent. This can tailor the agent's behavior, knowledge, or responses for specific projects or user preferences.
    -   The root `AGENTS.md` mentions specific instructions for the `codex-rs` context (e.g., related to `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR`).
-   **Loading Mechanism**:
    -   The main `README.md` states Codex looks for `AGENTS.md` in `~/.codex/` (global), repo root, and current working directory, merging them top-down.
    -   This merged content is included in the `instructions` sent to the AI model as part of the system prompt (the `prefix` in `agent-loop.ts` combined with `this.instructions`).

## State Management

### Session ID
-   A `sessionId` (a UUID) is generated or retrieved for each `AgentLoop` instance.
-   It's included in the default headers for API requests, allowing OpenAI to potentially group related requests or for logging/tracing purposes.

### Response Storage & Transcript
-   **`disableResponseStorage` flag**: Controls how conversation history is managed.
    -   If `false` (default): The agent relies on the server to store responses, and uses `previous_response_id` to link conversational turns.
    -   If `true`: The agent maintains a local `transcript` of all non-system messages. This entire transcript is sent with each request to provide context, and `previous_response_id` is not used. The `store: false` parameter is sent to the API.
-   The local `transcript` is updated after each successful stream completion if `disableResponseStorage` is true.

### Pending Aborts
-   A `pendingAborts` set tracks `call_id`s of tool calls that were emitted by the model but whose execution might have been cancelled by the user before a tool output could be sent back.
-   If such pending aborts exist at the start of a new `run`, dummy `function_call_output` items with an "aborted" status are prepended to the input to ensure the AI model's state remains consistent.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
