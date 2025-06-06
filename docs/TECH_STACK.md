# Technology Stack

## Table of Contents
- [Overview](#overview)
- [Programming Languages](#programming-languages)
- [Node.js Ecosystem (`codex-cli`)](#nodejs-ecosystem-codex-cli)
  - [Core & Frameworks](#core--frameworks)
  - [CLI & UI](#cli--ui)
  - [API Interaction & Utilities](#api-interaction--utilities)
  - [Development & Build Tools](#development--build-tools)
- [Rust Ecosystem (`codex-rs`)](#rust-ecosystem-codex-rs)
  - [Core & Async](#core--async)
  - [CLI & UI (TUI)](#cli--ui-tui)
  - [Serialization & Parsing](#serialization--parsing)
  - [System & Utilities](#system--utilities)
  - [Sandboxing & Security](#sandboxing--security)
- [External Services & APIs](#external-services--apis)
- [Development Environment & Tools](#development-environment--tools)

## Overview
This document outlines the complete technology stack used in the Codex CLI project, covering programming languages, frameworks, libraries, external services, and development utilities for both the `codex-cli` (TypeScript) and `codex-rs` (Rust) components.

## Programming Languages
-   **TypeScript**:
    -   Version: `^5.0.3` (from `codex-cli/package.json`, specific version likely `5.8.3` from `pnpm-lock.yaml`)
    -   Usage: Primary language for `codex-cli`.
-   **JavaScript (ESM)**:
    -   Usage: `codex-cli` is a Node.js application using ECMAScript Modules.
-   **Rust**:
    -   Edition: `2024` (from `codex-rs/Cargo.toml`)
    -   Usage: Primary language for `codex-rs` components.

## Node.js Ecosystem (`codex-cli`)
*(Versions are primarily from `pnpm-lock.yaml` where available, falling back to `package.json`.)*

### Core & Frameworks
-   **Node.js**: Version `^22.0.0` (runtime requirement from `codex-cli/package.json`)
-   **React**: `^18.2.0` (actual: `18.3.1`) - For building the terminal UI with Ink.
-   **Ink**: `^5.2.0` - React renderer for interactive command-line UIs.
-   **Express**: `^5.1.0` (actual: `5.1.0`) - Web framework, usage might be for internal tools or specific features.

### CLI & UI
-   **`@inkjs/ui`**: `^2.0.0` - Additional UI components for Ink.
-   **Meow**: `^13.2.0` - CLI app helper.
-   **Chalk**: `^5.2.0` (actual: `5.4.1`) - Terminal string styling.
-   **Figures**: `^6.1.0` - Unicode symbols with fallbacks for Windows.
-   **Marked**: `^15.0.7` (actual: `15.0.8`) - Markdown parser.
-   **`marked-terminal`**: `^7.3.0` - Custom Marked renderer for terminal output.
-   **`cli-spinners`**: `3.2.0` - Spinners for asynchronous operations.
-   **`cli-cursor`**: `5.0.0` - Control the CLI cursor.
-   **`cli-truncate`**: `4.0.0` - Truncate text in the terminal.
-   **`cli-boxes`**: `3.0.0` - Boxes in the terminal.

### API Interaction & Utilities
-   **OpenAI**: `^4.95.1` - Official Node.js library for OpenAI API.
-   **Zod**: `^3.24.3` - TypeScript-first schema validation.
-   **`https-proxy-agent`**: `^7.0.6` - HTTP/HTTPS proxy agent.
-   **Dotenv**: `^16.1.4` (actual: `16.5.0`) - Loads environment variables from `.env` files.
-   **Diff**: `^7.0.0` - JavaScript library for text diffing.
-   **`js-yaml`**: `^4.1.0` - YAML parser and stringifier.
-   **`shell-quote`**: `^1.8.2` - Quote and parse shell commands.
-   **`strip-ansi`**: `^7.1.0` - Strip ANSI escape codes.
-   **`file-type`**: `^20.1.0` (actual: `20.4.1`) - Detect the file type of a Buffer/Uint8Array.
-   **`package-manager-detector`**: `^1.2.0` - Detects the package manager used in a project.

### Development & Build Tools
-   **Vite**: `^6.3.4` - Build tool (used by Vitest).
-   **Vitest**: `^3.1.2` - Test framework.
-   **ESLint**: `^9.22.0` (actual: `@eslint/js@9.24.0`, various plugins) - Linter for TypeScript/JavaScript.
-   **Prettier**: `^3.5.3` - Code formatter.
-   **TypeScript**: `^5.0.3` (actual: `5.8.3`) - Language compiler and type checker.
-   **`ts-node`**: `^10.9.1` (actual: `10.9.2`) - TypeScript execution environment for Node.js.
-   **ESBuild**: `^0.25.2` - Extremely fast JavaScript bundler/minifier (used by `build.mjs`).
-   **Husky**: `^9.1.7` - Git hooks manager.
-   **`lint-staged`**: `^15.5.1` - Run linters on staged Git files.
-   **PNPM**: Used as the package manager (deduced from `pnpm-lock.yaml` and `pnpm-workspace.yaml`).

## Rust Ecosystem (`codex-rs`)
*(Versions from `Cargo.toml` files for `codex-rs` and its sub-crates)*

### Core & Async
-   **Tokio**: `1` (e.g., `1.39.2` in lockfile, specific features like `io-std`, `macros`, `process`, `rt-multi-thread`, `signal` enabled) - Asynchronous runtime for Rust.
-   **Anyhow**: `1` (e.g., `1.0.86`) - Flexible error handling.
-   **Futures**: `0.3` (e.g., `0.3.30`) - Zero-cost asynchronous programming in Rust.
-   **Bytes**: `1.10.1` (actual: `1.7.0`) - Utilities for working with bytes.
-   **Async-channel**: `2.3.1` - Multi-producer, multi-consumer channels for async Rust.

### CLI & UI (TUI)
-   **Clap**: `4` (e.g., `4.5.12`, with `derive` feature) - Command Line Argument Parser.
-   **Ratatui**: `0.29.0` (with features like `unstable-widget-ref`, `unstable-rendered-line-info`) - Library for building Terminal User Interfaces.
-   **Crossterm**: `0.28.1` (with `bracketed-paste` feature) - Terminal manipulation library.
-   **`tui-input`**: `0.11.1` - Input field widget for Ratatui.
-   **`tui-markdown`**: `0.3.3` - Markdown renderer for Ratatui.
-   **`tui-textarea`**: `0.7.0` - Text area widget for Ratatui.
-   **`owo-colors`**: `4.2.0` - Terminal colors (used in `codex-exec`).
-   **`codex-ansi-escape`**: Local workspace crate for ANSI escape codes.

### Serialization & Parsing
-   **Serde**: `1` (e.g., `1.0.204`, with `derive` feature) - Framework for serializing and deserializing Rust data structures.
-   **`serde_json`**: `1` (e.g., `1.0.123`) - JSON support for Serde.
-   **TOML**: `0.8.20` - TOML parser (for configuration).
-   **Regex**: `1` (e.g., `1.10.5`) - Regular expressions.
-   **Shlex**: `1.3.0` - Lexical analysis of shell-like syntaxes.
-   **`mime_guess`**: `2.0` (e.g., `2.0.5`) - Guessing MIME types from file extensions.
-   **Tree-sitter**: `0.25.3` - Incremental parsing system.
-   **`tree-sitter-bash`**: `0.23.3` - Bash grammar for Tree-sitter.

### System & Utilities
-   **Dirs**: `6` (e.g., `6.0.1`) - System directory locations.
-   **`fs-err`**: `3.1.0` - Drop-in replacement for `std::fs` with better error messages.
-   **`fs2`**: `0.4.3` - Cross-platform file locks and file duplication.
-   **Time**: `0.3` (e.g., `0.3.40`, with features `formatting`, `local-offset`, `macros`) - Time and date handling.
-   **Rand**: `0.9` (e.g., `0.9.0-alpha.1`) - Random number generation.
-   **UUID**: `1` (e.g., `1.10.0`, with `serde`, `v4` features) - UUID generation and parsing.
-   **Base64**: `0.21` (e.g., `0.21.7`) - Base64 encoding/decoding.
-   **Patch**: `0.7` (e.g., `0.7.0`) - Applying unified diffs.
-   **`codex-apply-patch`**: Local workspace crate.
-   **`mcp-types`**: Local workspace crate for Multi-Codex Protocol types.

### Sandboxing & Security
-   **Landlock**: `0.4.1` (Linux-specific) - Linux sandboxing mechanism.
-   **Seccompiler**: `0.5.0` (Linux-specific) - Seccomp BPF filter generation.
-   **Libc**: `0.2.172` (e.g., `0.2.160`) - Raw FFI bindings to platform libraries (used for OS interactions).
-   **`openssl-sys`**: `*` (vendored for musl builds) - FFI bindings to OpenSSL.

## External Services & APIs
-   **OpenAI API**: Primary AI model provider.
-   **Other AI Providers**: Configurable to support OpenRouter, Azure, Gemini, Ollama, Mistral, DeepSeek, XAI, Groq, ArceeAI, and other OpenAI API-compatible services.

## Development Environment & Tools
-   **Git**: For version control.
-   **Nix / Nix Flakes**: (`flake.nix`, `flake.lock`) For reproducible development environments.
-   **Docker**: Recommended for sandboxing `codex-cli` on Linux.
-   **`just`**: (`codex-rs/justfile`) A command runner, similar to Make.
-   **`git-cliff`**: (`cliff.toml`) For changelog generation.
-   **Codespell**: (`.codespellrc`) For spell checking in code.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
