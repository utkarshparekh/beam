<!--
    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.
-->

# AGENTS.md — AI Coding Agent Guide for Apache Beam

This guide helps AI coding agents understand the Apache Beam repository structure, development workflow, and common patterns.

## Project Overview

Apache Beam is a unified model for defining both batch and streaming data-parallel processing pipelines. See [README.md](README.md) and https://beam.apache.org/get-started/beam-overview/.

### The Beam Model

The Beam programming model evolved from Google's MapReduce, FlumeJava, and Millwheel projects, originally known as the "Dataflow Model".

**Key concepts:**

- `PCollection` — represents a dataset (bounded or unbounded)
- `PTransform` — represents a data processing operation
- `Pipeline` — the overall data processing workflow
- `PipelineRunner` — executes the pipeline on a distributed backend

### SDKs

This repository contains three language-specific SDKs:

- **Java SDK** — `sdks/java/`
- **Python SDK** — `sdks/python/`
- **Go SDK** — `sdks/go/`

### Runners

Beam pipelines can execute on multiple distributed processing backends:

- **DirectRunner** — local execution for testing
- **PrismRunner** — portable local runner
- **DataflowRunner** — Google Cloud Dataflow
- **FlinkRunner** — Apache Flink
- **SparkRunner** — Apache Spark
- **JetRunner** — Hazelcast Jet
- **Twister2Runner** — Twister2

## Repository Layout

```
apache/beam/
├── .github/              # GitHub workflows and CI configuration
├── .test-infra/          # Test infrastructure and Jenkins configs
├── buildSrc/             # Gradle build scripts and plugins
├── contributor-docs/     # Contributor documentation
├── dev-support/docker/   # Docker development environments
├── examples/             # Example pipelines (Java, Python, Go)
├── infra/                # Infrastructure as code
├── it/                   # Integration tests
├── learning/             # Learning materials
├── model/                # Beam model definitions (protobuf)
├── playground/           # Interactive Beam Playground
├── release/              # Release scripts and tools
├── runners/              # Runner implementations
├── scripts/              # Utility scripts
├── sdks/                 # Language-specific SDKs
│   ├── java/             # Java SDK
│   ├── python/           # Python SDK
│   └── go/               # Go SDK
├── vendor/               # Vendored dependencies
└── website/              # beam.apache.org website source
```

### SDK directories

- `sdks/java/core/` — core Java SDK
- `sdks/java/io/` — Java I/O connectors
- `sdks/java/extensions/` — extensions (SQL, ML, etc.)
- `sdks/python/apache_beam/` — Python SDK core
- `sdks/go/pkg/beam/` — Go SDK core

### Runners directories

- `runners/direct-java/` — Direct runner
- `runners/flink/` — Flink runner (multiple versions)
- `runners/spark/` — Spark runner
- `runners/google-cloud-dataflow-java/` — Dataflow runner

### Model directory

- `model/pipeline/` — pipeline model protobuf definitions
- `model/fn-execution/` — function execution model
- `model/job-management/` — job management model

## Development Setup Prerequisites

### Required tools

See [CONTRIBUTING.md](CONTRIBUTING.md) for the authoritative list.

**Java:**

- JDK 11 (preferred; 8, 17, and 21 also supported)
- Set `JAVA_HOME`

**Python:**

- Python 3.10, 3.11, 3.12, 3.13 (see `python_versions` in `gradle.properties`)
- `pyenv` recommended for managing multiple versions
- Virtual environment tools (`virtualenv`, `venv`, or `pyenv-virtualenv`)

**Go:**

- Latest Go 1.x
- Clone to `$GOPATH/src/github.com/apache/beam` (recommended)

**Build tools:**

- Gradle (via included `./gradlew` wrapper)
- Docker (for containerized builds and testing)

**Optional:**

- `tox` (Python testing)
- `pre-commit` (commit hooks)

### Setup options

**Automated local setup (Linux/macOS):**

```bash
./local-env-setup.sh
```

**Docker-based development environment:**

```bash
./start-build-env.sh
```

### Verify setup

```bash
./gradlew :checkSetup
```

This validates Go, Java, and Python environments.

## Build and Test Commands

Beam uses Gradle as the build system. Run `./gradlew` from the repository root.

### Common commands

```bash
# Validate environment
./gradlew :checkSetup

# Build all
./gradlew build

# Run tests
./gradlew test

# Run a specific Java test
./gradlew :examples:java:test --tests org.apache.beam.examples.subprocess.ExampleEchoPipelineTest --info

# Continue on errors
./gradlew compileJava --continue
```

### Quick smoke tests by language

```bash
# Java
./gradlew :examples:java:wordCount

# Python
./gradlew :sdks:python:wordCount

# Go
export GOLANG_PROTOBUF_REGISTRATION_CONFLICT=ignore
./gradlew :sdks:go:examples:wordCount
```

### Pre-commit checks

Before creating a PR, run relevant checks locally:

```bash
./gradlew spotlessApply && \
./gradlew -PenableCheckerFramework=true \
  checkstyleMain checkstyleTest javadoc spotbugsMain \
  compileJava compileTestJava
```

**Language-specific Gradle pre-commit tasks:**

| Language | Tasks |
|----------|-------|
| Java | `javaPreCommit`, `javaioPreCommit`, `sqlPreCommit` |
| Python | `pythonPreCommit`, `pythonLintPreCommit`, `pythonFormatterPreCommit`, `pythonDocsPreCommit` |
| Go | `goPreCommit`, `goPortablePreCommit`, `goPrismPreCommit` |
| Website | `websitePreCommit` |

### Python-specific commands

```bash
# From sdks/python/
tox -e py310
python -m pytest apache_beam/...
yapf -i --recursive apache_beam/
./gradlew :sdks:python:test-suites:tox:pycommon:linter
```

### Go-specific commands

```bash
./gradlew :sdks:go:goTest
./gradlew :sdks:go:goBuild

# From sdks/go/test/
./run_validatesrunner_tests.sh
```

## Contribution Workflow

### Finding work

1. Browse issues: https://github.com/apache/beam/issues
2. Look for the "good first issue" label
3. Comment `.take-issue` to assign yourself
4. Comment `.free-issue` to unassign yourself
5. Comment `.close-issue` when done

### Pull request process

1. Fork and clone the repo; add `upstream` remote
2. Create a feature branch
3. Make changes following coding standards below
4. Add unit tests
5. Include the Apache license header in every new source file
6. Run pre-commit checks locally
7. Commit with descriptive messages
8. Open a PR linking to the issue
9. Add fixup commits during review; squash only after review is complete
10. Re-trigger unrelated CI failures with `retest this please`

See [CONTRIBUTING.md](CONTRIBUTING.md) and [contributor-docs/code-change-guide.md](contributor-docs/code-change-guide.md) for details.

### Pre-commit hooks

```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

Configured hooks (see `.pre-commit-config.yaml`):

- `yapf` — Python formatting (`sdks/python/apache_beam/`)
- `pylint` — Python linting (`sdks/python/apache_beam/`)

### License headers

Every source file must include the Apache License header. New dependencies must have licenses [compatible with Apache](https://www.apache.org/legal/resolved.html#criteria).

### Release schedule

Minor releases ship every 6 weeks. Changes must land on `master` before the release branch is cut.

## Per-Language Coding Conventions

### Java

- Use `spotlessApply` for formatting
- Enable Checker Framework: `-PenableCheckerFramework=true`
- Run checkstyle: `checkstyleMain checkstyleTest`
- Follow the [PTransform Style Guide](https://beam.apache.org/contribute/ptransform-style-guide/)
- Make transform classes immutable (`private final` fields)
- Use AutoValue for builders; fluent `withBlah()` methods
- Validate parameters in `.withBlah()` and `.expand()`
- Test with `TestPipeline`, `PAssert`, `DoFnTester`, `CombineFnTester`
- Use SLF4J 1.x (not 2.x)
- Wiki: https://cwiki.apache.org/confluence/display/BEAM/Java+Tips

### Python

- Format with `yapf` (version in `sdks/python/tox.ini`)
- Lint with `pylint` (see `.pre-commit-config.yaml`)
- Install from sources: `pip install -e ".[test,gcp]"`
- Generate protos: `python gen_protos.py`
- Test with `pytest` and `tox`
- Wiki: https://cwiki.apache.org/confluence/display/BEAM/Python+Tips

### Go

- Clone to `$GOPATH/src/github.com/apache/beam`
- Update dependencies: `go get -u ./...`
- Integration tests must call `integration.CheckFilters(t)`
- `TestMain` must call `ptest.Main(m)` for integration tests
- See `sdks/go/README.md` and https://cwiki.apache.org/confluence/display/BEAM/Go+Tips

## Common Agent Task Entry Points

| Task | Where to start |
|------|----------------|
| Java I/O connector | `sdks/java/io/<connector-name>/` |
| Python I/O connector | `sdks/python/apache_beam/io/` |
| Go I/O connector | `sdks/go/pkg/beam/io/` |
| New PTransform | [PTransform Style Guide](https://beam.apache.org/contribute/ptransform-style-guide/) |
| Runner changes | `runners/` + [Runner Guide](https://beam.apache.org/contribute/runner-guide/) |
| Website docs | `website/www/site/content/` → `./gradlew websitePreCommit` |
| Model/protobuf changes | `model/` |
| CI/workflows | `.github/workflows/` (see `.github/workflows/README.md`) |

## Pitfalls to Avoid

### General

- Do not open PRs without a linked issue
- Do not make mega-changes without prior discussion on dev@
- Do not squash commits during active review
- Do not skip license headers on new files
- Do not force-push to `master`
- Do not mask data loss — fail rather than silently drop data
- Do not log sensitive data or per-element INFO logs

### Java

- Do not use SLF4J 2.x
- Do not make transforms mutable
- Do not use `SerializableCoder` in performance-critical paths without reason
- Refresh Gradle cache with `--rerun-tasks` if builds fail after branch switches

### Python

- Do not skip virtualenv activation before local testing
- Do not forget `python gen_protos.py` after proto changes
- Do not update pinned dependencies before a release branch cut

### Go

- Do not clone outside `$GOPATH/src/github.com/apache/beam`
- Do not skip `integration.CheckFilters(t)` in integration tests

## Additional Resources

- [CONTRIBUTING.md](CONTRIBUTING.md)
- [contributor-docs/](contributor-docs/)
- [Code Change Guide](contributor-docs/code-change-guide.md)
- https://beam.apache.org/contribute/
- [PTransform Style Guide](https://beam.apache.org/contribute/ptransform-style-guide/)
- [Runner Guide](https://beam.apache.org/contribute/runner-guide/)
- [Design docs](https://s.apache.org/beam-design-docs)
- [Jenkins trigger phrases](.test-infra/jenkins/README.md)
- Community: dev@beam.apache.org, #beam on ASF Slack
