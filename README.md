# Bazel rules for JFrog Artifactory

[![CI Status](https://github.com/albertocavalcante/rules_jfrog/workflows/CI/badge.svg)](https://github.com/albertocavalcante/rules_jfrog/actions)

A Bazel ruleset for publishing build artifacts to JFrog Artifactory generic repositories.

## 🚧 Status: In Development

**This project is currently under active development and not yet ready for production use.**

### What's Implemented ✅
- [x] Project structure and template customization complete
- [x] Bzlmod (MODULE.bazel) support with rules_jfrog module
- [x] Repository structure following Bazel best practices
- [x] CI/CD pipeline configuration for albertocavalcante/rules_jfrog
- [x] Cross-platform architecture planning (Go-based implementation)
- [x] All template placeholders updated to JFrog-specific naming
- [x] Build and test infrastructure working

### What's Pending 🚧
- [ ] **Core functionality**: `artifactory_generic_publish` rule implementation
- [ ] **Go-based uploader**: HTTP client for cross-platform artifact uploads
- [ ] **Authentication**: Bearer token support with environment variable substitution
- [ ] **Error handling**: Comprehensive error reporting and retry logic
- [ ] **Documentation**: Complete usage examples and API documentation
- [ ] **Testing**: Unit tests and integration tests
- [ ] **Release**: First stable release (v1.0.0)

## Overview

`rules_jfrog` enables teams to publish build artifacts directly to JFrog Artifactory generic repositories from their Bazel build process. The ruleset uses a **Go-based HTTP client** for consistent cross-platform behavior, eliminating dependencies on external tools like `curl` or PowerShell.

### Key Features (Planned)

- 🎯 **Simple API**: Single rule for publishing any file to Artifactory
- 🌐 **Cross-platform**: Works on Linux, macOS, and Windows via Go binary
- 🔐 **Secure**: Bearer token authentication with environment variable support
- 📦 **Hermetic**: No external dependencies, self-contained Go binary
- 🚀 **Fast**: Direct HTTP uploads with proper error handling and retries
- 🎛️ **Bazel-native**: Integrates seamlessly with Bazel's build graph

## Planned Usage

> ⚠️ **Note**: This is the intended API design. Implementation is still in progress.

### Installation (Future)

Add to your `MODULE.bazel`:

```starlark
bazel_dep(name = "rules_jfrog", version = "1.0.0")
```

### Basic Usage (Future)

```starlark
# In your BUILD.bazel file
load("@rules_jfrog//jfrog:defs.bzl", "artifactory_generic_publish")

cc_binary(
    name = "my_app",
    srcs = ["main.cc"],
)

artifactory_generic_publish(
    name = "publish_app",
    src = ":my_app",
    repository_url = "https://company.jfrog.io/artifactory/generic-local",
    path = "releases/my-app/v1.0.0/my-app-linux",
    auth_token = "$(ARTIFACTORY_TOKEN)",
)
```

```bash
# Set your authentication token
export ARTIFACTORY_TOKEN="your-bearer-token-here"

# Publish the artifact
bazel run :publish_app
```

## Architecture

### Go-Based Implementation

Unlike traditional shell-script based approaches, `rules_jfrog` uses a **Go binary** for HTTP uploads:

- **Consistent behavior** across all platforms (Linux, macOS, Windows)
- **No external dependencies** on `curl`, `wget`, or PowerShell
- **Better error handling** with structured error messages and exit codes
- **HTTP/2 support** with connection reuse and proper timeout handling

### Project Structure

```
rules_jfrog/
├── MODULE.bazel                 # Bzlmod module definition
├── jfrog/
│   ├── defs.bzl                # Public API exports
│   ├── publish.bzl             # Publishing rule implementation (TODO)
│   └── private/
│       ├── toolchains_repo.bzl # Platform detection & Go binary management
│       └── uploader/           # Go-based HTTP uploader (TODO)
│           ├── main.go         # HTTP upload implementation
│           ├── go.mod          # Go module definition
│           └── BUILD.bazel     # Bazel build for Go binary
├── e2e/smoke/                  # Integration tests
├── examples/                   # Usage examples (TODO)
├── docs/                       # Documentation
├── README.md                   # This file
├── PRD.md                      # Product Requirements Document
├── CONTRIBUTING.md             # Contribution guidelines
└── LICENSE                     # License file
```

## Development Status

This project was created from a Bazel ruleset template and is being adapted for JFrog Artifactory publishing. The template provided excellent infrastructure for:

- Bzlmod support with proper module extensions
- Cross-platform CI/CD with GitHub Actions  
- Pre-commit hooks for code formatting (buildifier)
- Automated release processes
- Comprehensive testing setup

### Current Development Phase

The **template customization phase is complete** ✅. All placeholder references have been updated and the project structure is ready for implementation. The next major milestone is implementing the actual publishing functionality with the Go-based HTTP uploader.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and contribution guidelines.

### Development Commands

```bash
# Install pre-commit hooks for formatting
pre-commit install

# Build all targets
bazel build //...

# Run all tests (when implemented)
bazel test //...

# Update BUILD files
bazel run //:gazelle

# Format all files  
pre-commit run --all-files
```

## Roadmap

### Phase 1: Core Implementation (Current)
- [ ] Implement `artifactory_generic_publish` rule
- [ ] Build Go-based HTTP uploader with cross-platform binaries
- [ ] Add comprehensive error handling and logging
- [ ] Create basic usage examples

### Phase 2: Enhanced Features
- [ ] Batch upload support
- [ ] Checksum verification
- [ ] Progress reporting for large files
- [ ] Configuration file support

### Phase 3: Advanced Features  
- [ ] Support for other Artifactory repository types (Maven, npm, Docker)
- [ ] Artifact downloading/fetching capabilities
- [ ] Properties and metadata support
- [ ] Advanced retry and resilience features

## License

This project is licensed under the [Apache License 2.0](LICENSE).

## Support

- 📖 **Documentation**: See [docs/](docs/) directory
- 🐛 **Issues**: Report bugs and feature requests via [GitHub Issues](https://github.com/albertocavalcante/rules_jfrog/issues)
- 💬 **Discussions**: Join conversations in [GitHub Discussions](https://github.com/albertocavalcante/rules_jfrog/discussions)

---

**Note**: This README will be updated as development progresses. Check back frequently for the latest status and usage instructions.