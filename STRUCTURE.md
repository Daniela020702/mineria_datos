# Minería de Datos - ITAM 2025

Course materials for Data Mining at ITAM.

## Quick Start

### Prerequisites

- [uv](https://docs.astral.sh/uv/) - Python package manager
- [Just](https://just.systems/) - Command runner
- [Quarto](https://quarto.org/) - Scientific publishing system
- Python 3.13+

### Setup

```bash
# Install dependencies
just setup

# Start preview server
just preview
```

## Available Commands

Run `just` to see all available commands:

```bash
just install        # Install dependencies
just preview        # Preview the book locally
just render         # Build production site
just format         # Format code
just lint           # Check code quality
just fix            # Fix linting issues automatically
just lab            # Launch Jupyter Lab
just notebook       # Launch Jupyter Notebook
just clean          # Clean generated files
just pre-commit     # Run pre-commit hooks
just build          # Full build pipeline (format, lint, render)
```

## Development

### With VS Code DevContainers

1. Install "Dev Containers" extension
2. Open folder in VS Code
3. "Reopen in Container"
4. Everything is pre-configured!

### With GitHub Codespaces

Click "Code" → "Create codespace on main"

### Local Development

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install Just (if not already installed)
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to ~/.local/bin

# Install Quarto
# macOS
brew install quarto

# Linux
wget https://github.com/quarto-dev/quarto-cli/releases/download/v1.6.39/quarto-1.6.39-linux-amd64.deb
sudo dpkg -i quarto-1.6.39-linux-amd64.deb

# Clone and setup
git clone https://github.com/YOUR_USERNAME/mineria_datos_itam.git
cd mineria_datos_itam
just setup
```

## Project Structure

```
.
├── .github/          # CI/CD workflows
│   └── workflows/
│       └── quarto-publish.yml
├── .devcontainer/    # VS Code devcontainer configuration
├── chapters/         # Quarto book content (.qmd files)
│   ├── _quarto.yml   # Quarto configuration
│   ├── index.qmd     # Book homepage
│   ├── 00-*.qmd      # Requirements and setup
│   ├── 01-*.qmd      # Introduction
│   ├── 02-*.qmd      # Principles
│   ├── 03-*.qmd      # Linear regression
│   ├── 04-*.qmd      # Classification
│   ├── 05-*.qmd      # Trees
│   └── 06-*.qmd      # Boosting
├── notebooks/        # Jupyter notebooks (examples & exercises)
│   ├── introduccion.ipynb
│   ├── regresion_lineal.ipynb
│   ├── california_housing_case_study.ipynb
│   └── ...
├── docs/             # Generated site (auto-built by CI)
├── tareas/           # Student assignments
├── pyproject.toml    # Python dependencies
├── uv.lock           # Locked dependencies
├── justfile          # Command definitions
└── .pre-commit-config.yaml  # Pre-commit hooks
```

## Architecture Overview

### Design Decisions

**uv vs conda/mamba**: 10-100x faster dependency resolution, simpler toolchain, PyPI-native. All dependencies are specified in `pyproject.toml` and locked in `uv.lock` for reproducible builds. Migrated from conda in January 2025 for better performance and modern Python ecosystem integration.

**Just vs Makefile**: Modern syntax with better error messages, cross-platform compatibility (including Windows), and more intuitive command definitions. See `justfile` for all available commands and their implementation.

**Direct notebook references**: Notebooks in `notebooks/` are referenced directly in `chapters/_quarto.yml` via relative paths like `../notebooks/*.ipynb`. No symlinks needed - this is cleaner, more portable, and the Quarto-native way of including notebooks in books.

### Build Pipeline

The course uses a three-stage build and deployment pipeline:

1. **Local Development**:
   ```
   just preview → Quarto renders chapters/ → Live preview on localhost:4200
   ```
   Changes auto-reload as you edit `.qmd` and `.ipynb` files.

2. **CI/CD (GitHub Actions)**:
   ```
   Push to main → GitHub Actions triggered →
   uv installs dependencies → Quarto renders →
   Deploys to GitHub Pages
   ```
   Workflow file: `.github/workflows/quarto-publish.yml`

3. **Execution Strategy**:
   - `freeze: auto` in `_quarto.yml` caches notebook outputs in `chapters/_freeze/`
   - Re-executes only when source code changes
   - Dramatically speeds up repeated renders
   - First render: ~2-3 minutes, subsequent renders: ~30 seconds

### Notebook Integration

The Quarto book in `chapters/` seamlessly includes Jupyter notebooks from the `notebooks/` directory:

```yaml
# chapters/_quarto.yml example
book:
  chapters:
    - 03-regresion_lineal.qmd                          # Regular Quarto chapter
    - ../notebooks/violaciones_supuestos_regresion.ipynb  # Notebook included directly
    - ../notebooks/analisis_advertising_dataset.ipynb     # Another notebook
```

**Key points:**
- No symlinks are used (unlike the previous conda-based setup)
- Notebooks remain in their own directory for better organization
- Quarto follows relative paths and renders notebooks natively
- Notebook outputs are preserved and cached with `freeze: auto`
- Students can work on notebooks independently using `just lab`

### GitHub Pages Configuration

For automated deployment to work, configure in your GitHub repository:

**Required Settings** (Settings → Pages):
- **Source**: GitHub Actions (NOT "Deploy from a branch")
- This enables the workflow to deploy directly

**Workflow Permissions** (Settings → Actions → General):
- **Workflow permissions**: "Read and write permissions"
- ✓ "Allow GitHub Actions to create and approve pull requests"

**First Deployment:**
- Takes ~3-5 minutes (downloads all dependencies)
- Subsequent builds: ~1-2 minutes (uses caching)
- Site will be available at: `https://YOUR_USERNAME.github.io/mineria_datos_itam/`

**Monitoring:**
- Check Actions tab for workflow status
- Green checkmark = successful deployment
- Red X = check logs for errors

### Troubleshooting Common Issues

**`just: command not found`**
```bash
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash -s -- --to ~/.local/bin
export PATH="$HOME/.local/bin:$PATH"
```

**`uv sync` fails**
```bash
# Clean cache and retry
uv cache clean
uv sync --locked --reinstall
```

**Notebooks don't render in Quarto**
```bash
# Ensure Jupyter kernel is available
uv run jupyter kernelspec list
# Clear Quarto cache
just clean
# Try rendering again
just render
```

**GitHub Actions deployment fails**
- Check that GitHub Pages source is set to "GitHub Actions"
- Verify workflow has write permissions (Settings → Actions → General)
- Check Actions tab for specific error messages in logs

## Course Content

1. **Requerimientos y Computación** - Prerequisites and setup
2. **Introducción** - Introduction to data mining
3. **Principios** - Fundamental principles
4. **Regresión Lineal** - Linear regression concepts and applications
5. **Clasificación** - Classification techniques
6. **Árboles de Decisión** - Decision trees and ensemble methods
7. **Boosting** - Advanced boosting algorithms (XGBoost, LightGBM, CatBoost)

## Contributing

1. Create a feature branch
2. Make changes
3. Run `just format` and `just lint`
4. Commit changes (pre-commit hooks will run automatically)
5. Push and create PR

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## Technology Stack

- **Python 3.13** - Latest stable Python
- **uv** - Fast Python package manager (replaces conda/pip)
- **Quarto** - Scientific publishing system
- **Just** - Modern command runner
- **Ruff** - Fast Python linter and formatter
- **Pre-commit** - Git hooks for code quality

### Key Libraries

- **Data Science**: NumPy, Pandas, Matplotlib, Seaborn
- **Machine Learning**: scikit-learn
- **Gradient Boosting**: XGBoost, LightGBM, CatBoost
- **Notebooks**: Jupyter, JupyterLab

## Automated Publishing

The course website is automatically rendered and published to GitHub Pages on every push to the main branch. The CI/CD pipeline:

1. Installs Python 3.13 and uv
2. Installs all dependencies
3. Renders the Quarto book
4. Deploys to GitHub Pages

## License

MIT License - see LICENSE file for details.

## Contact

For questions or issues, please open an issue on GitHub.
