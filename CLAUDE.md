# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Weaver is a Python project scaffolding tool that generates entity-resolution projects from a bundled cookiecutter template. It provides a CLI interface for creating new projects with different configurations optimized for various use cases (knowledge graphs, search engines, basic projects).

## Architecture

- **Entry Point**: `src/weaver/cli.py` contains the main Typer CLI application
- **Commands**:
  - `create`: Generate a new project from a template
  - `list-templates`: List available project templates
  - `templates`: Show template details
  - `version`: Show weaver version
- **Templates**: All templates use the bundled `entity-resolution` cookiecutter template located in `src/weaver/templates/entity-resolution/`
  - `advanced-search`: Hybrid sparse and vector search
  - `knowledge-graph`: AI company knowledge graph with Neo4j
  - `news-analyzer`: News aggregator with bias analysis
  - `basic`: Basic entity-relationship project

## Development Commands

### Package Management
- Uses `uv` for dependency management (uv.lock present)
- Install dependencies: `uv sync`
- Add new dependency: `uv add <package>`

### Running the CLI
- Main entry point: `uv run weaver <command>`
- Create project: `uv run weaver create --template <template-name> --name <project_name>`
- List templates: `uv run weaver list-templates`

### Project Structure
```
src/weaver/
├── __init__.py
├── cli.py                      # Main CLI application with Typer
├── config.py                   # Configuration classes
├── generators/
│   ├── __init__.py
│   ├── base.py                 # Base generator class
│   └── cookiecutter.py         # Cookiecutter generator implementation
└── templates/
    └── entity-resolution/      # Bundled cookiecutter template
        ├── cookiecutter.json   # Template configuration
        ├── hooks/              # Pre/post generation hooks
        │   ├── pre_gen_project.py
        │   └── post_gen_project.py
        └── {{cookiecutter.project_slug}}/  # Project template
```

## Key Dependencies

- **typer**: CLI framework for the command interface
- **cookiecutter**: Template engine for project generation
- **rich**: Terminal formatting and output

## Template System

The bundled entity-resolution template supports:

### Configuration Options
- **Project types**: basic, search_engine, knowledge_graph
- **Databases**: postgresql, sqlite, neo4j, mongodb
- **Orchestrators**: prefect, dagster, simple
- **Search engines**: none, vector_hybrid, elasticsearch
- **API frameworks**: fastapi, flask, django, none
- **Features**: vector search, NLP, web scraping, API scraping, Docker, pytest
- **Ontology generation**: with Anthropic or OpenAI LLM providers

### Template Presets
- `advanced-search`: PostgreSQL + vector search + Prefect
- `knowledge-graph`: Neo4j + web scraping + Prefect
- `news-analyzer`: PostgreSQL + NLP + vector search + Elasticsearch
- `basic`: SQLite + simple orchestrator

## Config-Template Consistency

The `Config` class in `src/weaver/config.py` matches the fields in `cookiecutter.json`. When adding new fields to the template:
1. Update `cookiecutter.json` in the template
2. Add corresponding field to `Config` class
3. Update `_prepare_cookiecutter_context()` in `cookiecutter.py`
4. Update `validate_config()` for any validation rules