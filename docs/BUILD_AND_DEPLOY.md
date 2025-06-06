# Build and Deployment

## Table of Contents
- [Overview](#overview)
- [Development Environment Setup](#development-environment-setup)
  - [Prerequisites](#prerequisites)
  - [Cloning the Repository](#cloning-the-repository)
  - [Node.js Environment (`codex-cli`)](#nodejs-environment-codex-cli)
  - [Rust Environment (`codex-rs`)](#rust-environment-codex-rs)
  - [Nix Flake Development (Alternative)](#nix-flake-development-alternative)
- [Building the Project](#building-the-project)
  - [Building `codex-cli` (TypeScript)](#building-codex-cli-typescript)
  - [Building `codex-rs` (Rust)](#building-codex-rs-rust)
- [Build Tools and Scripts](#build-tools-and-scripts)
  - [`codex-cli` Build Script (`codex-cli/build.mjs`)](#codex-cli-build-script-codex-clibuildmjs)
  - [`codex-rs` Command Runner (`codex-rs/justfile`)](#codex-rs-command-runner-codex-rsjustfile)
- [Deployment Procedures](#deployment-procedures)
  - [Deploying `codex-cli` (npm Package)](#deploying-codex-cli-npm-package)
  - [Deploying `codex-rs` (GitHub Releases)](#deploying-codex-rs-github-releases)
- [Environment Configuration](#environment-configuration)
  - [API Keys](#api-keys)
  - [Codex Configuration Files](#codex-configuration-files)
  - [Project-Specific Instructions (`AGENTS.md`)](#project-specific-instructions-agentsmd)
- [CI/CD Pipeline Details](#cicd-pipeline-details)
  - [TypeScript CI (`.github/workflows/ci.yml`)](#typescript-ci-githubworkflowsciyml)
  - [Rust CI (`.github/workflows/rust-ci.yml`)](#rust-ci-githubworkflowsrust-ciyml)
  - [Rust Release (`.github/workflows/rust-release.yml`)](#rust-release-githubworkflowsrust-releaseyml)

## Overview
This document provides comprehensive instructions for setting up the development environment, building the project, understanding build tools, deployment procedures, and configuring the environment for both `codex-cli` (TypeScript) and `codex-rs` (Rust).

## Development Environment Setup

### Prerequisites
-   **Operating Systems**: macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 (via WSL2).
-   **Node.js**: Version 22 or newer (LTS recommended) for `codex-cli`.
-   **PNPM**: Package manager for the Node.js parts. Install via `corepack enable` (comes with Node.js).
-   **Rust Toolchain**: Latest stable version (e.g., via `rustup`) for `codex-rs`. (The CI uses `dtolnay/rust-toolchain@1.87`).
-   **Git**: For cloning the repository and version control.
-   **Docker** (Optional): Recommended for sandboxing `codex-cli` on Linux.
-   **Nix** (Optional): For using Nix Flake development environment (Nix >= 2.4 with flakes enabled).
-   **`gh` CLI and `zstd`**: Required for `./scripts/install_native_deps.sh` if building `codex-cli` from source on Linux and needing sandboxing binaries.

### Cloning the Repository
```bash
git clone https://github.com/openai/codex.git
cd codex
```

### Node.js Environment (`codex-cli`)
1.  **Navigate to `codex-cli` directory**:
    ```bash
    cd codex-cli
    ```
2.  **Enable Corepack** (if not already enabled):
    ```bash
    corepack enable
    ```
3.  **Install dependencies**:
    ```bash
    pnpm install
    ```
    This will install all necessary Node.js packages defined in `package.json` and `pnpm-lock.yaml`.

### Rust Environment (`codex-rs`)
1.  **Navigate to `codex-rs` directory**:
    ```bash
    cd codex-rs
    ```
2.  **Install Rust toolchain**: If not already installed, get it from [rustup.rs](https://rustup.rs/).
3.  Cargo (Rust's package manager) will automatically handle dependencies when building or running commands.

### Nix Flake Development (Alternative)
The project supports Nix flakes for a reproducible development environment.
1.  **Prerequisite**: Nix >= 2.4 with flakes enabled (`experimental-features = nix-command flakes` in `~/.config/nix/nix.conf`).
2.  **Enter development shell**:
    -   For `codex-cli`: `nix develop .#codex-cli`
    -   For `codex-rs`: `nix develop .#codex-rs`
    This shell includes necessary dependencies like Node.js and Rust, installs project dependencies, and provides a `codex` command alias.
3.  **Using with Direnv**: Create a `.envrc` file in the respective subdirectories (`codex-cli` or `codex-rs`) with `use flake ../flake.nix#<codex-cli_or_codex-rs>` and run `direnv allow`.

## Building the Project

### Building `codex-cli` (TypeScript)
From the `codex-cli/` directory:
-   **Development Build**:
    ```bash
    pnpm build:dev
    ```
    This creates a non-minified build with inline sourcemaps in `codex-cli/dist/cli-dev.js`.
-   **Production Build**:
    ```bash
    pnpm build
    ```
    This creates a minified production build in `codex-cli/dist/cli.js` with external sourcemaps.
-   **Linux Native Dependencies**: If building from source on Linux and requiring sandboxing binaries:
    ```bash
    ./scripts/install_native_deps.sh
    ```

### Building `codex-rs` (Rust)
From the `codex-rs/` directory:
-   **Debug Build (for development)**:
    ```bash
    cargo build
    ```
-   **Release Build**:
    ```bash
    cargo build --release
    ```
    Binaries will be located in `codex-rs/target/debug/` or `codex-rs/target/release/`.
-   **Specific targets** (as used in CI):
    ```bash
    cargo build --target <target-triple> # e.g., x86_64-unknown-linux-musl
    ```

## Build Tools and Scripts

### `codex-cli` Build Script (`codex-cli/build.mjs`)
-   **Tool**: Uses `esbuild` for fast TypeScript bundling and transformation.
-   **Functionality**:
    -   Bundles `src/cli.tsx` into an ESM module.
    -   Handles different build modes (development vs. production) for minification and sourcemaps.
    -   Includes a plugin (`ignoreReactDevToolsPlugin`) to strip an ESM-unfriendly import from Ink.
    -   Adjusts the shebang for development builds to enable Node.js source map support.
    -   Outputs to `codex-cli/dist/`.

### `codex-rs` Command Runner (`codex-rs/justfile`)
-   **Tool**: Uses `just`, a command runner.
-   **Functionality**: Provides recipes for common tasks:
    -   `just help`: Displays available commands.
    -   `just install`: Installs `codex-tui` binary.
    -   `just tui *args`: Runs the TUI app.
    -   `just proto *args`: Runs the Proto app (likely related to MCP development).

## Deployment Procedures

### Deploying `codex-cli` (npm Package)
The `codex-cli` is published as an npm package `@openai/codex`.
1.  **Staging the release**:
    From the `codex-cli/` directory, run the `stage_release.sh` script:
    -   For a standard release: `pnpm stage-release` or `pnpm stage-release --tmp <directory>`
    -   For a "fat" package including Rust binaries for Linux: `pnpm stage-release --native`
    This script:
        - Builds the JS artifacts.
        - Copies necessary files (`bin/`, `dist/`, `src/`, `README.md`) to a temporary staging directory.
        - Generates a timestamp-based version number.
        - Modifies `package.json` in the staging directory with the new version.
        - If `--native` is specified, it runs `scripts/install_native_deps.sh --full-native` to bundle Rust binaries.
2.  **Publishing**:
    After verifying the staged package in the temporary directory:
    ```bash
    cd /path/to/staging_directory
    npm publish
    ```
    Or `npm publish --tag native` if it's a native-bundled release.

### Deploying `codex-rs` (GitHub Releases)
The `codex-rs` components are intended to be released as pre-compiled binaries via GitHub Releases.
-   **Trigger**: Pushing a Git tag matching the pattern `rust-v*.*.*` (e.g., `rust-v0.1.0`).
-   **Workflow**: `.github/workflows/rust-release.yml`
    1.  **Tag Check**: Validates that the tag matches the version in `codex-rs/Cargo.toml`.
    2.  **Build**: Builds release binaries for multiple targets (macOS aarch64/x86_64, Linux x86_64 musl/gnu, Linux aarch64 gnu).
        -   Uses `cargo build --target <target-triple> --release --all-targets --all-features`.
        -   Stages artifacts in `codex-rs/dist/<target-triple>/`.
        -   Compresses artifacts using `zstd`.
    3.  **Release**:
        -   Uploads artifacts to a GitHub Release tagged as `codex-rs-${{ github.sha }}-${{ github.run_attempt }}-${{ github.ref_name }}`.
        -   Marks the release as "prerelease".
        -   Uses `facebook/dotslash-publish-release` for further release steps defined in `.github/dotslash-config.json`.

## Environment Configuration

### API Keys
-   **OpenAI API Key**: Set via `OPENAI_API_KEY` environment variable or in a `.env` file at the project root.
-   **Other Providers**: For services like Azure, OpenRouter, etc., specific environment variables (e.g., `AZURE_OPENAI_API_KEY`, `<PROVIDER>_API_KEY`) are required as detailed in the main `README.md` and `codex-rs/README.md`.

### Codex Configuration Files
-   **`codex-cli`**:
    -   `~/.codex/config.json` or `~/.codex/config.yaml`.
    -   Supports parameters like `model`, `approvalMode`, `providers`, `history`.
-   **`codex-rs`**:
    -   `~/.codex/config.toml` (or path specified by `CODEX_HOME`).
    -   Supports parameters like `model`, `model_provider`, `approval_policy`, `profiles`, `sandbox_permissions`, `mcp_servers`.

### Project-Specific Instructions (`AGENTS.md`)
-   Codex can load custom instructions from `AGENTS.md` files located at:
    1.  `~/.codex/AGENTS.md` (global)
    2.  `AGENTS.md` (repository root)
    3.  `AGENTS.md` (current working directory)
-   Disable with `--no-project-doc` or `CODEX_DISABLE_PROJECT_DOC=1`.

## CI/CD Pipeline Details

### TypeScript CI (`.github/workflows/ci.yml`)
-   **Trigger**: Pull requests and pushes to `main`.
-   **Environment**: Ubuntu latest, Node.js 22, PNPM.
-   **Steps**:
    -   Checkout, setup Node/PNPM, cache PNPM store.
    -   Install dependencies (`pnpm install`).
    -   Format checks (TypeScript, Markdown, config files).
    -   Run tests (`pnpm run test`).
    -   Linting (`eslint`).
    -   Type checking (`pnpm run typecheck`).
    -   Build (`pnpm run build`).
    -   Test staging a release (`pnpm stage-release`).
    -   ASCII check and ToC check for `README.md`.

### Rust CI (`.github/workflows/rust-ci.yml`)
-   **Trigger**: Pull requests and pushes to `main` affecting `codex-rs/**` or `.github/**`, also workflow dispatch.
-   **Jobs**:
    -   `general`: Format check (`cargo fmt`).
    -   `lint_build_test`: Runs on a matrix of OS (macOS, Ubuntu, Windows) and Rust targets.
        -   Steps: Checkout, setup Rust toolchain (with clippy), cache Cargo dependencies.
        -   Install musl tools for `x86_64-unknown-linux-musl` target.
        -   Run `cargo clippy`.
        -   Run `cargo build` for individual crates (on `x86_64-unknown-linux-gnu` only, for stricter feature checking).
        -   Run `cargo test`.

### Rust Release (`.github/workflows/rust-release.yml`)
-   **Trigger**: Pushing Git tags matching `rust-v*.*.*`.
-   **Jobs**:
    -   `tag-check`: Verifies the Git tag version matches the version in `codex-rs/Cargo.toml`.
    -   `build`: Builds release binaries for various targets (macOS aarch64/x86_64, Linux x86_64 musl/gnu, Linux aarch64 gnu).
        -   Uploads compressed artifacts for each target.
    -   `release`: Creates a GitHub pre-release with the built artifacts.
        -   Uses `softprops/action-gh-release`.
        -   Uses `facebook/dotslash-publish-release`.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
