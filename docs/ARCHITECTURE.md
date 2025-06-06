# Architecture

## Table of Contents
- [Overview](#overview)
- [High-Level System Architecture](#high-level-system-architecture)
  - [Codex CLI (TypeScript)](#codex-cli-typescript)
  - [Codex RS (Rust)](#codex-rs-rust)
  - [Interaction (Future)](#interaction-future)
- [Major Architectural Patterns](#major-architectural-patterns)
  - [CLI (Command Line Interface)](#cli-command-line-interface)
  - [Agent-Based System](#agent-based-system)
  - [Reactive UI (for CLI)](#reactive-ui-for-cli)
  - [Sandboxing](#sandboxing)
  - [Multi-Codex Protocol (MCP)](#multi-codex-protocol-mcp)
- [System Boundaries and External Dependencies](#system-boundaries-and-external-dependencies)
- [Data Flow](#data-flow)
  - [Interactive Mode](#interactive-mode)
  - [Command Execution](#command-execution)
- [Key Architectural Decisions and Trade-offs](#key-architectural-decisions-and-trade-offs)
  - [TypeScript for Main CLI](#typescript-for-main-cli)
  - [Rust for Core Logic & Performance](#rust-for-core-logic--performance)
  - [Security Model (Approval Modes & Sandboxing)](#security-model-approval-modes--sandboxing)
  - [Configuration System](#configuration-system)
- [Future Directions (Speculative)](#future-directions-speculative)

## Overview
This document describes the architecture of the Codex CLI system. It covers the high-level structure, major components, architectural patterns, and key design decisions.

## High-Level System Architecture
The Codex project is primarily composed of two distinct yet related sub-systems: `codex-cli` (written in TypeScript) and `codex-rs` (written in Rust).

### Codex CLI (TypeScript)
-   **Path**: `codex-cli/`
-   **Description**: This is the main user-facing component and the current stable version of Codex. It provides an interactive command-line interface that allows users to interact with an AI coding assistant.
-   **Key Responsibilities**:
    -   Parsing user input and commands.
    -   Managing interaction with AI models (primarily OpenAI).
    -   Displaying AI responses and suggested actions (e.g., file modifications, shell commands).
    -   Handling user approval for actions.
    -   Applying approved changes to the file system or executing commands.
    -   Terminal UI rendering using React and Ink.
-   **Runtime**: Node.js.

### Codex RS (Rust)
-   **Path**: `codex-rs/`
-   **Description**: This is a newer, ongoing effort to rewrite and enhance parts of the Codex functionality in Rust. The primary motivations are performance, smaller distributable binaries, and more robust sandboxing.
-   **Key Responsibilities (and aspirations)**:
    -   Core AI interaction logic.
    -   Secure command execution and sandboxing (leveraging OS-specific features like Seatbelt on macOS and Landlock/seccomp on Linux).
    -   Headless operation for automation (`codex-rs/exec`).
    -   A potential terminal UI (`codex-rs/tui`).
    -   Implementation of the Multi-Codex Protocol (MCP) for tool integration.
-   **Runtime**: Compiled native executable.

### Interaction (Future)
Currently, `codex-cli` is the primary executable. The `codex-rs` components are largely standalone or experimental. The main `README.md` mentions:
> "Fat" package that additionally bundles the native Rust CLI binaries for Linux. End-users can then opt-in at runtime by setting CODEX_RUST=1.

This suggests a future where `codex-cli` might delegate certain operations (like sandboxed command execution) to the Rust binaries, or `codex-rs` might become the primary executable with `codex-cli`'s UI components potentially being adapted or replaced.

## Major Architectural Patterns

### CLI (Command Line Interface)
Both `codex-cli` and `codex-rs` are fundamentally CLI applications, providing entry points for user interaction and automation through terminal commands.

### Agent-Based System
Codex acts as an "agent" that interprets user requests, interacts with external services (AI models), and performs actions on behalf of the user. The `codex-cli/src/utils/agent/` directory highlights this pattern.

### Reactive UI (for CLI)
`codex-cli` uses [Ink](https://github.com/vadimdemedes/ink) with [React](https://react.dev/) to build its interactive terminal user interface. This allows for a more dynamic and responsive CLI experience compared to traditional print-and-prompt CLIs.

### Sandboxing
A critical architectural concern is the safe execution of AI-suggested commands.
-   **`codex-cli`**:
    -   On macOS, it uses `sandbox-exec` (Apple Seatbelt).
    -   On Linux, it recommends Docker for sandboxing, with scripts to manage a minimal container image and firewall rules.
-   **`codex-rs`**: Aims to implement more direct sandboxing using Linux Landlock/seccomp and macOS Seatbelt, as detailed in `codex-rs/cli/src/landlock.rs` and `codex-rs/cli/src/seatbelt.rs`.

### Multi-Codex Protocol (MCP)
The `codex-rs` component is developing a system referred to as MCP (`mcp-client/`, `mcp-server/`, `mcp-types/`, `docs/protocol_v1.md`). This appears to be a protocol for Codex to communicate with external tools or services, potentially allowing for an extensible ecosystem of capabilities. The configuration in `codex-rs/README.md` shows how MCP servers can be defined.

## System Boundaries and External Dependencies

-   **User**: Interacts with Codex via the terminal.
-   **AI Models**: Primarily OpenAI models (GPT series). Codex is configurable to use other providers compatible with the OpenAI API (e.g., Azure, OpenRouter, Ollama). This is managed via `codex-cli/src/utils/openai-client.ts` and `codex-rs/core/src/client.rs`.
-   **File System**: Codex reads and writes files in the user's project directory.
-   **Shell**: Codex executes shell commands.
-   **Git**: Used for version control; Codex can commit changes.
-   **NPM/PNPM/Yarn/Bun**: For installing `codex-cli`.
-   **Cargo**: For building `codex-rs`.
-   **Operating System**: For sandboxing features and general execution.
-   **External Tools (via MCP)**: Future integration with other developer tools.

## Data Flow

### Interactive Mode (`codex-cli`)
1.  User launches `codex` or `codex "initial prompt"`.
2.  The Ink/React UI (`codex-cli/src/app.tsx`) initializes.
3.  User types a prompt.
4.  The prompt is sent to the AI model provider via `codex-cli/src/utils/agent/agent-loop.ts` and `openai-client.ts`.
5.  The AI model returns a response, which may include natural language, code diffs, or shell commands.
6.  The UI displays the response.
7.  If actions (file changes, commands) are suggested:
    -   Codex presents them to the user for approval (based on `approvalMode`).
    -   `codex-cli/src/approvals.ts` manages this.
8.  If approved:
    -   File patches are applied (e.g., using `codex-cli/src/utils/agent/apply-patch.ts`).
    -   Shell commands are executed (e.g., via `codex-cli/src/utils/agent/exec.ts`), potentially within a sandbox.
9.  Results are displayed, and the loop continues.

### Command Execution (Conceptual, focusing on sandboxing)
1.  AI suggests a shell command.
2.  User approves (or auto-approval is active).
3.  **`codex-cli`**:
    -   On macOS: The command is wrapped with `sandbox-exec`.
    -   On Linux (with Docker sandbox): `codex-cli` communicates with a Docker container where the command is run.
4.  **`codex-rs` (intended)**:
    -   The command is passed to the Rust execution component.
    -   Appropriate sandboxing (Seatbelt, Landlock) is applied directly by Rust code.
    -   Command output/status is returned.

## Key Architectural Decisions and Trade-offs

### TypeScript for Main CLI
-   **Pros**: Rapid development, large ecosystem (Node.js), good tooling, existing libraries for CLI UI (Ink). Suitable for I/O-bound tasks like interacting with APIs.
-   **Cons**: Larger distributable size (Node.js runtime dependency), potentially higher memory footprint and slower startup compared to native binaries, sandboxing is less direct (relies on `sandbox-exec` or Docker).

### Rust for Core Logic & Performance
-   **Pros**: Performance, memory safety without a garbage collector, small native binaries, direct access to OS-level sandboxing APIs, strong type system.
-   **Cons**: Steeper learning curve than TypeScript, potentially slower development for UI-heavy parts, ecosystem for CLI UI not as mature as Ink for Node.js (though `ratatui` used in `codex-rs/tui` is powerful).

### Security Model (Approval Modes & Sandboxing)
-   **Decision**: Provide granular control over AI autonomy (`suggest`, `auto-edit`, `full-auto`) and enforce sandboxing for command execution.
-   **Trade-off**: `Full Auto` mode offers maximum convenience but relies heavily on the robustness of the sandbox. Sandboxing itself can sometimes interfere with legitimate tool operations if not configured correctly, and perfect sandboxing is a hard problem. The different approaches (Seatbelt, Landlock, Docker) have their own complexities and platform limitations.

### Configuration System
-   **Decision**: Allow rich configuration via files (`~/.codex/config.json` for `codex-cli`, `~/.codex/config.toml` for `codex-rs`) and environment variables.
-   **Pros**: Flexibility for users to customize models, providers, behavior.
-   **Cons**: Potential for confusion with two different config file formats and sets of options during the transition or if both systems are used in conjunction. The `codex-rs/README.md` details precedence rules for its configuration.

## Future Directions (Speculative)
-   **Unified Executable**: `codex-rs` could become the primary executable, potentially embedding a JavaScript runtime for parts of the UI or business logic if needed, or fully rewriting all components in Rust.
-   **Enhanced MCP**: The Multi-Codex Protocol could enable a rich plugin architecture, allowing Codex to leverage a wider array of developer tools and services in a structured way.
-   **Deeper IDE Integration**: While currently a CLI tool, the core logic could be adapted for IDE extensions.
-   **Wider OS Support for Robust Sandboxing**: Expanding direct, native sandboxing capabilities to more platforms or refining existing ones.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
