# Codex CLI Documentation

## Table of Contents
- [Project Overview](#project-overview)
- [Quick Start](#quick-start)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Further Documentation](#further-documentation)

## Project Overview
Codex CLI is a lightweight, open-source coding agent that runs in your terminal. It aims to provide ChatGPT-level reasoning with the power to execute code, manipulate files, and iterate, all under version control. It's designed for developers who are comfortable working in a terminal environment.

The project consists of two main parts:
- **`codex-cli`**: The primary interface, written in TypeScript using Node.js and React/Ink for the terminal UI. This is the current stable and most feature-rich component.
- **`codex-rs`**: A newer implementation in Rust, aiming for smaller, standalone binaries, better performance, and enhanced sandboxing capabilities, especially on Linux. This component is under active development.

Codex CLI is an experimental project.

## Quick Start

1.  **Install `codex-cli` globally:**
    ```bash
    npm install -g @openai/codex
    ```
2.  **Set your OpenAI API Key:**
    ```bash
    export OPENAI_API_KEY="your-api-key-here"
    ```
    (Or set it in a `.env` file in your project root)
3.  **Run interactively:**
    ```bash
    codex
    ```
    Or with an initial prompt:
    ```bash
    codex "explain this codebase to me"
    ```

## Prerequisites

-   **Operating Systems**: macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 (via WSL2).
-   **Node.js**: Version 22 or newer (LTS recommended) for `codex-cli`.
-   **Git** (Optional, Recommended): Version 2.23+ for version control integration.
-   **Rust Toolchain** (for `codex-rs` development/contribution): Latest stable version.

## Installation

### `codex-cli` (TypeScript - Recommended for general use)

Install globally using npm (or your preferred package manager):
```bash
npm install -g @openai/codex
```

For alternative installation methods or building from source, refer to the main project `README.md`.

### `codex-rs` (Rust - Under Development)

The Rust version is not yet recommended for general use. Build instructions can be found in `codex-rs/README.md`. Pre-compiled binaries will be available via GitHub Releases once it's more mature.

## Further Documentation

This document provides a high-level overview. For more detailed information, please refer to the following documents in this `docs/` directory:

-   [`ARCHITECTURE.md`](./ARCHITECTURE.md): High-level system architecture and design patterns.
-   [`TECH_STACK.md`](./TECH_STACK.md): Detailed breakdown of technologies, frameworks, and libraries used.
-   [`COMPONENTS.md`](./COMPONENTS.md): In-depth look at the major components and modules of `codex-cli` and `codex-rs`.
-   [`BUILD_AND_DEPLOY.md`](./BUILD_AND_DEPLOY.md): Instructions for building the project, development setup, and deployment.
-   [`DEBUGGING_AND_TROUBLESHOOTING.md`](./DEBUGGING_AND_TROUBLESHOOTING.md): Tips for debugging, common issues, and testing.
-   [`API_REFERENCE.md`](./API_REFERENCE.md): Documentation for CLI arguments and configuration options.
-   [`DATABASE_SCHEMA.md`](./DATABASE_SCHEMA.md): Information on data storage, such as history and configuration files.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}* (Note: Version information will be filled in during a later step if found)
