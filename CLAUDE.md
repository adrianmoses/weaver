# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Weaver is a Python project scaffolding tool that generates projects from templates using cookiecutter. It provides a CLI interface for creating new projects with different templates, including a specialized "knowledge-graph" template for entity resolution projects.

## Architecture

- **Entry Point**: `src/cli.py` contains the main Typer CLI application
- **Commands**: Currently implements a `new` command for project generation
- **Templates**: Supports multiple templates:
  - `knowledge-graph`: Uses cookiecutter with the entity-resolution-cookiecutter template from GitHub
  - `custom`: Uses internal project generation logic
  - `default`: Default template option

## Development Commands

### Package Management
- Uses `uv` for dependency management (uv.lock present)
- Install dependencies: `uv sync`
- Add new dependency: `uv add <package>`

### Running the CLI
- Main entry point: `python -m src.cli`
- Run specific command: `python -m src.cli new --name <project_name> --template <template_type>`

### Project Structure
```
src/
├── __init__.py     # Package initialization
└── cli.py          # Main CLI application with Typer
```

## Key Dependencies

- **typer**: CLI framework for the command interface
- **cookiecutter**: Template engine for project generation

## AIDER Integration

This project follows AIDER (Architecture Informed Development with Evidence-based Reporting) methodology:

- Pattern tags should use format: `# AIDER:PAT-{CATEGORY}-{NUMBER}`
- Run `aider pcs` to measure Pattern Compliance Score
- Configuration in `.aider/config.yaml` (if present)
- Documentation in `docs/AIDER_PROJECT.md`

## Template System

The knowledge-graph template is configured with these defaults:
- Data sources: web_scraping, api
- Storage backend: postgresql  
- Pipeline orchestrator: prefect

Target repository: https://github.com/adrianmoses/entity-resolution-cookiecutter