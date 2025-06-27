# Product Requirements Document: rules_jfrog

## Executive Summary

Create a Bazel ruleset called `rules_jfrog` that enables teams to publish build artifacts to JFrog Artifactory generic repositories directly from their Bazel build process. This initial version focuses exclusively on **publishing files to generic repositories using bearer token authentication**.

**Project Name**: `rules_jfrog`  
**Version**: 1.0.0  
**Target Audience**: Development teams using Bazel build system with JFrog Artifactory  

## Problem Statement

Teams using Bazel for builds often need to publish artifacts (binaries, libraries, configuration files) to JFrog Artifactory as part of their CI/CD pipeline. Currently, this requires:
- Manual upload scripts outside of Bazel
- Complex CI/CD pipeline configurations
- Inconsistent artifact publishing processes
- No integration with Bazel's dependency graph

## Goals and Objectives

### Primary Goal
Provide a simple, native Bazel rule to publish build artifacts to JFrog Artifactory generic repositories.

### Success Criteria
- Developers can publish any Bazel target output to Artifactory with a single `bazel run` command
- Publishing integrates seamlessly with existing Bazel builds
- Cross-platform compatibility (Linux, macOS, Windows)
- Secure authentication handling

## User Stories

### US-001: Basic File Publishing
**As a** developer  
**I want to** publish a compiled binary to Artifactory  
**So that** other teams can download and use the latest version  

**Acceptance Criteria:**
- Can specify source file, target repository URL, and destination path
- Upload succeeds with proper authentication
- Build fails if upload fails

### US-002: Configuration File Publishing  
**As a** DevOps engineer  
**I want to** publish configuration files to a shared repository  
**So that** deployment processes can fetch the latest configs  

**Acceptance Criteria:**
- Can publish any file type (JSON, YAML, binary, etc.)
- Files are uploaded to specified paths in the repository
- Process is repeatable and reliable

### US-003: Secure Authentication
**As a** security-conscious developer  
**I want to** authenticate to Artifactory using bearer tokens  
**So that** my credentials are secure and can be rotated  

**Acceptance Criteria:**
- Supports bearer token authentication only
- Tokens can be passed via environment variables
- No credentials are hardcoded in build files

## Functional Requirements

### FR-001: Publishing Rule
- **Rule Name**: `artifactory_generic_publish`
- **Purpose**: Upload a single file to Artifactory generic repository
- **Required Attributes**:
  - `src`: Label pointing to the file to upload
  - `repository_url`: String containing the base URL of the Artifactory generic repository
  - `path`: String specifying the target path within the repository
  - `auth_token`: String containing the bearer token for authentication

### FR-002: Upload Implementation
- Use HTTP PUT method to upload files
- Set `Authorization: Bearer <token>` header
- Set appropriate `Content-Type` header
- Handle upload failures gracefully with clear error messages

### FR-003: Cross-Platform Support
- Work on Linux, macOS, and Windows
- Use Go-based HTTP client for consistent cross-platform behavior
- Single binary approach eliminates platform-specific script dependencies

### FR-004: Bazel Integration
- Follow Bazel best practices for rule implementation
- Use Bzlmod (MODULE.bazel) for dependency management
- Provide clean public API through `defs.bzl`
- Support `bazel run` execution model

## Non-Functional Requirements

### NFR-001: Performance
- Upload should be efficient for files up to 1GB
- Minimal overhead in Bazel build graph analysis
- No unnecessary file copies during upload

### NFR-002: Security
- Never log or expose authentication tokens
- Support environment variable substitution for tokens
- Fail securely if authentication fails

### NFR-003: Reliability  
- Retry failed uploads (at least 3 attempts)
- Provide clear error messages for common failure scenarios
- Validate inputs before attempting upload

### NFR-004: Maintainability
- Go-based HTTP upload implementation with Starlark rule definitions
- Well-documented code with inline comments
- Follows Bazel style guidelines
- Comprehensive error handling
- Single Go binary for cross-platform consistency

## Technical Specifications

### Project Structure
```
rules_jfrog/
├── MODULE.bazel                 # Bzlmod module definition
├── jfrog/
│   ├── defs.bzl                # Public API exports
│   ├── publish.bzl             # Main publishing rule implementation
│   └── private/
│       ├── utils.bzl           # Internal utilities (if needed)
│       └── uploader/           # Go-based HTTP uploader
│           ├── main.go         # HTTP upload implementation
│           ├── go.mod          # Go module definition
│           └── BUILD.bazel     # Bazel build for Go binary
├── examples/
│   └── basic_publish/          # Usage examples
├── README.md                   # Documentation
└── LICENSE                     # License file
```

### API Specification

```starlark
artifactory_generic_publish(
    name = "publish_my_file",
    src = ":my_target",
    repository_url = "https://company.jfrog.io/artifactory/generic-local",
    path = "releases/my-app/v1.0.0/my-app-linux",
    auth_token = "$(ARTIFACTORY_TOKEN)",
)
```

### Implementation Requirements

1. **Rule Implementation**:
   - Use Bazel's `rule()` function with proper implementation
   - Execute Go binary for upload via `ctx.actions.run()`
   - Pass parameters via command-line arguments to Go binary

2. **HTTP Upload (Go Implementation)**:
   - Go-based HTTP client using `net/http` package
   - Cross-platform binary (no shell script dependencies)
   - Consistent behavior across Linux, macOS, and Windows
   - Handle both success and failure cases with proper exit codes

3. **Error Handling**:
   - Validate repository URL format
   - Check for required authentication
   - Provide meaningful error messages for HTTP failures

### Example Usage

```starlark
# In user's MODULE.bazel
bazel_dep(name = "rules_jfrog", version = "1.0.0")

# In user's BUILD.bazel
load("@rules_jfrog//jfrog:defs.bzl", "artifactory_generic_publish")

cc_binary(
    name = "my_tool",
    srcs = ["main.cc"],
)

artifactory_generic_publish(
    name = "publish_tool",
    src = ":my_tool", 
    repository_url = "https://company.jfrog.io/artifactory/tools",
    path = "linux/my-tool/v1.2.3/my-tool",
    auth_token = "$(ARTIFACTORY_TOKEN)",
)
```

```bash
# Usage
export ARTIFACTORY_TOKEN="your-token-here"
bazel run :publish_tool
```

## Success Metrics

- **Functionality**: Successfully uploads files to Artifactory in 100% of valid test cases
- **Performance**: Uploads complete within 30 seconds for files under 100MB
- **Reliability**: Less than 1% failure rate due to implementation issues
- **Usability**: New users can successfully publish a file within 5 minutes of reading documentation

## Out of Scope (Version 1.0)

- **Repository downloading/fetching** - Only publishing is supported
- **Maven, npm, Docker repository support** - Only generic repositories
- **Authentication methods other than bearer tokens** - No API keys, username/password
- **Batch uploads** - Only single file uploads per rule
- **Properties/metadata** - No Artifactory properties support
- **Checksum verification** - No built-in checksum validation
- **Progress reporting** - No upload progress indicators

## Implementation Notes

### Platform-Specific Considerations

**All Platforms (Linux/macOS/Windows)**:
- Single Go binary for HTTP uploads
- Consistent behavior across all platforms
- No dependency on external tools (curl, PowerShell)
- Hermetic build with pre-built Go binaries

### Security Considerations

- Never hardcode tokens in BUILD files
- Support environment variable substitution: `"$(ENV_VAR)"`
- Recommend using Bazel's `select()` for environment-specific tokens
- Clear error messages without exposing sensitive information

### Testing Strategy

- Unit tests for rule implementation
- Integration tests with mock Artifactory server
- Cross-platform compatibility tests
- Error handling and edge case tests

## Future Considerations (Post-1.0)

- Generic repository downloading/fetching
- Support for other Artifactory repository types
- Properties and metadata support
- Batch upload capabilities
- Checksum verification
- Progress reporting and logging improvements
