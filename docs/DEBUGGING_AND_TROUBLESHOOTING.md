# Debugging and Troubleshooting

## Table of Contents
- [Overview](#overview)
- [General Debugging Techniques](#general-debugging-techniques)
  - [Verbose Logging (`codex-cli`)](#verbose-logging-codex-cli)
  - [Visual Debugger (`codex-cli`)](#visual-debugger-codex-cli)
  - [Rust Debugging (`codex-rs`)](#rust-debugging-codex-rs)
- [Logging](#logging)
  - [Console Output](#console-output)
  - [API Request/Response Logging](#api-requestresponse-logging)
  - [Rust Tracing/Logging](#rust-tracinglogging)
- [Common Issues and Solutions](#common-issues-and-solutions)
  - [API Key Issues](#api-key-issues)
  - [Model Access Problems (o3/o4-mini)](#model-access-problems-o3o4-mini)
  - [Zero Data Retention (ZDR) Errors](#zero-data-retention-zdr-errors)
  - [Installation Issues](#installation-issues)
  - [Unexpected File Edits or Commands](#unexpected-file-edits-or-commands)
  - [Windows Compatibility](#windows-compatibility)
- [Performance Monitoring](#performance-monitoring)
- [Testing Strategies](#testing-strategies)
  - [`codex-cli` (TypeScript)](#codex-cli-typescript)
  - [`codex-rs` (Rust)](#codex-rs-rust)
  - [CI Testing](#ci-testing)

## Overview
This document provides guidance on debugging, troubleshooting common issues, understanding logging, and testing strategies for the Codex CLI project.

## General Debugging Techniques

### Verbose Logging (`codex-cli`)
For `codex-cli`, you can enable detailed logging of API requests and responses by setting the `DEBUG` environment variable:
```bash
DEBUG=true codex
```
This is useful for understanding the interaction between the CLI and the AI model provider.

### Visual Debugger (`codex-cli`)
To debug `codex-cli` using a visual debugger like the one in VS Code or Chrome Developer Tools:
1.  **Build the CLI with sourcemaps**: Ensure you have a development build (e.g., by running `pnpm build` which generates sourcemaps, or `pnpm build:dev` for inline sourcemaps). The `codex-cli/dist/cli.js.map` or `codex-cli/dist/cli-dev.js.map` file should exist.
2.  **Run Node.js with `--inspect-brk`**:
    ```bash
    node --inspect-brk ./codex-cli/dist/cli.js [your codex arguments]
    ```
    (Use `cli-dev.js` if you used `pnpm build:dev`)
    The program will pause until a debugger is attached.
3.  **Attach Debugger**:
    -   **VS Code**: Use the "Debug: Attach to Node Process" command from the command palette and select the process running on port `9229`.
    -   **Chrome**: Navigate to `chrome://inspect` and find the target on `localhost:9229`, then click "inspect" (or "trace").

### Rust Debugging (`codex-rs`)
For `codex-rs` components, standard Rust debugging techniques apply:
-   Use a debugger like `gdb` or `lldb`.
-   Compile in debug mode (default for `cargo build`) for more debugging symbols.
-   Utilize print statements (`println!`, `dbg!`) or the `tracing` crate for logging.
-   IDE integrations (e.g., with VS Code using `rust-analyzer` and CodeLLDB extension) provide a good debugging experience.

## Logging

### Console Output
Both `codex-cli` and `codex-rs` output information to the console. The level of detail can vary based on verbosity settings or debug flags.

### API Request/Response Logging
As mentioned above, `DEBUG=true` for `codex-cli` enables logging of full API request and response details. For `codex-rs`, the `tracing` crate is used, and its output can be configured to show similar levels of detail, often controlled by environment variables like `RUST_LOG`.

### Rust Tracing/Logging
The `codex-rs` components use the `tracing` crate for structured logging.
-   **Configuration**: Typically configured via the `RUST_LOG` environment variable (e.g., `RUST_LOG=codex_core=debug,codex_tui=info`).
-   **Output**: Can be directed to stderr or files using `tracing-subscriber` and `tracing-appender`. The TUI component (`codex-rs/tui/`) has specific log layer setup.

## Common Issues and Solutions

### API Key Issues
-   **Issue**: Errors related to API key (e.g., "Invalid API key").
-   **Solution**:
    -   Ensure `OPENAI_API_KEY` (or provider-specific key like `AZURE_OPENAI_API_KEY`) is correctly set as an environment variable or in a `.env` file.
    -   Verify the key is active and has sufficient credits/quota on the provider's platform.
    -   If using a custom provider, ensure the `envKey` and `baseURL` in the configuration are correct.

### Model Access Problems (o3/o4-mini)
-   **Issue**: `o3` or `o4-mini` models not working.
-   **Solution**:
    -   Your OpenAI API account might need verification to stream responses. Check the OpenAI platform for any verification steps.
    -   Ensure the model name is correct and supported by your API key/organization.

### Zero Data Retention (ZDR) Errors
-   **Issue**: `400 Previous response cannot be used for this organization due to Zero Data Retention.`
-   **Solution**:
    -   If your OpenAI organization has ZDR enabled, ensure you are using a recent version of `codex-cli`: `npm i -g @openai/codex@latest`.
    -   For `codex-rs`, set `disable_response_storage = true` in your `~/.codex/config.toml`.

### Installation Issues
-   **Issue**: Problems installing `codex-cli` with `npm install -g`.
-   **Solution**:
    -   Ensure Node.js version 22+ is installed.
    -   Avoid using `sudo npm install -g`. Fix npm permissions instead if you encounter permission errors. (See npm documentation for `npm config set prefix` or using a Node version manager like `nvm`).

### Unexpected File Edits or Commands
-   **Issue**: Codex suggests or applies changes that are incorrect or harmful.
-   **Solution**:
    -   Use `approvalMode = "suggest"` (default) or `auto-edit` instead of `full-auto` to review changes before they are applied.
    -   If a change is denied, provide feedback to the model to guide it better.
    -   Ensure your work is under Git version control so you can easily revert unwanted changes.

### Windows Compatibility
-   **Issue**: `codex-cli` does not run directly on Windows.
-   **Solution**: Use Windows Subsystem for Linux (WSL2).

## Performance Monitoring
-   No specific performance monitoring tools are documented within the project itself.
-   Standard OS tools (e.g., `top`, `htop`, Activity Monitor) can be used to monitor CPU and memory usage.
-   For `codex-rs`, profiling tools compatible with Rust (e.g., `perf`, Valgrind with Callgrind, Instruments on macOS) can be used on debug or release builds with debug symbols.

## Testing Strategies

### `codex-cli` (TypeScript)
-   **Framework**: [Vitest](https://vitest.dev/) (`^3.1.2`)
-   **Test Files**: Located in `codex-cli/tests/`. Files typically end with `.test.ts` or `.test.tsx`.
-   **Running Tests**:
    -   Run all tests: `pnpm test` (from `codex-cli/` directory)
    -   Watch mode (rerun on changes): `pnpm test:watch`
-   **Coverage**: The project aims for meaningful assertions, though 100% coverage is not strictly required.
-   **Types of Tests**:
    -   Unit tests for utility functions and individual components.
    -   Integration tests for agent loops, API interactions (often mocked), and CLI behavior.
    -   Snapshot tests (e.g., `codex-cli/tests/__snapshots__/`) for UI components or command outputs.
-   **Test Environment**: `ink-testing-library` is used for testing Ink components.

### `codex-rs` (Rust)
-   **Framework**: Standard Rust testing framework (`cargo test`).
-   **Test Files**:
    -   Unit tests are often co-located with the code in `src/*.rs` files within `#[cfg(test)]` modules.
    -   Integration tests are typically in the `tests/` directory of each crate (e.g., `codex-rs/core/tests/`).
-   **Running Tests**:
    -   From the `codex-rs/` workspace root: `cargo test --all-features`
    -   For a specific crate: `cargo test -p <crate_name>`
-   **Assertions**: Uses standard Rust `assert!` macros and libraries like `pretty_assertions` and `predicates` for more expressive assertions.
-   **Live Tests**: Some tests (`live_agent.rs`, `live_cli.rs` in `codex-rs/core/tests/`) might interact with actual APIs or system resources, requiring careful setup or mocking (e.g., using `wiremock`).

### CI Testing
-   **`codex-cli`**: The `.github/workflows/ci.yml` workflow runs formatting checks, lints, type checks, unit/integration tests (`pnpm test`), and a build check on Ubuntu with Node.js 22.
-   **`codex-rs`**: The `.github/workflows/rust-ci.yml` workflow runs `cargo fmt` (formatting), `cargo clippy` (linting), `cargo build`, and `cargo test` across a matrix of operating systems (macOS, Ubuntu, Windows) and Rust target architectures.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
