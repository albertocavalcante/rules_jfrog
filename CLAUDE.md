# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `rules_jfrog` - a Bazel ruleset for publishing build artifacts to JFrog Artifactory generic repositories. The project has been customized from a Bazel ruleset template and is ready for implementing the core JFrog functionality. The end goal is to provide a simple `artifactory_generic_publish` rule that allows publishing files to Artifactory using bearer token authentication and a Go-based HTTP client.

## Key Commands

### Build and Test
- `bazel build //...` - Build all targets
- `bazel test //...` - Run all tests
- `bazel run //:gazelle` - Update BUILD files (regenerate bzl_library targets)

### Code Formatting and Linting
- `pre-commit install` - Install pre-commit hooks for automatic formatting
- `pre-commit run --all-files` - Run formatters on all files
- The project uses buildifier for Starlark code formatting

### Development Workflow
- Use `bazel run` to execute publish targets (when implemented)
- The project supports both WORKSPACE.bazel and bzlmod (MODULE.bazel) usage

## Architecture

### Current Structure
- `MODULE.bazel` - Bzlmod module definition for rules_jfrog
- `jfrog/` - Main directory containing JFrog-specific functionality
- `jfrog/defs.bzl` - Public API exports (placeholder for `artifactory_generic_publish` rule)
- `jfrog/extensions.bzl` - Bzlmod extensions (placeholder for JFrog configuration)
- `jfrog/repositories.bzl` - Repository rules and dependency management
- `jfrog/toolchain.bzl` - Toolchain implementation (placeholder)
- `jfrog/private/` - Internal implementation details

### Target Implementation (Per PRD.md)
The project structure is ready for implementing:
- `jfrog/publish.bzl` - Main publishing rule implementation (TODO)
- `jfrog/private/uploader/` - Go-based HTTP uploader (TODO)
- Cross-platform binary distribution and execution

### Key Components
- **Toolchain System**: Currently implements a language-specific toolchain pattern that downloads platform-specific tools
- **Cross-Platform Support**: Handles Linux, macOS, and Windows through platform-specific toolchain repositories
- **Bzlmod Integration**: Uses module extensions for toolchain registration and version resolution

## Development Notes

### Implementation Status
The template customization is complete and the project is ready for core functionality implementation:
1. ✅ All template placeholders updated to JFrog-specific naming
2. ✅ Module name updated to "rules_jfrog" 
3. ✅ Directory structure converted from mylang to jfrog
4. 🚧 Need to implement HTTP upload functionality
5. 🚧 Need to implement Go-based uploader binary
6. 🚧 Need to implement `artifactory_generic_publish` rule

### Publishing Implementation Requirements
Based on PRD.md, the core rule should:
- Accept `src`, `repository_url`, `path`, and `auth_token` attributes
- Use HTTP PUT with bearer token authentication
- Support cross-platform execution using Go-based HTTP client
- Handle environment variable substitution for tokens

### Testing
- Integration tests are in `e2e/smoke/` directory
- Unit tests are in `jfrog/tests/` (placeholder tests ready for JFrog functionality)
- CI runs tests in both bzlmod and WORKSPACE modes

### Security Considerations
- Never expose authentication tokens in logs or error messages
- Support environment variable substitution for sensitive data
- Follow Bazel security best practices for rule implementation