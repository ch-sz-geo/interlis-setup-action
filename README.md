# interlis-setup-action

GitHub Actions for setting up INTERLIS tools in your CI/CD workflows.

This repository provides two GitHub Actions for automating the download and installation of INTERLIS tools:
- **Setup iliCompiler** - Installs ili2c (INTERLIS to Code Compiler)
- **Setup ilimanager** - Installs ilimanager (INTERLIS Repository Index Tool)

## Table of Contents

- [Overview](#overview)
- [Available Actions](#available-actions)
  - [Setup iliCompiler](#setup-ilicompiler)
  - [Setup ilimanager](#setup-ilimanager)
- [Usage](#usage)
- [Examples](#examples)
- [Requirements](#requirements)
- [Outputs](#outputs)

## Overview

INTERLIS is a language and methodology for describing data models for the exchange of datasets. This action repository simplifies the setup of INTERLIS tools in GitHub Actions workflows, allowing you to:

- Automatically download and install specific versions of INTERLIS tools
- Use the latest available version or pin to a specific version
- Configure custom installation directories
- Access jar file paths for direct tool invocation

## Available Actions

### Setup iliCompiler

Configurable GitHub Action that downloads and installs **ili2c** (INTERLIS compiler).

**Action Path:** `./setup-iliCompiler`

**Description:** Downloads and installs the INTERLIS to Code Compiler

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | iliCompiler version to install (e.g., `1.0.0`). Use `latest` for the newest version. | No | `latest` |
| `path` | Directory path where the compiler should be installed | No | `ili2c` |

#### Outputs

| Output | Description |
|--------|-------------|
| `install-path` | Directory where ili2c was installed |
| `jar-path` | Full path to the ili2c jar file |

#### How It Works

1. Sets up Java 11 (required runtime)
2. Resolves the requested version (fetches latest from release JSON if needed)
3. Downloads the ili2c release from `https://downloads.interlis.ch/ili2c/`
4. Extracts the archive to the specified directory
5. Locates and outputs the jar file path
6. Verifies the installation by running `--help`

---

### Setup ilimanager

Configurable GitHub Action that downloads and installs **ilimanager** (INTERLIS repository index tool).

**Action Path:** `./setup-ilimanager`

**Description:** Downloads and installs ilimanager for managing INTERLIS repositories

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | ilimanager version to install (e.g., `1.0.0`). Use `latest` for the newest version. | No | `latest` |
| `path` | Directory path where ilimanager should be installed | No | `ilimanager` |

#### Outputs

| Output | Description |
|--------|-------------|
| `install-path` | Directory where ilimanager was installed |
| `jar-path` | Full path to the ilimanager jar file |

#### How It Works

1. Sets up Java 11 (required runtime)
2. Resolves the requested version (fetches latest from release JSON if needed)
3. Downloads the ilimanager release from `https://downloads.interlis.ch/ilimanager/`
4. Extracts the archive to the specified directory
5. Renames the versioned jar file to `ilimanager.jar`
6. Outputs the installation and jar paths
7. Verifies the installation by running `--help`

---

## Usage

### Basic Usage

Add one of these actions to your GitHub Actions workflow:

#### Using Setup iliCompiler (Latest Version)

```yaml
- uses: ./.github/actions/setup-iliCompiler@v1
  with:
    version: 'latest'
    path: './tools/ili2c'
```

#### Using Setup ilimanager (Latest Version)

```yaml
- uses: ./.github/actions/setup-ilimanager@v1
  with:
    version: 'latest'
    path: './tools/ilimanager'
```

#### Using Specific Versions

```yaml
- uses: ./.github/actions/setup-iliCompiler@v1
  with:
    version: '4.7.0'
    path: './tools/ili2c'

- uses: ./.github/actions/setup-ilimanager@v1
  with:
    version: '1.0.0'
    path: './tools/ilimanager'
```

---

## Examples

### Example 1: INTERLIS Model Validation Workflow

```yaml
name: Validate INTERLIS Models

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup ili2c
        id: setup-ili2c
        uses: ./.github/actions/setup-iliCompiler@v1
        with:
          version: 'latest'
          path: './tools/ili2c'
      
      - name: Validate INTERLIS models
        run: |
          java -jar ${{ steps.setup-ili2c.outputs.jar-path }} \
            --models models/ \
            --check
```

### Example 2: Combined Setup Workflow

```yaml
name: Setup INTERLIS Environment

on: [workflow_dispatch]

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup ili2c
        id: setup-compiler
        uses: ./.github/actions/setup-iliCompiler@v1
        with:
          version: 'latest'
          path: './tools/ili2c'
      
      - name: Setup ilimanager
        id: setup-manager
        uses: ./.github/actions/setup-ilimanager@v1
        with:
          version: 'latest'
          path: './tools/ilimanager'
      
      - name: Display installation paths
        run: |
          echo "ili2c installed to: ${{ steps.setup-compiler.outputs.install-path }}"
          echo "ili2c jar: ${{ steps.setup-compiler.outputs.jar-path }}"
          echo "ilimanager installed to: ${{ steps.setup-manager.outputs.install-path }}"
          echo "ilimanager jar: ${{ steps.setup-manager.outputs.jar-path }}"
```

### Example 3: Version-Pinned Workflow

```yaml
name: Build with Fixed Versions

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup ili2c (v4.7.0)
        id: compiler
        uses: ./.github/actions/setup-iliCompiler@v1
        with:
          version: '4.7.0'
      
      - name: Setup ilimanager (v1.13.2)
        id: manager
        uses: ./.github/actions/setup-ilimanager@v1
        with:
          version: '1.13.2'
      
      - name: Generate code from INTERLIS models
        run: |
          java -jar ${{ steps.compiler.outputs.jar-path }} \
            --models models/RealEstateRegister.ili \
            --out_java src/generated
```

---

## Requirements

- **GitHub Actions Runner:** Ubuntu, macOS, or Windows
- **Java:** Version 11 or higher (automatically installed by the actions)
- **Internet Access:** Required to download tool releases from `downloads.interlis.ch`

---

## Outputs

Both actions provide the following outputs that can be used in subsequent steps:

### `install-path`
The directory where the tool was installed. Useful for referencing configuration files or additional resources.

```yaml
${{ steps.setup-ili2c.outputs.install-path }}  # e.g., './tools/ili2c'
```

### `jar-path`
The full path to the main jar file. Use this to invoke the tool directly.

```yaml
java -jar ${{ steps.setup-ili2c.outputs.jar-path }}
```

---

## File Structure

```
interlis-setup-action/
├── README.md                     # This file
├── setup-iliCompiler/
│   └── action.yml               # ili2c installation action
└── setup-ilimanager/
    └── action.yml               # ilimanager installation action
```

---

## Resources

- [INTERLIS Official Website](https://www.interlis.ch/)
- [ili2c Documentation](https://downloads.interlis.ch/ili2c/)
- [ilimanager Documentation](https://downloads.interlis.ch/ilimanager/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

## License

[Add your license information here]

## Contributing

[D. Wehrli](https://github.com/WehrliDavid)
