# Project Overview: Example Bazel Monorepo

## Introduction

Welcome to the Example Bazel Monorepo! This project is designed as a comprehensive learning resource for developers who want to master Bazel, Google's powerful build and test tool. The repository demonstrates how to build and manage a multi-language monorepo using Bazel, supporting **Golang**, **Java**, **Scala**, **Python**, and **TypeScript**.

Rather than using a typical "to-do list" application, this project models a book shop and reading catalogue website called **Antilibrary** 📗📕📒📚, providing a more interesting and realistic domain context.

**Target Audience**: Developers learning Bazel from zero to hero, looking for practical examples and best practices.

**Bazel Version**: Currently supporting Bazel 4.1.0 (as of mid-June 2021)

## Why Bazel and Monorepo?

### Benefits of Bazel:
- **Fast, incremental builds**: Only rebuilds what changed
- **Reproducible builds**: Same inputs = same outputs, every time
- **Multi-language support**: Build everything in one system
- **Scalability**: Handles massive codebases (used by Google, Uber, Dropbox)
- **Hermetic builds**: All dependencies explicitly declared

### Benefits of Monorepo:
- **Atomic commits**: Changes across multiple services in one commit
- **Simplified dependency management**: One version of truth
- **Easy code sharing and reuse**: Import any code in the repo
- **Unified tooling**: One build system, one CI/CD pipeline
- **Better collaboration**: All code visible to everyone

## High-Level Architecture

This monorepo implements an "Antilibrary" system with the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                    Antilibrary System                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Frontend   │───▶│   Store API  │───▶│   Database   │  │
│  │ (TypeScript) │    │   (Java)     │    │  (Postgres)  │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                               │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │     CLI      │    │ Book Sorting │    │   Scraping   │  │
│  │   (Golang)   │    │ (Python/Scala│    │   (Python)   │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐  │
│  │        Infrastructure (Terraform - AWS/GCloud)        │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Component Overview:

1. **Frontend** (`/frontend`): TypeScript/React web application
2. **Store API** (`/store-api`): Spring Boot REST API with PostgreSQL
3. **CLI** (`/cli`): Golang command-line interface for "Blind Date With a 📖"
4. **Book Sorting** (`/book_sorting`, `/scala-book-sorting`): Python and Scala implementations
5. **Scraping** (`/scraping`): Python web scraper for NYT bestsellers
6. **Store Layout Solver** (`/store/layoutsolver`): Java optimization algorithms
7. **Infrastructure** (`/infrastructure`): Terraform IaC for AWS and GCloud
8. **PyPI Package** (`/py_antilibrary`): Distributable Python package

## Repository Structure

```
example-bazel-monorepo/
│
├── WORKSPACE.bazel          # Root workspace file - defines external dependencies
├── BUILD.bazel              # Root build file - minimal, mostly for buildifier
├── .bazelrc                 # Bazel configuration and flags
├── .bazelversion            # Pins Bazel version (4.1.0)
│
├── 3rdparty/                # Third-party dependencies management
│   ├── go_workspace.bzl     # Go dependencies
│   ├── jvm_workspace.bzl    # Scala/JVM dependencies (via bazel-deps)
│   ├── maven_install.json   # Pinned Maven dependencies for Java
│   ├── requirements.in      # Python pip dependencies (source)
│   ├── requirements.txt     # Python pip dependencies (pinned)
│   └── typescript/          # TypeScript/NodeJS package.json & yarn.lock
│
├── book_sorting/            # Python library for book sorting algorithms
│   ├── BUILD.bazel
│   └── main.py
│
├── cli/                     # Golang CLI application
│   ├── BUILD.bazel
│   └── main.go
│
├── frontend/                # TypeScript/React web frontend
│   ├── BUILD.bazel
│   ├── src/
│   └── tsconfig.json
│
├── infrastructure/          # Terraform Infrastructure-as-Code
│   ├── aws/                 # AWS resources
│   ├── gcloud/              # Google Cloud resources
│   └── terragrunt.hcl
│
├── py_antilibrary/          # Distributable Python package
│   └── BUILD.bazel
│
├── scala-book-sorting/      # Scala implementation of book sorting
│   ├── BUILD.bazel
│   └── src/
│
├── scraping/                # Python web scraping tools
│   └── nyt_bestsellers/     # NYT bestseller list scraper
│
├── store/                   # Store-related Java code
│   └── layoutsolver/        # Optimization algorithms for store layout
│
├── store-api/               # Spring Boot REST API
│   ├── BUILD.bazel
│   ├── README.md
│   └── src/
│       └── main/java/com/book/store/
│
└── tools/                   # Build tools, linters, and utilities
    ├── build/               # Build-related tools
    ├── dependencies/        # Dependency management scripts
    ├── linting/             # Linting system integration
    ├── python/              # Python tooling
    ├── springboot/          # Spring Boot build rules
    └── typing/              # MyPy type checking configuration
```

## Language Support and Workflows

### 1. Golang Support (`/cli`)

**Rules Used**: `rules_go` and `gazelle`

**Key Files**:
- `WORKSPACE.bazel`: Loads `io_bazel_rules_go` and `bazel_gazelle`
- `3rdparty/go_workspace.bzl`: Defines Go dependencies

**Build Example**:
```bash
bazel build //cli:cli
```

**Run Example**:
```bash
bazel run //cli:cli
```

**Dependency Management**:
- Dependencies declared in `go_workspace.bzl`
- Gazelle can auto-generate BUILD files: `bazel run //:gazelle`

### 2. Java Support (`/store-api`, `/store`)

**Rules Used**: `rules_jvm_external` for Maven dependencies

**Key Files**:
- `WORKSPACE.bazel`: `maven_install` configuration
- `3rdparty/maven_install.json`: Pinned dependency versions

**Build Example**:
```bash
bazel build //store-api:store-api
```

**Spring Boot Application**:
- Uses custom Spring Boot rules from `//tools/springboot`
- Packages as executable JAR with embedded Tomcat

**Dependency Management**:
1. Add dependency to `maven_install.artifacts` list in `WORKSPACE.bazel`
2. Run: `bazel run @maven//:pin` to update `maven_install.json`

### 3. Python Support (`/book_sorting`, `/scraping`, `/py_antilibrary`)

**Rules Used**: `rules_python` and pip integration

**Key Files**:
- `WORKSPACE.bazel`: Python toolchain registration (Python 3.9)
- `3rdparty/requirements.in`: High-level dependencies
- `3rdparty/requirements.txt`: Pinned dependencies (pip-compiled)

**Build Example**:
```bash
bazel build //book_sorting:main
```

**Run Example**:
```bash
bazel run //book_sorting:main
```

**Type Checking with MyPy**:
- Integrated via `bazel-mypy-integration`
- Configuration in `tools/typing/mypy.ini`
- Runs automatically during build

**Dependency Management**:
1. Add package to `3rdparty/requirements.in`
2. Run: `bazel run //3rdparty:requirements.update`
3. This generates updated `requirements.txt`

### 4. Scala Support (`/scala-book-sorting`)

**Rules Used**: `rules_scala`

**Key Files**:
- `WORKSPACE.bazel`: Loads Scala rules and registers toolchains
- `tools/dependencies/jvm_dependencies.yaml`: Scala dependency definitions
- `3rdparty/jvm_workspace.bzl`: Generated by bazel-deps

**Build Example**:
```bash
bazel build //scala-book-sorting:scala-book-sorting
```

**Dependency Management**:
1. Edit `tools/dependencies/jvm_dependencies.yaml`
2. Run: `./tools/update_jvm_dependencies.sh`
3. This uses `bazel-deps` to generate Bazel rules

### 5. TypeScript/NodeJS Support (`/frontend`)

**Rules Used**: `rules_nodejs`

**Key Files**:
- `WORKSPACE.bazel`: Loads NodeJS rules
- `3rdparty/typescript/package.json`: NPM dependencies
- `3rdparty/typescript/yarn.lock`: Lockfile

**Build Example**:
```bash
bazel build //frontend:frontend
```

**Dependency Management**:
1. Update `3rdparty/typescript/package.json`
2. Run: `cd 3rdparty/typescript && yarn install`
3. Commit updated `yarn.lock`

## Understanding Bazel Concepts

### Workspaces
- **WORKSPACE.bazel**: Root file that defines the workspace and external dependencies
- Establishes the root of your Bazel project
- Loads rules (like `rules_go`, `rules_python`) and external repositories

### BUILD Files
- **BUILD.bazel** or **BUILD**: Define build targets in each directory
- Contains rules like `go_binary`, `py_library`, `java_library`
- Specifies dependencies between targets

### Targets
- Format: `//path/to/package:target_name`
- Examples:
  - `//cli:cli` - The cli binary in the cli package
  - `//book_sorting:main` - The main target in book_sorting
  - `//...` - All targets in all packages (recursive)

### Rules
- Define how to build a target
- Examples: `go_binary`, `py_library`, `java_test`, `ts_library`
- Each language has its own set of rules

### Dependencies
- `deps`: Code dependencies (other libraries/packages)
- `data`: Runtime data files
- Example:
  ```python
  py_library(
      name = "mylib",
      srcs = ["mylib.py"],
      deps = ["//other/package:lib"],
  )
  ```

## Build and Test Workflows

### Building Everything
```bash
# Build all targets in the repository
bazel build //...
```

### Testing Everything
```bash
# Run all tests in the repository
bazel test //...
```

### Building Specific Targets
```bash
# Build the CLI
bazel build //cli:cli

# Build the store API
bazel build //store-api:store-api

# Build the frontend
bazel build //frontend:frontend
```

### Running Applications
```bash
# Run the CLI
bazel run //cli:cli

# Run the store API
bazel run //store-api:store-api
```

### Query Capabilities
```bash
# Show all dependencies of a target
bazel query 'deps(//cli:cli)'

# Show reverse dependencies (what depends on this)
bazel query 'rdeps(//..., //book_sorting:main)'

# Show all tests
bazel query 'tests(//...)'
```

## Infrastructure as Code (Terraform)

### AWS Resources (`/infrastructure/aws`)
- Defines cloud resources for deployment
- Managed with Terraform and Terragrunt

### GCloud Resources (`/infrastructure/gcloud`)
- Google Cloud Platform resources
- Also managed with Terraform

**Note**: Infrastructure is managed outside Bazel but lives in the monorepo for version control and atomic updates.

## Deployment and Distribution

### Artifact Distribution
- Deployable artifacts pushed to S3 with commit-hash versioning
- Currently supports `store-api` fat JAR deployment

### Python Package Publishing
- Uses `graknlabs/bazel-distribution`
- Can publish `py_antilibrary` to PyPI
- Configuration in WORKSPACE.bazel

### Build Distribution Example
```bash
# Build deployment artifact
bazel build //store-api:store-api_deploy.jar
```

## Development Workflow

### 1. Local Development Cycle
```bash
# 1. Make code changes
vim cli/main.go

# 2. Build affected targets
bazel build //cli:cli

# 3. Run tests
bazel test //cli:...

# 4. Run locally
bazel run //cli:cli
```

### 2. Linting
```bash
# Lint source code
./tools/linting/lint.sh

# Lint Bazel files
./tools/linting/lint_bzl_files.sh

# Or format BUILD files
bazel run //:buildifier
```

### 3. Continuous Integration (CI)
- **CI Platform**: Buildkite
- Runs on every commit
- Executes: `bazel test //...` and `bazel build //...`
- Results visible at: https://buildkite.com/thundergolfer-inc/the-one-true-bazel-monorepo

### 4. Build Observability
- **Tool**: BuildBuddy.IO
- Every build gets a unique URL: `https://app.buildbuddy.io/invocation/xyz123...`
- Provides:
  - Build timeline and performance analysis
  - Cache hit rates
  - Test results and logs
  - Remote execution metrics

## Key Bazel Files Explained

### .bazelrc
- Configuration file for Bazel
- Sets default flags and options
- Examples of what's configured:
  - Build output settings
  - Remote cache configuration
  - Platform-specific settings

### .bazelversion
- Pins the exact Bazel version
- Ensures all developers use the same version
- Current version: `4.1.0`

### WORKSPACE.bazel
- **Most important file for beginners**
- Defines all external dependencies
- Registers toolchains (Go, Python, Java, etc.)
- Structured in sections by language
- Load external rules and repositories here

## Common Bazel Commands Reference

### Building
```bash
bazel build //cli:cli              # Build specific target
bazel build //cli:...              # Build all targets in cli package
bazel build //...                  # Build everything
bazel build //... --keep_going     # Build all, don't stop on first error
```

### Testing
```bash
bazel test //cli:cli_test          # Run specific test
bazel test //cli:...               # Test everything in cli package
bazel test //...                   # Test everything
bazel test //... --test_output=all # Show full test output
```

### Running
```bash
bazel run //cli:cli                # Run binary
bazel run //cli:cli -- --arg1      # Run with arguments (after --)
```

### Cleaning
```bash
bazel clean                        # Clean build outputs
bazel clean --expunge              # Deep clean (removes all caches)
```

### Querying
```bash
bazel query //...                  # List all targets
bazel query 'deps(//cli:cli)'      # Show dependencies
bazel query 'tests(//...)'         # Show all tests
bazel cquery //cli:cli             # Configured query (shows actual config)
```

## Learning Path: Zero to Hero

### Level 1: Beginner
1. **Understand the basics**:
   - Read this overview document
   - Understand WORKSPACE.bazel structure
   - Learn what BUILD files do
   
2. **Simple experiments**:
   - Run: `bazel build //cli:cli`
   - Run: `bazel test //book_sorting:...`
   - Modify a simple Python or Go file and rebuild

3. **Explore targets**:
   - Use `bazel query //...` to see all targets
   - Pick a target and use `bazel query 'deps(//path/to:target)'`

### Level 2: Intermediate
1. **Add new code**:
   - Create a new Go or Python file
   - Write a BUILD file for it
   - Add dependencies to existing targets

2. **Understand dependency management**:
   - Add a Python dependency via requirements.in
   - Add a Go dependency in go_workspace.bzl
   - Add a Java/Maven dependency

3. **Work with multiple languages**:
   - Make a Python library depend on Scala code
   - Have a Go binary use a Java library

### Level 3: Advanced
1. **Write custom rules**:
   - Study existing custom rules in `/tools`
   - Create your own rule for specific needs

2. **Optimize builds**:
   - Use remote caching
   - Profile builds with BuildBuddy
   - Understand action caching and incremental builds

3. **Advanced queries**:
   - Use `cquery` for configuration-dependent queries
   - Use `aquery` for action-level analysis
   - Create query scripts for automation

### Level 4: Hero
1. **Scale the monorepo**:
   - Add new services/applications
   - Manage cross-language dependencies
   - Optimize CI/CD pipeline

2. **Custom toolchain integration**:
   - Integrate new build tools
   - Create language-specific toolchains
   - Manage different compiler versions

3. **Contribute back**:
   - Document your learnings
   - Share custom rules
   - Help others in the community

## Troubleshooting Common Issues

### "No such package" error
- Check that BUILD file exists in the package
- Verify WORKSPACE.bazel has required dependencies loaded

### "Target not found" error
- Check BUILD file has the target defined
- Verify target name matches exactly (case-sensitive)

### Dependency resolution errors
- For Python: Re-run `bazel run //3rdparty:requirements.update`
- For Java: Re-run `bazel run @maven//:pin`
- For Go: Check `3rdparty/go_workspace.bzl`

### Build cache issues
```bash
bazel clean
bazel build //...
```

### Version mismatches
- Check `.bazelversion` file
- Ensure you have the correct Bazel version installed

## Additional Resources

### Official Documentation
- [Bazel Documentation](https://docs.bazel.build/)
- [Bazel Best Practices](https://docs.bazel.build/versions/master/best-practices.html)
- [Bazel Glossary](https://docs.bazel.build/versions/master/glossary.html)

### Language-Specific Rules
- [rules_go](https://github.com/bazelbuild/rules_go)
- [rules_python](https://github.com/bazelbuild/rules_python)
- [rules_jvm_external](https://github.com/bazelbuild/rules_jvm_external)
- [rules_scala](https://github.com/bazelbuild/rules_scala)
- [rules_nodejs](https://github.com/bazelbuild/rules_nodejs)

### Community Resources
- [Awesome Bazel](https://github.com/jin/awesome-bazel) - Curated list of Bazel resources
- [Awesome Monorepo](https://github.com/korfuri/awesome-monorepo) - Monorepo resources
- [BazelCon](https://conf.bazel.build/) - Annual Bazel conference

### Articles and Talks
- [Why Google Stores Billions of Lines in a Single Repository](http://delivery.acm.org/10.1145/2860000/2854146/p78-potvin.pdf)
- [Monorepos: Please Do!](https://medium.com/@adamhjk/monorepo-please-do-3657e08a4b70)
- [Advantages of Monorepos](https://danluu.com/monorepo/)

## Next Steps

1. **Explore the codebase**: 
   - Look at different BUILD files
   - Understand how targets are defined
   - See how dependencies are declared

2. **Try building components**:
   - `bazel build //cli:cli`
   - `bazel build //store-api:store-api`
   - `bazel test //...`

3. **Read language-specific code**:
   - Pick a language you're comfortable with
   - Read the implementation
   - Understand how it integrates with Bazel

4. **Make small changes**:
   - Modify a simple function
   - Add a new file
   - Create a new BUILD target

5. **Dive deeper into WORKSPACE.bazel**:
   - Understand each section
   - See how external dependencies are loaded
   - Learn about toolchain registration

## Conclusion

This Example Bazel Monorepo provides a comprehensive, real-world demonstration of building and managing a multi-language project with Bazel. It's designed as a learning resource that grows with you from beginner to expert level.

The key to mastering Bazel is:
1. **Start small**: Build and run existing targets
2. **Experiment**: Make changes and see what happens
3. **Read BUILD files**: They're the blueprint of the build
4. **Use queries**: Understand the dependency graph
5. **Iterate**: Small changes, frequent builds

Remember: Bazel's learning curve is steep, but the benefits of fast, reliable, reproducible builds make it worthwhile for large projects!

Happy Building! 🚀📦
