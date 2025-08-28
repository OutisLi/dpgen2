# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Installation
- Install package: `pip install -e .`
- Install with test dependencies: `pip install -e .[test]`
- Install with GUI dependencies: `pip install -e .[gui]`
- Install with docs dependencies: `pip install -e .[docs]`

### Testing
- Run all tests: `python -m pytest tests/`
- Run specific test file: `python -m pytest tests/test_specific_file.py`
- Run tests with coverage: `python -m pytest tests/ --cov=dpgen2`
- Run tests in verbose mode: `python -m pytest tests/ -v`

### Code Quality
- Lint with ruff: `ruff check dpgen2/`
- Format with ruff: `ruff format dpgen2/`
- Fix linting issues: `ruff check dpgen2/ --fix`
- Type checking: `pyright dpgen2/`

### Documentation
- Build docs: `cd docs && make html`
- View docs locally: Open `docs/_build/html/index.html`

### Running DPGEN2
- Submit workflow: `dpgen2 submit CONFIG.json`
- Check status: `dpgen2 status CONFIG.json WORKFLOW_ID`
- Watch workflow: `dpgen2 watch CONFIG.json WORKFLOW_ID`
- Download artifacts: `dpgen2 download CONFIG.json WORKFLOW_ID`
- Start GUI: `dpgen2 gui`

## Project Architecture

DPGEN2 is a concurrent learning workflow system for generating machine learning potential energy models. It implements the DP-GEN algorithm using dflow (Argo Workflows on Kubernetes).

### Core Components

**Main Entry Point**: `dpgen2/entrypoint/main.py`
- CLI interface with commands: submit, resubmit, status, watch, download, gui
- Handles JSON configuration files and workflow management

**Workflow Engine**: `dpgen2/flow/dpgen_loop.py`
- Implements the main concurrent learning loop
- Contains `ConcurrentLearning` and `ConcurrentLearningLoop` steps
- Orchestrates the iterative training-exploration-selection-labeling process

**Operators** (`dpgen2/op/`): Individual workflow steps
- `prep_dp_train.py`: Prepare and run DeePMD-kit training
- `prep_lmp.py`: Prepare and run LAMMPS exploration
- `select_confs.py`: Select configurations for labeling
- `prep_fp.py`: Prepare and run first-principles calculations
- `collect_data.py`: Collect and aggregate training data

**Super Operators** (`dpgen2/superop/`): Composite operators
- `block.py`: Main concurrent learning block (training + exploration + selection + labeling)
- Combines multiple individual operators into a single workflow step

**Exploration** (`dpgen2/exploration/`): Exploration strategy components
- `scheduler/`: Exploration scheduling and convergence checking
- `selector/`: Configuration selection algorithms
- `task/`: Task group definitions for exploration
- `report/`: Exploration reporting and analysis

**First-Principles Integration** (`dpgen2/fp/`): 
- Support for VASP, CP2K, ABACUS, Gaussian
- Input preparation and output parsing

**Configuration** (`dpgen2/conf/`):
- `alloy_conf.py`: Alloy configuration handling
- `conf_generator.py`: Configuration generation utilities
- `unit_cells.py`: Unit cell definitions

### Workflow Algorithm

The DP-GEN algorithm follows these iterative steps:
1. **Training**: Train multiple DP models with different random seeds
2. **Exploration**: Use models to explore configuration space via LAMMPS
3. **Selection**: Select configurations based on model deviation analysis
4. **Labeling**: Run first-principles calculations on selected configurations
5. **Collection**: Add new labeled data to training dataset

The workflow continues until convergence (no more critical configurations found).

### Key Design Patterns

- **Operators vs Workflow**: Operators are implemented and tested independently, then composed into workflows
- **Mock Testing**: Workflows are tested with mocked operators for reliability
- **Artifact Management**: Heavy use of dflow artifacts for data flow between steps
- **Configuration-Driven**: JSON configuration files define workflow parameters

### File Structure Conventions

- Operators are in `dpgen2/op/` with prefix `prep_` or `run_`
- Super operators are in `dpgen2/superop/`
- Exploration logic is in `dpgen2/exploration/` with subdirectories for components
- Tests are in `tests/` mirroring the source structure
- Examples are in `examples/` with JSON configuration files

### Dependencies

Key dependencies include:
- `dflow`: Workflow orchestration (Argo Workflows)
- `dpdata`: Data format conversion
- `pydflow`: Python workflow interface
- `numpy`, `scipy`: Scientific computing
- `lbg`, `fpop`: Additional utilities

### Testing Strategy

- Unit tests for individual operators
- Workflow tests with mocked operators
- Integration tests using fake data sets
- Context managers for test setup in `tests/context.py`

## Development Guidelines

### Code Style and Formatting
- Uses ruff for linting and formatting with line length 150 characters
- Python 3.7+ compatibility required (target version py312 for ruff)
- Double quotes for strings, spaces for indentation
- Allows unused variables when underscore-prefixed

### Testing Patterns
- Mock all external dependencies in workflow tests using `tests/mocked_ops.py`
- Use context managers from `tests/context.py` for test setup
- Test operators independently before integrating into workflows
- Fake datasets available in `tests/fake_data_set.py`

### Configuration and Examples
- JSON configuration files define all workflow parameters
- Working examples available in `examples/` directory for different systems (water, AlMg, CH4, etc.)
- Each example includes input.json and any required template files