# PySB Repository Structure

## Overview

PySB (Python Systems Biology) is a framework for building rule-based mathematical models of biochemical systems. It provides a Python-native approach to modeling that integrates seamlessly with scientific Python libraries like NumPy, SciPy, and SymPy.

## Core Architecture

### Model Definition System (`pysb/core.py`)

The heart of PySB is the model definition system, which provides the following core classes:

#### Model
- **Purpose**: Central container that holds all components of a biological model
- **Key Components**:
  - `monomers` - Protein/molecule definitions
  - `parameters` - Rate constants and initial values
  - `rules` - Reaction rules defining interactions
  - `observables` - Quantities to measure during simulation
  - `expressions` - Mathematical expressions using parameters
  - `compartments` - Spatial locations/volumes
  - `initials` - Initial species conditions
  - `tags` - Labels for pattern matching
  - `energypatterns` - Energy-based modeling components
- **Key Methods**:
  - `get_species()` - Generate all possible species from rules
  - `get_reactions()` - Generate reaction network
  - `odes` - System of ordinary differential equations

#### Monomer
- **Purpose**: Represents proteins or other molecules
- **Features**:
  - Define binding sites and their possible states
  - Create patterns for matching specific molecular configurations
  - Support for multi-state sites

#### Rule
- **Purpose**: Define biochemical reactions using pattern-based syntax
- **Types**:
  - Synthesis rules (∅ → Product)
  - Degradation rules (Reactant → ∅)
  - Complex formation/dissociation
  - State transitions
- **Special Features**:
  - `delete_molecules` - Remove bound molecules
  - `move_connected` - Move entire complexes between compartments
  - `total_rate` - Use macroscopic rates
  - `energy` - Energy-based rules

#### ComplexPattern
- **Purpose**: Represent molecular complexes and binding patterns
- **Features**:
  - Graph-based representation
  - Pattern matching with wildcards (ANY, WILD)
  - Match-once semantics
  - Compartment-aware

#### Observable
- **Purpose**: Define measurable quantities in the model
- **Types**:
  - Species-based (count entire complexes)
  - Molecule-based (count individual proteins)
- **Usage**: Track simulation outputs for comparison with experimental data

### Pattern Matching (`pysb/pattern.py`)

Sophisticated pattern matching system for finding molecular patterns:

- **Graph-based matching** using NetworkX
- **Filter predicates** for flexible component selection:
  - `Pattern` - Match by molecular pattern
  - `Name` - Match by regex on names
  - `Module` - Match by defining module
  - `Function` - Match by defining function
- **Logical operations** (AND, OR, NOT) on filters
- **SpeciesPatternMatcher** for efficient runtime matching

### Macro System (`pysb/macros.py`)

High-level modeling shortcuts for common biochemical patterns:

#### Basic Macros
- `equilibrate()` - Reversible binding equilibrium
- `bind()` - Simple protein-protein binding
- `catalyze()` - Michaelis-Menten enzymatic reactions
- `synthesize()`/`degrade()` - Production/degradation

#### Advanced Macros
- `catalyze_state()` - Enzymatic state changes
- `bind_table()` - Combinatorial binding interactions
- `assemble_pore_sequential()` - Step-wise pore assembly
- `bind_complex()` - Binding to existing complexes

### Builder Pattern (`pysb/builder.py`)

Alternative model construction approach:
- Object-oriented interface
- No namespace pollution
- Parameter estimation support
- Prior distribution specification
- Subclassable for custom modeling motifs

### Annotation System (`pysb/annotation.py`)

Lightweight semantic annotation:
- RDF triple-based (subject, predicate, object)
- MIRIAM-compatible annotations
- Integration with identifiers.org
- Biomodels.net qualifiers support

## Integration Components

### BioNetGen Interface (`pysb/bng.py`)

Integration with BioNetGen for:
- Network generation from rules
- Stochastic/deterministic simulation
- BNGL format import/export
- Multiple interface modes:
  - `BngConsole` - Interactive console
  - `BngFileInterface` - File-based communication

### Export System (`pysb/export/`)

Export models to various formats:
- **BNGL** (`bngl.py`) - BioNetGen language
- **SBML** (`sbml.py`) - Systems Biology Markup Language
- **Kappa** (`kappa.py`) - Kappa language
- **MATLAB** (`matlab.py`) - MATLAB ODE format
- **Mathematica** (`mathematica.py`) - Mathematica format
- **JSON** (`json.py`) - JSON representation
- **StochKit** (`stochkit.py`) - StochKit XML format
- **PySB Flat** (`pysb_flat.py`) - Flattened Python code

### Import System (`pysb/importers/`)

Import models from:
- **BNGL** (`bngl.py`) - BioNetGen models
- **SBML** (`sbml.py`) - SBML models
- **JSON** (`json.py`) - JSON format

### Simulator Framework (`pysb/simulator/`)

Multiple simulation backends with unified interface:

#### Base Architecture
- `base.py` - Abstract simulator interface
- `SimulationResult` - Unified results container

#### Deterministic Simulators
- `scipyode.py` - SciPy ODE solvers (LSODA, etc.)
- `cupsoda.py` - GPU-accelerated ODE solving

#### Stochastic Simulators
- `bng.py` - BioNetGen's SSA
- `stochkit.py` - StochKit interface
- `cuda_ssa.py` - GPU-accelerated SSA (CUDA)
- `opencl_ssa.py` - GPU-accelerated SSA (OpenCL)

#### Rule-Based Simulators
- `kappa.py` - Kappa simulator interface

## Utility Components

### Analysis Tools (`pysb/tools/`)
- `sensitivity_analysis.py` - Parameter sensitivity analysis
- `render_reactions.py` - Reaction network visualization
- `render_species.py` - Species visualization
- `species_graph.py` - Species interaction graphs

### Other Utilities
- `integrate.py` - Legacy ODE integration interface
- `pathfinder.py` - Path analysis in reaction networks
- `util.py` - General utility functions
- `logging.py` - Logging configuration

## Key Design Principles

1. **Declarative Syntax**: Components automatically export to namespace for natural Python syntax
2. **Pattern-Based**: Use graph patterns to specify molecular interactions
3. **Modular**: Clear separation between model definition, simulation, and analysis
4. **Extensible**: Easy to add new simulators, exporters, or analysis tools
5. **Pythonic**: Leverages Python features like operator overloading and context managers

## Workflow

1. **Define Model**: Create monomers, parameters, rules, and observables
2. **Generate Network**: Convert rules to species and reactions
3. **Simulate**: Choose appropriate simulator backend
4. **Analyze**: Use built-in tools or export for external analysis

## Example Usage Pattern

```python
from pysb import *

Model()

# Define a simple protein
Monomer('A', ['b'])
Parameter('A_0', 100)
Initial(A(b=None), A_0)

# Define binding reaction
Monomer('B', ['b'])
Parameter('B_0', 100)
Initial(B(b=None), B_0)

Parameter('kf', 1e-3)
Parameter('kr', 1e-1)
Rule('A_B_bind', A(b=None) + B(b=None) | A(b=1) % B(b=1), kf, kr)

# Observable
Observable('A_B_complex', A(b=1) % B(b=1))
```

This structure enables PySB to provide a powerful, expressive framework for systems biology modeling while maintaining compatibility with established tools and formats.