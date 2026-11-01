# Development and Integration of Lemke's Algorithm Package

This is a final report for Google Summer of Code 2026.
- Contributor: Natalie Mesablishvili
- Mentors: Rahul Savani, Bernhard von Stengel
- Organization: [Gambit: The package for computation in game theory](https://www.gambit-project.org/)


## Project Overview

Lemke's algorithm is a method for solving Linear Complementarity Problems (LCPs).
Computing equilibria of two-player games can be reduced to solving such problems,
which makes Lemke's algorithm important to game-theory software.

The [Gambit library](https://gambitproject.readthedocs.io/en/stable/)
already has a C++ implementation of Lemke's algorithm.
This [Google Summer of Code 2026 project](https://summerofcode.withgoogle.com/programs/2026/projects/LYQiX01q)
focused on improving and integrating Bernhard von Stengel's
[***lemke*** package](https://github.com/gambitproject/lemke),
which is implemented entirely in pure Python.

Previously, ***lemke*** worked primarily as a standalone command-line tool.
Algorithm logic, file I/O and console output were closely coupled.
As a result, there was no clean API to call programmatically,
so the package could not easily be used as a library.

The goal was to turn ***lemke*** into a dual-purpose tool: usable from
the command line, and importable as a Python library that Gambit can call directly.
For example, Gambit should be able to run `Lcp.from_file("lcp.txt").run_lemke()` and
receive a result object to process further, rather than just having results printed to the console.


## My Contributions

The work was organized into three areas:

### 1. Core Algorithms and Python API
This area aimed to restructure the core logic into a clean, reusable and well-defined API
for solving LCPs and games, so the package could function as an importable library.

#### Main Changes
- `Lcp` and `Bimatrix` classes now directly accept data, instead of being initialized from a file.
  Reading from a file is handled by a separate class method `from_file()`.
  Also, redundant `__init__()` methods were removed.
- Added a callback class to observe algorithm execution in `lemke.py`,
  so that the destination and level of detail for printed output
  are decided by the caller, instead of global variables.
- Rounding/precision bugs were fixed when reading decimals from a file.
- A standardized result class for LCPs `LcpResult` was introduced,
  which is returned when Lemke's algorithm finishes computation.
  The result object encapsulates the solution and its details,
  such as the pivot count and the variables in the basis.
  When no solution is found, this object provides the last
  calculated values reached by the algorithm.
  Library users can call a solver and decide for themselves how to handle the result.
  Also, it was ensured that no pivoting is performed in case of a trivial LCP.

#### Merged Pull Requests
- [Refactor lcp & bimatrix initialization (#15)](https://github.com/gambitproject/lemke/pull/15)
- [Introduce callback functions for printing in `lemke.py` (#17)](https://github.com/gambitproject/lemke/pull/17)
- [Add a result class for LCPs (#18)](https://github.com/gambitproject/lemke/pull/18)
- [Correct and update how numbers are parsed from a file (#21)](https://github.com/gambitproject/lemke/pull/21)
- [Fix how `runlemke()` failure is checked in `runLH()` and `runtrace()` (#27)](https://github.com/gambitproject/lemke/pull/27)



### 2. Command-Line Interface
Command-line arguments were previously handled with global variables and manual parsing of `sys.argv`.
This was replaced by the `Click` library, which provides automatic help messages and input validation.
The values parsed by Click are now passed to the appropriate library methods,
which makes the CLI a thin layer over the library.

#### Merged Pull Requests
- [Use Click to parse command-line arguments in `lemke.py` (#10)](https://github.com/gambitproject/lemke/pull/10)
- [Use Click to parse command-line arguments in `randomstart.py` (#12)](https://github.com/gambitproject/lemke/pull/12)
- [Use Click to parse command-line arguments in `bimatrix.py` (#16)](https://github.com/gambitproject/lemke/pull/16)


### 3. Reliability and Maintainability
This area made the package more reliable and easier to maintain through automated testing,
structured error handling, documentation, and clean code style.

#### Main Changes
- A test suite was added which includes both unit tests and correctness tests for algorithms.
- A [readthedocs page](https://gambitproject.readthedocs.io/projects/lemke/en/latest/) 
  was created for the documentation which includes installation instructions, API and CLI references,
  input file formats, usage tutorials, and some information on how to contribute to the development of *lemke*.
  API reference is automatically generated from the added docstrings, so that it never drifts from the code.
  And, similarly, CLI reference is automatically generated from the Click commands.
- GitHub Actions workflow was added which automatically runs `flake8` and `ruff` for linting,
  and also tests (including doctests) on any push or pull request.
- Old error-handling by printing a message and calling `exit(1)` was replaced with Python exceptions
  (such as `ValueError`, `RuntimeError`) that callers can catch and handle appropriately.
- Classes, functions, and variables were renamed to match Python conventions and type hints were added.


#### Merged Pull Requests
- [Add GitHub Actions workflow for linting and tests (#11)](https://github.com/gambitproject/lemke/pull/11)
- [Remove requirements.txt (#13)](https://github.com/gambitproject/lemke/pull/13)
- [Add initial test suite (#8)](https://github.com/gambitproject/lemke/pull/8)
- [Update tests for pygambit 16.7 compatibility (#14)](https://github.com/gambitproject/lemke/pull/14)
- [Add docstrings to `lemke.py` and `bimatrix.py` (#19)](https://github.com/gambitproject/lemke/pull/19)
- [Add docstrings to `randomstart.py` (#20)](https://github.com/gambitproject/lemke/pull/20)
- [Update the front page in docs (#28)](https://github.com/gambitproject/lemke/pull/28)
- [Add file formats page in docs (#29)](https://github.com/gambitproject/lemke/pull/29)
- [Add library usage tutorials in docs (#30)](https://github.com/gambitproject/lemke/pull/30)
- [Add a contributing guide to the docs (#31)](https://github.com/gambitproject/lemke/pull/31)
- [Add CLI documentation (#32)](https://github.com/gambitproject/lemke/pull/32)
- [Update the documentation (#33)](https://github.com/gambitproject/lemke/pull/33)
- [Add type hints and follow conventional Python naming (#34)](https://github.com/gambitproject/lemke/pull/34)


## Future Work
- *lemke* package can be integrated with Gambit to provide an alternative to the existing C++ implementation.
- Callback-based tracing of algorithms and standardized result objects should be added
  beyond `lemke.py`. For bimatrix games, the result object should be designed with Gambit's
  [NashComputationResult](https://gambitproject.readthedocs.io/en/stable/api/pygambit.nash.NashComputationResult.html)
  object in mind, since when integrating *lemke* to Gambit, *lemke*'s result object has to be converted to `NashComputationResult` object.
- Error-handling and testing can be further improved by covering more edge cases.
- In addition to algorithm correctness and unit tests, performance tests can be added to measure speed
  on large problems/games, including runtime comparisons between Gambit’s existing methods
  and the integrated *lemke* library methods.
- Support for polymatrix games can be added (with CLI via Click, callbacks, result objects, etc.).
- Current tests already include solving extensive-form games (`tests/sequence_form_helper.py`).
  This code can be updated and moved into the package to add support for extensive-form games.
- Other algorithms that use Lemke's algorithm, or are related to it could also be added to the package.
