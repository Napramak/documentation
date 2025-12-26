# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the documentation repository for the **Napramak Logistics Platform**. It uses MkDocs with the Material theme to
generate static documentation that is automatically deployed to GitHub Pages at https://docs.napramak.io/.

## Core Technologies

- **MkDocs**: Static site generator for documentation
- **Material for MkDocs**: Material Design theme with enhanced features
- **Task**: Task runner for build automation (Taskfile.yaml)
- **uv**: Python package manager for dependency management
- **Python 3.11+**: Required runtime

## Essential Commands

### Development Workflow

```bash
# Start local dev server with live reload at http://127.0.0.1:8000
task docs:serve

# Build static site (output to site/ directory)
task docs:build

# Clean generated files
task docs:clean
```

### Setup

```bash
# Install all development tools (Homebrew, Python, uv, etc.)
task dev:install

# List all available tasks
task
```

## Architecture

### Build Process

The documentation build follows this sequence (see `docs:build` and `docs:serve` tasks):

1. **Preparation phase** (`docs:prepare`): Downloads the OpenAPI specification
   from https://api.demo.napramak.ru/api-json and saves it as `docs/api/swagger.json`
2. **MkDocs build**: Processes Markdown files with Jinja2 templating, applies Material theme, and generates static HTML

### Template System

Documentation uses **mkdocs-macros-plugin** for Jinja2 templating:

- Template variables are defined in `docs/project_info.yml`
- Variables can be used in Markdown files: `{{ project.name }}` → "Napramak"
- Available variables: `project.name`, `project.required full_name`

### Swagger UI Integration

The REST API documentation (`docs/api/rest_api.md`) embeds an interactive Swagger UI using:

```markdown
<swagger-ui src="./swagger.json"/>
```

The `swagger.json` file is automatically downloaded from the demo API during the build preparation phase, so it should
never be committed to the repository.

### MkDocs Configuration

Key configuration in `mkdocs.yml`:

- **Theme features**: Navigation tabs, code copy buttons, dark/light mode toggle
- **Markdown extensions**: Mermaid diagrams, admonitions, syntax highlighting, emoji support
- **Plugins**: search, autorefs, macros, swagger-ui-tag, minify (for production optimization)
- **Validation**: Warnings for broken links and missing files (not strict errors)

## CI/CD Pipeline

GitHub Actions workflow (`.github/workflows/docs.yml`):

- **Build job**: Runs on all pushes and PRs, validates documentation can build
- **Deploy job**: Only on `master` branch pushes, deploys to GitHub Pages
- Uses Task runner for builds: `task docs:build`

## Multi-Language Support

The documentation supports **English**, **Russian**, and **Polish** using the `mkdocs-static-i18n` plugin:

- **Default language**: English (`en`)
- **Additional languages**: Russian (`ru`), Polish (`pl`)
- **Structure**: Suffix-based (e.g., `index.md` for English, `index.ru.md` for Russian, `index.pl.md` for Polish)
- **Language switcher**: Automatically appears in the UI when multiple languages are available

### How i18n Works

The i18n plugin:

1. Builds separate documentation versions: English in `/`, Russian in `/ru/`, and Polish in `/pl/`
2. Automatically reconfigures the Material theme and search for each language
3. Provides language-specific navigation translations (configured in `mkdocs.yml`)
4. Falls back to English content if translations are missing

## Important Notes

### API Documentation Dependencies

The `docs:prepare` task **must** run before building/serving docs because it downloads the required `swagger.json` file.
Both `docs:build` and `docs:serve` automatically call `docs:prepare`, so running them directly is sufficient.

### Python Environment

The project uses `uv` for dependency management with a virtual environment in `.venv/`. Dependencies are defined in
`pyproject.toml` and locked in `uv.lock`.

### Documentation Structure

```
docs/
├── index.md              # English homepage (uses Jinja2 templates)
├── index.ru.md           # Russian homepage
├── index.pl.md           # Polish homepage
├── api/
│   ├── rest_api.md       # English API docs (embeds Swagger UI)
│   ├── rest_api.ru.md    # Russian API docs
│   ├── rest_api.pl.md    # Polish API docs
│   └── swagger.json      # Auto-downloaded, do not commit
└── project_info.yml      # Template variables (shared across languages)
```

### Editing Documentation

When adding new pages:

1. Create Markdown files in `docs/` directory (e.g., `new-page.md`)
2. Create Russian version with `.ru.md` suffix (e.g., `new-page.ru.md`)
3. Create Polish version with `.pl.md` suffix (e.g., `new-page.pl.md`)
4. Add navigation entries to `nav` section in `mkdocs.yml`
5. Add navigation translations to both `i18n.languages[ru].nav_translations` and `i18n.languages[pl].nav_translations` in `mkdocs.yml`
6. Use Jinja2 variables from `project_info.yml` for consistency
7. Test locally with `task docs:serve` before committing
