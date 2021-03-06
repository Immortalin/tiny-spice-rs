# Changelog

## [UNRELEASED]

### Added
- Toplevel program `tiny-spice` 
- SPICE deck reader
 * Components: I, V, R, D, C

### Changed
- Configuration object to hold parameters for analyses


### Fixed
- Sort test_results.log


## [0.6.0] Capacitors
Deal with capacitors

### Added
- Capacitor model for DC operating point analysis
- Integration method - backward euler built into cap linearisation

### Changed
- Analysis functions return stats
- Use Colon & Nagel method to limit the diode overvoltaging
- Diode model now uses ::new() method for instantiations
- `f32`s now `f64`s. This helps convergence A LOT

### Fixed
- Convergence problems with diode networks



## [0.5.0] Transient Analysis
Added iteration count based transient analysis.

### Added
- Iteration count based time marching algorithm form SPICE2
- Sinusoidal current source
- Write names, units & waveform data to a file - to be read with KST
- Transient analysis works with the diode bridge circuit!

### Changed
- Extracted the convergence test to a seperate function

### Fixed


## [0.4.0] Non-Linear Solver
DC operating point of simple circuits with: non-linear diode model; current 
sources and resistors.

### Added
- Diode model (forward biased)
- Newton-Raphson solver

### Changed
- Added proper testing for different circuits


## [0.3.0] Gaussian Elimination with Partial Pivot
Use Guassian Elimination with partial pivot as in Wikipedia algorithm. This
should help with numerical stability.

### Changed
- Gaussian Elimination now works from the column with the biggest `abs()`.
  This helps with numerical stability, apparently.


## [0.2.0] Modified Nodal Analysis
Can handle independent voltage sources now by incorporating Modified Nodal
Anaysis (MNA) techniques. Solver didn't have to be changed!

### Added 
- Circuit builder: voltage sources
- Started Glossary

### Changed
- Matrix now constructed in MNA format


## [0.1.0] DC Operating Point
Can find DC operating point of simple circuits consisting of current sources and
resistors.

### Added 
- Circuit builder: resistors and current sources
- Naive Guassian Elimination
- Back-Substitution

