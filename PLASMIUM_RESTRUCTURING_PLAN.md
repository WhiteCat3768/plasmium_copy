# Comprehensive Restructuring Plan: Plasmium (Stellar Core Simulation)

## Executive Summary

Complex stellar physics simulation (~3,600 lines) modeling nuclear reactions, element abundance evolution, and thermodynamic conditions in stellar cores. Current structure uses flat module architecture with global state and wildcard imports.

**Project Name:** Plasmium (formerly referenced as "Star Soft" in documentation)

**Core Purpose:** Simulate stellar core evolution from hydrogen burning through advanced nuclear processes with comprehensive reaction networks (48 isotopes, 83+ nuclear processes) and physical modeling.

---

## Current Architecture Analysis

### File Inventory (14 Python files, 3,612 total lines)

| File | Lines | Primary Functionality |
|------|-------|----------------------|
| `Plasmium_main.py` | 389 | Main entry point, simulation loop, I/O, visualization |
| `Processes.py` | 1,003 | 83 nuclear process rate calculations |
| `Differentials_solving_methods.py` | 547 | Courant & RK4 numerical integration methods |
| `Neutrino_spectrum.py` | 496 | Neutrino energy spectrum calculations |
| `Element_operations.py` | 283 | Element data handling & parsing utilities (16 functions) |
| `Resonant_reactions.py` | 226 | Resonant nuclear reaction handling (Breit-Wigner) |
| `Electron_positron_operations.py` | 168 | Electron/positron concentration handling |
| `Unresonant_reactions.py` | 145 | Non-resonant nuclear reaction handling (S-factor) |
| `Decay_reactions.py` | 96 | Radioactive decay handling (19 decays) |
| `Displays.py` | 95 | Output formatting functions |
| `Parcer.py` | 75 | Post-processing visualization script |
| `Temperature_change.py` | 61 | Temperature evolution calculations |
| `Advanced_tweakables.py` | 12 | Simulation configuration parameters (9 settings) |
| `World_constants.py` | 16 | Physical constants class (15 constants) |

**Data Files:**
- `Elements_data.txt` - Nuclear data for 48 isotopes (n0 to P30)
- `Simulation_input.txt` - Input configuration file
- `Resonant_reactions_resonance_parameters.txt` - Resonance parameters

**Documentation:**
- `Star Soft Documentation.pdf` - User documentation
- `Star Soft Documentation.tex` - LaTeX source
- `Star Soft Documentation.toc` - Table of contents

### Strengths to Preserve
1. **Clear physics domain separation** - Dedicated modules for reaction types (unresonant, resonant, decay, processes)
2. **Comprehensive nuclear network** - 48 isotopes tracked, 83 nuclear processes
3. **Multiple integration methods** - Courant (adaptive) and RK4 implementations
4. **Complete physics modeling** - Temperature evolution, neutrino cooling, e⁻/e⁺ annihilation
5. **Modular reaction matrices** - Structured data storage for reactions

### Critical Issues to Address

1. **Wildcard imports breaking encapsulation**
   - `from Decay_reactions import *`
   - `from Differentials_solving_methods import *`
   - `from Displays import *`
   - `from Processes import *`
   - `from Resonant_reactions import *`
   - `from Unresonant_reactions import *`
   - `from Element_operations import *`
   - `from Electron_positron_operations import *`
   - `from Temperature_change import *`

2. **Global state with scattered configuration**
   - `Worldconstants` class used globally as `Wc`
   - `Advancedtweakbles` class used globally as `At`
   - No centralized state management

3. **No package structure** - All 14 modules at root level

4. **Monolithic main script** - `Plasmium_main.py` mixes:
   - Input parsing
   - Physics calculations
   - Simulation loop control
   - Output writing
   - Visualization plotting

5. **Circular dependencies likely** between reaction modules via wildcard imports

6. **No type hints or documentation strings** on any functions

7. **Magic numbers hardcoded** throughout (e.g., matrix row indices like `[16]`, `[25]`, `[19]`, `[9]`)

8. **Mixed concerns** - Physics logic mixed with file I/O and matplotlib plotting

9. **Non-standard naming**:
   - Russian comments throughout
   - Inconsistent naming conventions (`Ammount_of_elements` vs `Amount_of_elements`)
   - Class names without proper casing (`Advancedtweakbles`, `Worldconstants`)

10. **No testing infrastructure** - Zero unit tests

11. **Matrix-based data structures** using parallel lists instead of proper objects:
    - `Elements = [[], [], [], [], []]` - 5 parallel lists for element data
    - Reaction matrices with 20-27 rows each
    - Hardcoded row indices for accessing specific data fields

12. **Hardcoded file paths** - All files referenced by string literals

---

## Multi-Step Restructuring Plan

### **Phase 1: Foundation & Package Structure** (Steps 1-4)

#### Step 1.1: Create Modern Package Hierarchy

```
plasmium/
├── __init__.py                 # Package initialization, version info, public API
├── __main__.py                 # Entry point (replaces Plasmium_main.py execution)
├── cli.py                      # Command-line interface
│
├── config/
│   ├── __init__.py
│   ├── settings.py             # Configuration management (replaces Advanced_tweakables.py)
│   ├── constants.py            # Physical constants (replaces World_constants.py)
│   └── input_parser.py         # Simulation_input.txt parsing
│
├── core/
│   ├── __init__.py
│   ├── simulation.py           # Main simulation loop & orchestration
│   ├── state.py                # Simulation state dataclasses
│   ├── integrators.py          # Numerical methods (Courant, RK4)
│   └── timestep.py             # Adaptive timestep logic
│
├── physics/
│   ├── __init__.py
│   ├── nuclear_network.py      # Element data & reaction network management
│   │
│   ├── reactions/
│   │   ├── __init__.py
│   │   ├── base.py             # Abstract reaction classes & protocols
│   │   ├── unresonant.py       # Non-resonant reactions (12 reactions, replaces Unresonant_reactions.py)
│   │   ├── resonant.py         # Resonant reactions (2 reactions, replaces Resonant_reactions.py)
│   │   ├── decay.py            # Radioactive decay (19 decays, replaces Decay_reactions.py)
│   │   └── processes.py        # Nuclear processes (83 processes, replaces Processes.py)
│   │
│   ├── thermodynamics/
│   │   ├── __init__.py
│   │   ├── temperature.py      # Temperature evolution (replaces Temperature_change.py)
│   │   └── equation_of_state.py # EOS calculations
│   │
│   ├── particles/
│   │   ├── __init__.py
│   │   ├── electrons.py        # Electron/positron handling (replaces Electron_positron_operations.py)
│   │   └── neutrinos.py        # Neutrino spectrum & cooling (replaces Neutrino_spectrum.py)
│   │
│   └── rates/
│       ├── __init__.py
│       └── rate_calculator.py  # Reaction rate computations
│
├── io/
│   ├── __init__.py
│   ├── element_data.py         # Elements_data.txt parsing (moves Element_operations.py functions)
│   ├── output_writer.py        # Simulation_output.txt, Logs.txt writing
│   └── logging_config.py       # Logging configuration
│
├── analysis/
│   ├── __init__.py
│   ├── visualizer.py           # Matplotlib plotting (replaces Parcer.py + plotting in Plasmium_main.py)
│   └── diagnostics.py          # Burning speeds, times, equilibrium detection
│
├── utils/
│   ├── __init__.py
│   └── helpers.py              # Utility functions (concentration lookups, mass calculations)
│
└── tests/
    ├── __init__.py
    ├── test_constants.py
    ├── test_element_data.py
    ├── test_reactions/
    │   ├── __init__.py
    │   ├── test_unresonant.py
    │   ├── test_resonant.py
    │   ├── test_decay.py
    │   └── test_processes.py
    ├── test_integrators.py
    ├── test_simulation.py
    └── test_regression.py      # Compare old vs new implementation outputs
```

#### Step 1.2: Define Core Data Classes

Replace list-based data structures with proper typed dataclasses:

```python
# physics/nuclear_network.py
@dataclass(frozen=True)
class Element:
    """Represents a nuclear isotope."""
    name: str           # e.g., 'H1', 'He4', 'C12'
    mass: float         # Atomic mass in atomic mass units
    charge: int         # Nuclear charge (proton number)
    spin: float         # Nuclear spin
    parity: int         # Parity: +1 or -1
    
@dataclass(frozen=True)
class Reaction:
    """Base class for all nuclear reactions."""
    reactants: tuple[str, ...]
    products: tuple[str, ...]
    q_value: float      # Energy release in SA units
    
@dataclass
class SimulationState:
    """Immutable snapshot of simulation state."""
    time: float
    temperature: float
    concentrations: dict[str, float]
    electron_concentration: float
    positron_concentration: float
    reaction_rates: dict[str, float]
    
@dataclass
class SimulationConfig:
    """Configuration parameters (replaces Advancedtweakbles)."""
    neutrino_spectrum_time: float = 1e3
    distance_neutrino_spectrum: float = 1.5e8
    max_up_concentration_change: float = 1.0
    max_time_step: float = 1e6
    max_relative_time_step_change: float = 0.03
    resonant_reactions_relative_temperature_update: float = 0.01
    speed_of_light_velocity_limit: float = 0.01
    resonance_width_energy_range: int = 500
    annihilation: bool = True
    neutrino_cooling: bool = False
```

#### Step 1.3: Establish Dependency Injection Pattern

- Remove all wildcard imports between modules
- Pass dependencies explicitly through function/method parameters
- Use factory patterns for complex object creation
- Implement service locator pattern for shared resources (constants, config)

#### Step 1.4: Set Up Development Infrastructure

- Create `pyproject.toml` with project metadata and dependencies
- Configure `pytest` for testing
- Set up `pre-commit` hooks (Black, Ruff, MyPy)
- Create `README.md` with installation and usage instructions
- Add `.gitignore` for Python projects

---

### **Phase 2: Core Physics Refactoring** (Steps 5-10)

#### Step 2.1: Constants & Configuration Management

**Files:** `config/constants.py`, `config/settings.py`

**Tasks:**
- Convert `Worldconstants` class to module-level constants with proper units
- Rename class to `WorldConstants` (proper casing)
- Add unit annotations and conversion utilities
- Convert `Advancedtweakbles` to Pydantic model or dataclass with validation
- Fix typos in class name and attribute names
- Implement configuration loading from YAML/JSON as alternative to txt format
- Add configuration validation (range checks, type validation)

**Key Constants to Preserve (from World_constants.py):**
- `c = 9.467e17` cm/year (speed of light)
- `Pi = 3.14159262`
- `e = 2.7182818284590`
- `M_nuc = 1.660539068e-24` g (atomic mass unit)
- `M_e = 9.1093837015e-28` g (electron mass)
- `St_Bol = 5.67e-8 * 1e+22 * 3.154e+7 * 1e-4` SA Watt/cm²/K⁴ (Stefan-Boltzmann)
- `R = 8.3 * 1e+22` SA Joule/K/mole (gas constant)
- `N_a = 6.022e+23` 1/mole (Avogadro's number)
- `k = 1.380649e-23 * 1e+22` SA Joule/K (Boltzmann constant)
- `h = 6.63e-34 * 1e+22 / 3.154e+7` SA Joule·year (Planck constant)
- `h_ = 1.0545726e-34 * 1e+22 / 3.154e+7` SA Joule·year (reduced Planck constant)
- `p0 = 1e+5 * 1e+16` SA Pascal
- `T0 = 273.15` K

**Key Settings to Preserve (from Advanced_tweakables.py):**
- `Neutrino_spectrum_time = 1e+3` years
- `Distance_neutrino_spectrum = 1.5e+8` km
- `Max_up_concentration_change = 1e+0`
- `Max_time_step = 1e+6` years
- `Max_relative_time_step_change = 0.03`
- `Resonant_reactions_relative_temperature_update = 0.01`
- `Speed_of_light_velocity_limit = 0.01` (fraction of c)
- `Resonance_width_energy_range = 500`
- `Annihilation = True`
- `Neutrino_cooling = False`

#### Step 2.2: Element Data Management

**Files:** `io/element_data.py`, `physics/nuclear_network.py`

**Tasks:**
- Create `Element` dataclass with validation
- Replace `Elements_list_create()` with proper parser returning list of `Element` objects
- Move helper functions from `Element_operations.py`:
  - `Is_element_new()` → utility function
  - `Mass_of_element()` → method on `Element` or lookup function
  - `Charge_of_element()` → method on `Element` or lookup function
  - `Spin_of_element()` → method on `Element` or lookup function
  - `Parity_of_element()` → method on `Element` or lookup function
  - `Concentration_of_element()` → utility function
  - `Mass_fraction_of_element()` → utility function
  - `Max_concentration_of_element()` → method in `NuclearNetwork`
  - `Compound_core()` → needs investigation (line 80+)
- Implement efficient element lookup (dict by name)
- Add mass fraction and concentration calculation methods
- Parse `Elements_data.txt` format: `{name} {mass} {charge} {spin}{parity}`

**Functions to Migrate from Element_operations.py (16 total):**
1. `Elements_list_create()` → `parse_elements_data()`
2. `Is_element_new()` → internal utility
3. `Mass_of_element()` → `get_element_mass()`
4. `Charge_of_element()` → `get_element_charge()`
5. `Spin_of_element()` → `get_element_spin()`
6. `Parity_of_element()` → `get_element_parity()`
7. `Concentration_of_element()` → `get_concentration()`
8. `Mass_fraction_of_element()` → `get_mass_fraction()`
9. `Max_concentration_of_element()` → `calculate_max_concentration()`
10. `Compound_core()` → investigate usage (starts line 80)
11. Plus 5 more functions (view rest of file)

#### Step 2.3: Reaction System Redesign

**Files:** `physics/reactions/base.py`, `physics/reactions/*.py`

**Tasks:**
- Create abstract `Reaction` base class with common interface
- Define protocol for reaction rate calculation
- Implement concrete classes:
  - `UnresonantReaction` (12 reactions from `Unresonant_reactions.py`)
  - `ResonantReaction` (2 reactions from `Resonant_reactions.py`)
  - `DecayReaction` (19 decays from `Decay_reactions.py`)
  - `Process` (83 processes from `Processes.py`)
- Replace matrix-based storage with object collections
- Implement reaction factory for parsing/loading
- Add reaction Q-value calculations
- Implement reaction type enumeration

**Matrix Structures to Replace:**

*Unresonant Reactions Matrix (22 rows × N reactions):*
- Rows include: reactants, products, S-factor parameters, temperature exponent, rate, etc.
- Row 16: reaction rates (used in `Differentials_solving_methods.py`)

*Resonant Reactions Matrix (27 rows × N reactions):*
- Rows include: resonance parameters, Breit-Wigner coefficients, concentrations, rate
- Row 22: last update temperature
- Row 25: reaction rates

*Decay Reactions Matrix (20 rows × N reactions):*
- Rows include: parent, daughters, decay constant, rate
- Row 19: reaction rates

*Processes Matrix (10 rows × 83 processes):*
- Rows include: 3 reactants, 4 products, rate
- Row 9: process rates

#### Step 2.4: Unresonant Reaction Implementation

**File:** `physics/reactions/unresonant.py`

**Current State:** 12 non-resonant reactions using S-factor formalism

**Functions to Migrate:**
- `Unresonant_reactions_matrix_create()` → `create_unresonant_reactions()`
- `Unresonant_reactions_matrix_update()` → `update_reaction_rates()`
- `Unresonant_reactions_matrix_print()` → debug method
- `Unresonant_reactions_reactions_count()` → calculate reaction rates
- Individual reaction functions (if any exist separately)

**Physics:** S-factor based cross-sections for non-resonant nuclear reactions

#### Step 2.5: Resonant Reaction Implementation

**File:** `physics/reactions/resonant.py`

**Current State:** 2 resonant reactions using Breit-Wigner formalism

**Functions to Migrate:**
- `Resonant_reactions_matrix_create()` → `create_resonant_reactions()`
- `Resonant_reactions_matrix_update()` → `update_cross_sections()` (expensive operation)
- `Resononant_reactions_reactions_count()` → calculate reaction rates
- Resonance parameter loading from `Resonant_reactions_resonance_parameters.txt`

**Physics:** Breit-Wigner resonance formula with velocity integration
- Expensive cross-section updates (optimized with temperature threshold check)
- Requires numerical integration over velocity distribution
- Parameters: resonance energy, widths, strengths

#### Step 2.6: Decay Reaction Implementation

**File:** `physics/reactions/decay.py`

**Current State:** 19 radioactive decay channels

**Functions to Migrate:**
- `Decay_reactions_matrix_create()` → `create_decay_reactions()`
- `Decay_reactions_matrix_update()` → `update_decay_constants()`
- `Decay_reactions_count()` → calculate decay rates

**Physics:** Radioactive decay with temperature/density dependence where applicable

#### Step 2.7: Nuclear Process Implementation

**File:** `physics/reactions/processes.py`

**Current State:** 83 individual process functions (`process_1` through `process_83`)

**Functions to Migrate:**
- `Processes_matrix_create()` → `create_processes()`
- `Processes_matrix_update()` → `update_process_rates()`
- `Processes_count()` → calculate process rates
- All 83 `process_N()` functions → data-driven approach

**Refactoring Strategy:**
- Convert 83 individual functions into data-driven approach
- Store process formulas as configurable expressions or coefficient tables
- Group processes by reaction chain (pp chains, CNO cycles, alpha captures, etc.)
- Implement process rate calculator with vectorization support using NumPy
- Consider using `numba` JIT compilation for performance-critical loops

**Process Categories (to be identified):**
- pp-chain reactions (process_1 through process_~10)
- CNO cycle reactions
- Alpha capture reactions
- Heavy element synthesis
- etc.

#### Step 2.8: Electron/Positron Physics

**File:** `physics/particles/electrons.py`

**Current State:** Electron/positron concentration tracking and annihilation

**Functions to Migrate from `Electron_positron_operations.py`:**
- `Electron_positron_initial_concentration_count()` → `calculate_initial_lepton_concentrations()`
- `Electron_positron_unresonant_matrix_create()` → track lepton effects in reactions
- `Electron_positron_resonant_matrix_create()` → same for resonant
- `Electron_positron_decay_matrix_create()` → same for decay
- `Electron_positron_processes_matrix_create()` → same for processes
- `Average_per_particle_weight_count()` → `calculate_mean_molecular_weight()`
- `Electron_positron_burning_speed()` → `calculate_lepton_production_rates()`
- `El_pos_annihilation()` → `annihilate_leptons()`
- `El_pos_annihilation_temperature_change()` → `annihilation_heating()`
- `El_pos_burning_time_count()` → `calculate_lepton_burning_times()`

**Physics:**
- Charge neutrality constraint for initial electron concentration
- Lepton production/consumption in weak interactions
- Instant e⁻/e⁺ annihilation with energy release
- Mean molecular weight calculation for EOS

#### Step 2.9: Neutrino Physics

**File:** `physics/particles/neutrinos.py`

**Current State:** Neutrino spectrum calculation and optional cooling

**Functions to Migrate from `Neutrino_spectrum.py`:**
- `Neutrino_unresonant_reactions_matrix_create()` → initialize neutrino tracking
- `Neutrino_resonant_reactions_matrix_create()` → same
- `Neutrino_decay_reactions_matrix_create()` → same
- `Neutrino_processes_matrix_create()` → same
- `Neutrino_spectrum_create()` → `calculate_neutrino_spectrum()`
- `Neutrino_spectrum_visualize()` → move to `analysis/visualizer.py`

**Physics:**
- Neutrino production from weak interactions
- Energy spectrum calculation
- Optional neutrino cooling (energy loss from core)
- Flux calculation at detector distance

---

### **Phase 3: Simulation Engine** (Steps 11-15)

#### Step 3.1: State Management

**File:** `core/state.py`

**Tasks:**
- Create immutable `SimulationState` dataclass using `frozen=True`
- Include all state variables:
  - `time: float` (years)
  - `temperature: float` (K)
  - `concentrations: dict[str, float]` (element concentrations)
  - `mass_fractions: dict[str, float]`
  - `electron_concentration: float`
  - `positron_concentration: float`
  - `reaction_rates: dict[str, float]`
  - `burning_speeds: dict[str, float]`
  - `burning_times: dict[str, float]`
- Implement state update methods returning new state instances (functional style)
- Add state validation and normalization (mass conservation fix)
- Implement state serialization for checkpointing

**Current State Variables in Plasmium_main.py:**
- `Time` - simulation time
- `T` - temperature
- `Mass_fractions` - [[element_names], [mass_fractions]]
- `Concentrations` - [[element_names], [concentrations]]
- `Electrons_concentration`, `Positrons_concentration`
- `Average_per_particle_weight` - mean molecular weight
- `Burning_speed`, `Elements_burning_time`
- `Unresonant_reactions`, `Resonant_reactions`, `Decay_reactions`, `Processes` matrices
- Various neutrino matrices

#### Step 3.2: Numerical Integrators

**File:** `core/integrators.py`

**Current State:** Two integration methods in `Differentials_solving_methods.py`

**Functions to Migrate:**
- `Courant_diff_solve()` → `CourantIntegrator.step()`
- `Runge_Kutta_4()` → `RK4Integrator.step()`

**Tasks:**
- Create `Integrator` abstract base class with protocol:
  ```python
  @abstractmethod
  def step(self, state: SimulationState, dt: float) -> tuple[SimulationState, Diagnostics]:
      pass
  ```
- Implement `CourantIntegrator` subclass:
  - Adaptive timestep based on burning times
  - Stability constraints
  - Return new state and diagnostics
- Implement `RK4Integrator` subclass:
  - Classic 4th-order Runge-Kutta
  - Fixed or adaptive timestep option
- Standardize interface across integrators
- Extract timestep logic into separate `timestep.py` module
- Implement adaptive timestep controls:
  - `Max_time_step = 1e+6` years
  - `Max_relative_time_step_change = 0.03`
  - Based on minimum burning time

**Integration Inputs (from current Courant/RK4 signatures):**
- Reaction matrices and rates
- Concentrations and mass fractions
- Physical parameters (R_core, T, ro, etc.)
- Lepton concentrations and effects
- Neutrino matrices
- Temperature change parameters

#### Step 3.3: Temperature Evolution

**File:** `physics/thermodynamics/temperature.py`

**Current State:** `Temperature_change.py` with temperature evolution logic

**Functions to Migrate:**
- `Temperature_change_speed_count()` → `calculate_temperature_rate()`
- `Temperature_change_time_count()` → `calculate_temperature_timescale()`
- Move from `Differentials_solving_methods.py`: temperature update logic

**Physics:**
- Energy generation from nuclear reactions
- Energy loss from neutrino emission
- Photon diffusion to surface
- Annihilation heating
- Heat capacity calculation
- Optional temperature evolution (configurable)

**Equation:**
```
dT/dt = (energy_generation - energy_loss) / heat_capacity
```

#### Step 3.4: Equation of State

**File:** `physics/thermodynamics/equation_of_state.py`

**New Module** (logic currently scattered in main loop)

**Tasks:**
- Implement ideal gas EOS with corrections
- Calculate pressure from temperature, density, composition
- Calculate mean molecular weight (currently in `Average_per_particle_weight_count()`)
- Include radiation pressure if needed
- Handle degenerate matter if applicable

#### Step 3.5: Main Simulation Loop

**Files:** `core/simulation.py`, `__main__.py`, `cli.py`

**Current State:** `Plasmium_main.py` contains entire simulation loop (389 lines)

**Tasks:**
- Create `Simulation` class orchestrating the entire process:
  ```python
  class Simulation:
      def __init__(self, config: SimulationConfig, network: NuclearNetwork)
      def initialize(self, input_params: dict) -> SimulationState
      def run(self, state: SimulationState) -> SimulationState
      def finalize(self, state: SimulationState) -> None
  ```
- Lifecycle: `initialize()` → `run()` → `finalize()`
- Extract I/O operations from physics calculations
- Implement progress tracking and logging
- Implement checkpointing capability (save/load state)
- Handle integration method selection (Courant vs RK4)
- Manage adaptive timestep loop
- Coordinate all physics updates per timestep

**Main Loop Logic to Preserve:**
1. Read input parameters from `Simulation_input.txt`
2. Initialize elements, reactions, concentrations
3. Calculate initial electron/positron concentrations
4. Create reaction matrices
5. Output initial state
6. While Time <= Time_limit:
   - Update reaction rates
   - Choose integrator (Courant or RK4)
   - Calculate dt and derivatives
   - Update concentrations
   - Renormalize mass fractions (conservation fix)
   - Update electron/positron concentrations
   - Apply annihilation if enabled
   - Update temperature if enabled
   - Generate neutrino spectrum at specified time
   - Output state at specified frequency
   - Log diagnostics
7. Finalize and visualize

---

### **Phase 4: I/O & Visualization** (Steps 16-18)

#### Step 4.1: Input/Output System

**Files:** `config/input_parser.py`, `io/output_writer.py`, `io/logging_config.py`

**Current State:** Input parsing in `Plasmium_main.py` lines 18-50, output throughout

**Input Parsing Tasks:**
- Create dedicated parser for `Simulation_input.txt`
- Current format (24 lines, every other line is data):
  1. R_core (core radius in km → cm)
  2. ro (density)
  3. T (temperature)
  4. T_surface (surface temperature)
  5. R_star (stellar radius in km → cm)
  6. Mass_fractions_input (semicolon-separated "Element: fraction" pairs)
  7. Time_limit (in Myr → years)
  8. Integration method ("Courant" or "Runge-Kutta_4") + Speed parameter
  9. Temperature_change_conf ("True" or "False")
  10. Frequency (output frequency)
  11. Test mode ("File" or other)
  - Plus additional parameters...
- Support alternative formats (YAML, JSON)
- Validate all inputs with clear error messages
- Provide sensible defaults

**Output Writing Tasks:**
- Create structured output writer
- Write to `Simulation_output.txt` in parseable format
- Write logs to `Logs.txt` with timestamps
- Support CSV output for easier post-processing
- Consider HDF5 for large simulations
- Separate data output from logging

**Logging Configuration:**
- Use Python's `logging` module instead of manual file writes
- Configurable log levels
- Timestamp formatting (currently uses `datetime.now().time()`)
- Structured log messages

#### Step 4.2: Visualization Module

**Files:** `analysis/visualizer.py`

**Current State:** 
- Plotting code in `Plasmium_main.py` (lines 327-389)
- Separate `Parcer.py` script for post-processing

**Functions to Migrate:**
- From `Plasmium_main.py`: main plotting loop (lines 327-389)
- From `Parcer.py`: all parsing and plotting functions
- From `Neutrino_spectrum.py`: `Neutrino_spectrum_visualize()`

**Tasks:**
- Move all plotting code from `Plasmium_main.py` and `Parcer.py`
- Create reusable plotting functions with consistent styling:
  - `plot_element_evolution()` - element abundances vs time
  - `plot_temperature_evolution()` - temperature vs time
  - `plot_lepton_concentrations()` - e⁻/e⁺ vs time
  - `plot_neutrino_spectrum()` - neutrino energy spectrum
  - `plot_reaction_rates()` - reaction network flow
- Use matplotlib with seaborn styling
- Support interactive features (mplcursors already used)
- Multiple output formats (PNG, PDF, SVG, interactive HTML)
- Configurable plot appearance

**Current Plot Features:**
- Log-log scale for time and abundances
- Viridis colormap for elements
- Interactive tooltips with mplcursors
- Temperature overlay
- Labels for all elements

#### Step 4.3: Diagnostics & Analysis

**Files:** `analysis/diagnostics.py`

**Current State:** Diagnostics scattered in `Plasmium_main.py` and `Displays.py`

**Functions to Migrate from `Displays.py`:**
- All display formatting functions (view file to get complete list)
- Time formatting
- State display functions
- Debug display functions

**Tasks:**
- Extract burning speed/time calculations into dedicated module
- Implement equilibrium detection algorithms
- Add reaction flow analysis
- Create summary statistics generator
- Implement conservation law checks (mass, energy, charge)
- Add timescale analysis
- Create diagnostic reports

**Diagnostics to Track:**
- Minimum burning time and corresponding element
- Reaction rates by type
- Energy generation rates
- Neutrino luminosity
- Temperature change timescale
- Mass conservation error
- Equilibrium conditions

---

### **Phase 5: Quality & Testing** (Steps 19-21)

#### Step 5.1: Testing Infrastructure

**Setup:**
- Install pytest framework
- Configure pytest in `pyproject.toml`
- Set up test fixtures for common objects
- Create mock data for testing

**Test Coverage Goals:**

*Unit Tests:*
- `test_constants.py` - Verify all physical constants
- `test_element_data.py` - Test element parsing and lookup
- `test_reactions/test_unresonant.py` - Test individual unresonant reactions
- `test_reactions/test_resonant.py` - Test resonant reaction cross-sections
- `test_reactions/test_decay.py` - Test decay rates
- `test_reactions/test_processes.py` - Spot-check key processes
- `test_integrators.py` - Test Courant and RK4 on known problems
- `test_temperature.py` - Test temperature evolution
- `test_leptons.py` - Test e⁻/e⁺ calculations

*Integration Tests:*
- Single timestep validation
- Multi-step evolution
- Conservation laws

*Regression Tests:*
- Run identical simulations with old and new code
- Compare outputs numerically (tolerance: 1e-10 relative)
- Automated comparison in CI

**Coverage Target:** >80% line coverage

#### Step 5.2: Documentation

**Documentation Types:**

*API Documentation:*
- Write comprehensive docstrings for all public APIs
- Use Google or NumPy style consistently
- Include type hints in signatures
- Document parameters, returns, raises
- Example usage in docstrings

*User Guide:*
- Installation instructions
- Quick start tutorial
- Input file format specification
- Configuration options explanation
- Output interpretation
- Common use cases

*Developer Guide:*
- Architecture overview
- Adding new reactions
- Extending physics modules
- Testing guidelines
- Contribution workflow

*Physics Documentation:*
- Document physics assumptions and approximations
- Reference equations and sources
- Validity ranges for formulas
- Known limitations

**Tools:**
- Sphinx for API documentation generation
- MkDocs for user guide
- Docstring linting with pydocstyle

#### Step 5.3: Code Quality Tools

**Configuration:**

*Code Formatting:*
- Black for automatic formatting
- Line length: 100 characters
- Target Python version: 3.10+

*Linting:*
- Ruff (fast, modern linter)
- Flake8 compatibility mode
- Custom rules for project-specific standards

*Type Checking:*
- MyPy for static type checking
- Strict mode enabled
- Type hints for all function signatures

*Pre-commit Hooks:*
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
  - repo: https://github.com/psf/black
    hooks:
      - id: black
  - repo: https://github.com/astral-sh/ruff-pre-commit
    hooks:
      - id: ruff
  - repo: https://github.com/pre-commit/mirrors-mypy
    hooks:
      - id: mypy
```

*CI/CD Pipeline:*
- GitHub Actions or GitLab CI
- Run tests on every push
- Check code quality
- Build documentation
- Publish to PyPI (optional)

---

### **Phase 6: Migration & Validation** (Steps 22-24)

#### Step 6.1: Incremental Migration Strategy

**Strangler Fig Pattern Approach:**

*Week 1-2: Foundation*
- Create package structure
- Implement constants and configuration
- Set up testing infrastructure
- Migrate element data parsing

*Week 3-4: Reaction System Foundation*
- Create reaction base classes
- Migrate unresonant reactions
- Migrate decay reactions
- Write tests for migrated code

*Week 5-6: Complete Reaction Implementations*
- Migrate resonant reactions (complex, expensive)
- Migrate all 83 processes
- Implement reaction network management
- Integration tests for reaction rates

*Week 7-8: State Management & Integrators*
- Create SimulationState dataclass
- Migrate Courant integrator
- Migrate RK4 integrator
- Test numerical accuracy

*Week 9-10: Simulation Engine & I/O*
- Create Simulation class
- Migrate main loop logic
- Implement I/O system
- End-to-end testing

*Week 11-12: Polish & Validation*
- Complete visualization module
- Write comprehensive documentation
- Performance optimization
- Full regression testing

#### Step 6.2: Validation Protocol

**Numerical Validation:**
1. Run identical simulations with old and new code
2. Compare outputs at each timestep
3. Acceptable tolerance: 1e-10 relative difference
4. Track divergence over time

**Conservation Laws:**
1. Mass conservation (with renormalization fix)
2. Charge conservation
3. Energy conservation (accounting for neutrino losses)
4. Baryon number conservation

**Edge Cases:**
1. Very low temperatures
2. Very high densities
3. Pure hydrogen initial composition
4. Equilibrium conditions
5. Very short/long timesteps

**Performance Benchmarks:**
1. Time per timestep
2. Memory usage
3. Scaling with number of isotopes
4. Comparison with original implementation

#### Step 6.3: Backward Compatibility

**Maintain Compatibility:**
- Keep `Simulation_input.txt` format support
- Provide conversion scripts for output formats
- Document any breaking changes clearly
- Offer migration guide for users

**Deprecation Strategy:**
- Warn about deprecated features
- Provide migration path
- Maintain old API temporarily with warnings
- Remove in major version bump

---

## Technical Standards to Implement

### Code Style

- **PEP 8 compliance** throughout
- **Type hints** for all function signatures (Python 3.10+ syntax)
- **Docstrings** (Google style) for all public APIs
- **Maximum line length:** 100 characters
- **Function length:** <50 lines ideally, extract helpers if longer
- **Class length:** <300 lines, split if needed
- **Module length:** <500 lines, refactor if exceeds

### Architecture Principles

1. **Single Responsibility:** Each module/class has one purpose
2. **Dependency Injection:** No hidden dependencies, pass explicitly
3. **Immutability:** State objects are frozen where possible
4. **Separation of Concerns:** Physics ≠ I/O ≠ Visualization
5. **Explicit over Implicit:** No magic, no wildcard imports
6. **Composition over Inheritance:** Prefer composition for code reuse
7. **Tell, Don't Ask:** Objects should do work, not expose data

### Performance Considerations

1. **Vectorization:** Use NumPy for array operations where possible
2. **Caching:** Memoize expensive calculations (reaction rates, cross-sections)
3. **Profiling:** Identify bottlenecks before optimization (cProfile, line_profiler)
4. **Optional JIT:** Consider Numba for critical loops (83 processes)
5. **Memory Efficiency:** Avoid unnecessary copies of large arrays
6. **Algorithmic Complexity:** Maintain O(n) scaling with number of reactions

### Error Handling

1. **Custom Exceptions:**
   - `PlasmiumError` (base)
   - `ConfigurationError`
   - `ElementNotFoundError`
   - `ReactionError`
   - `IntegrationError`
   - `ConservationViolationError`

2. **Validation:** On all inputs, state transitions, and outputs

3. **Graceful Degradation:** For optional features (neutrino cooling, etc.)

4. **Informative Messages:** Include context, values, and suggestions

### Logging Strategy

1. **Use Python logging module** with hierarchical loggers
2. **Levels:**
   - DEBUG: Detailed internal state
   - INFO: Progress updates, milestones
   - WARNING: Non-critical issues
   - ERROR: Recoverable errors
   - CRITICAL: Unrecoverable errors

3. **Structured Logging:** JSON format for machine parsing
4. **Context:** Include timestep, temperature, key metrics

---

## Expected Outcomes

### Deliverables

1. ✅ Fully functional restructured `plasmium` package
2. ✅ Comprehensive test suite (>80% coverage)
3. ✅ User and developer documentation
4. ✅ Migration guide from old structure
5. ✅ CI/CD pipeline configuration
6. ✅ Performance benchmarks comparing old vs new
7. ✅ Example notebooks demonstrating usage
8. ✅ Docker container for reproducible runs

### Benefits

| Aspect | Before | After |
|--------|--------|-------|
| **Maintainability** | Difficult - monolithic, unclear dependencies | Easy - modular, clear interfaces |
| **Testability** | Nearly impossible - global state, side effects | Easy - isolated components, mockable |
| **Readability** | Poor - no docs, inconsistent naming | Excellent - self-documenting, typed |
| **Extensibility** | Hard - changes ripple through codebase | Easy - add new reactions without modifying existing |
| **Reliability** | Unknown - no tests | High - comprehensive testing catches regressions |
| **Collaboration** | Difficult - non-standard structure | Easy - familiar Python package structure |
| **Performance** | Unknown - no benchmarks | Measured - profiling-guided optimization |
| **Documentation** | Outdated PDF/LaTeX | Auto-generated API + living user guide |

### Metrics for Success

- ✅ Zero wildcard imports
- ✅ All public APIs have type hints and docstrings
- ✅ Test coverage >80%
- ✅ All original functionality preserved (verified by regression tests)
- ✅ Output matches original within numerical precision (1e-10 relative)
- ✅ New features can be added without modifying existing code (Open/Closed Principle)
- ✅ CI pipeline passes on every commit
- ✅ Documentation builds successfully
- ✅ No circular dependencies
- ✅ All modules <500 lines

---

## Risk Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Loss of functionality** | Medium | High | Comprehensive regression testing, feature checklist |
| **Performance degradation** | Medium | Medium | Profiling-guided optimization, maintain algorithmic equivalence, optional Numba JIT |
| **Breaking existing workflows** | Low | Medium | Backward-compatible I/O, migration scripts, deprecation warnings |
| **Scope creep** | High | Medium | Strict adherence to phased plan, MVP first, park nice-to-haves |
| **Knowledge loss** | Low | High | Detailed documentation, preserve original code until validation complete, comment translation |
| **Numerical divergence** | Medium | High | Frequent comparison with original, small timesteps initially, tolerance tuning |
| **Incomplete test coverage** | Medium | Medium | Coverage requirements in CI, prioritize critical paths |

---

## Functionality Checklist

Ensure all existing functionality is preserved:

### Physics Modules
- [ ] 48 isotopes tracked (n0 to P30)
- [ ] 12 unresonant reactions
- [ ] 2 resonant reactions
- [ ] 19 decay reactions
- [ ] 83 nuclear processes
- [ ] Electron/positron concentration tracking
- [ ] e⁻/e⁺ annihilation with energy release
- [ ] Temperature evolution (optional)
- [ ] Neutrino spectrum calculation
- [ ] Neutrino cooling (optional)
- [ ] Mean molecular weight calculation

### Numerical Methods
- [ ] Courant adaptive integration
- [ ] RK4 integration
- [ ] Adaptive timestep control
- [ ] Mass conservation renormalization

### I/O
- [ ] Simulation_input.txt parsing (11+ parameters)
- [ ] Elements_data.txt parsing (48 elements)
- [ ] Simulation_output.txt writing
- [ ] Logs.txt with timestamps
- [ ] Resonant_reactions_resonance_parameters.txt loading

### Visualization
- [ ] Element evolution plot (log-log)
- [ ] Temperature evolution overlay
- [ ] Interactive tooltips
- [ ] Neutrino spectrum visualization
- [ ] Post-processing with Parcer.py functionality

### Diagnostics
- [ ] Burning speed calculation
- [ ] Burning time calculation
- [ ] Minimum burning time identification
- [ ] Reaction rate tracking
- [ ] Equilibrium condition checking
- [ ] General condition display

### Configuration
- [ ] 9 tweakable parameters from Advanced_tweakables
- [ ] 15 physical constants from World_constants
- [ ] Integration method selection
- [ ] Output frequency control
- [ ] Temperature evolution toggle
- [ ] Annihilation toggle
- [ ] Neutrino cooling toggle

---

## Next Steps

To begin execution of this plan:

1. **Create the package structure** (Phase 1, Step 1.1)
   ```bash
   mkdir -p plasmium/{config,core,physics/{reactions,thermodynamics,particles,rates},io,analysis,utils,tests}
   touch plasmium/__init__.py plasmium/__main__.py
   # ... create all __init__.py files
   ```

2. **Implement constants and configuration** (Phase 1, Step 2.1)
   - Migrate World_constants.py → config/constants.py
   - Migrate Advanced_tweakables.py → config/settings.py

3. **Set up testing infrastructure** (Phase 5, Step 5.1) - do this early!
   ```bash
   pip install pytest pytest-cov
   # Create pyproject.toml with pytest configuration
   ```

4. **Begin incremental migration** following the Strangler Fig pattern

This plan preserves all 3,612 lines of existing functionality while transforming the codebase into a modern, maintainable Python package.
