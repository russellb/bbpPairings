# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BBP Pairings is a Swiss-system chess tournament pairing engine that implements FIDE's Systems of Pairings and Programs Commission rules. It supports the Dutch system (2017 rules) and includes a flawed implementation of the Burstein system.

## Build Commands

```bash
# Clean build
make clean

# Build the main executable
make
# Or specify compiler (gcc/clang)
make COMP=gcc
make COMP=clang

# Build with debug symbols
make debug=yes optimize=no

# Run tests
make test

# Create distribution package
make dist

# Build options
make burstein=no     # Exclude Burstein system
make dutch=no        # Exclude Dutch system
make static=yes      # Static linking
```

The executable is named `bbpPairings.exe` regardless of platform.

## Command Line Usage

```bash
# Check tournament validity
bbpPairings.exe [--dutch|--burstein] input.trf -c

# Pair next round
bbpPairings.exe [--dutch|--burstein] input.trf -p [output.trf]

# Generate random tournament
bbpPairings.exe [--dutch|--burstein] -g [config.txt] -o output.trf [-s seed]
bbpPairings.exe [--dutch|--burstein] model.trf -g -o output.trf [-s seed]

# With checklist output
bbpPairings.exe [--dutch|--burstein] input.trf -p -l [checklist.list]
```

## Architecture

### Core Components

**Tournament Engine (`src/tournament/`):**
- `tournament.h/cpp`: Core tournament data structures (Player, Match, Tournament)
- `generator.cpp`: Random tournament generation logic
- `checker.cpp`: Tournament validation and consistency checking

**Swiss Systems (`src/swisssystems/`):**
- `common.h/cpp`: Base classes and shared functionality (SwissSystem, Pairing, Info)
- `dutch.cpp`: Dutch system implementation (2017 FIDE rules)
- `burstein.cpp`: Burstein system implementation (incomplete)

**Matching Algorithm (`src/matching/`):**
- `computer.cpp`: Weighted matching algorithm implementation
- Uses Galil-Micali-Gabow O(EV log V) algorithm for maximum weighted matching
- `detail/`: Graph structures, blossom handling for matching

**File Formats (`src/fileformats/`):**
- `trf.cpp`: TRF(bx) format parser and writer - extended TRF format
- `generatorconfiguration.cpp`: Configuration file parser for random generation

### TRF(bx) Format Extensions

Beyond standard TRF(x), BBP Pairings adds:
- `BBW`, `BBD`, `BBL`, `BBZ`, `BBF`, `BBU`: Custom point values for wins/draws/losses/byes
- `XXC`: Forbidden pairings specification
- `XXZ`: Player withdrawal markers
- Result codes: `0` (half-point bye), `-` (full-point bye), `Z` (zero-point bye)

### Key Design Patterns

1. **Swiss System Abstraction**: Factory pattern via `swisssystems::getInfo()` returns system-specific implementations
2. **Tournament State Management**: Tournament object maintains complete state including players, matches, accelerations, forbidden pairs
3. **Weighted Matching**: Core pairing algorithm treats pairing as graph matching problem with weights
4. **Build Configuration**: Compile-time feature flags (`OMIT_DUTCH`, `OMIT_BURSTEIN`) for custom builds

### Cross-Platform Support

- Supports Linux, macOS, and Windows (MinGW)
- GitHub Actions CI for Linux (gcc/clang) and macOS (clang)
- Platform detection in Makefile handles compiler differences
- macOS: Special handling for Apple clang vs GCC

### Testing

Tests are in `test/tests/` and use a custom framework:
```bash
# Run all tests
make test

# Test framework automatically discovers and runs all test files
# Tests validate specific edge cases (e.g., issue_7.cpp, issue_15.cpp)
```

## Important Implementation Details

- Player IDs are 1-indexed in TRF files
- Scores are stored as integers (10x actual score) for precision
- Maximum limits configurable at compile time (`MAX_PLAYERS`, `MAX_ROUNDS`, etc.)
- Default acceleration for Burstein system unless overridden with XXA codes
- Initial piece color must be specified or inferred from first round
- Error codes: 1 (no valid pairing), 2 (unexpected error), 3 (invalid request), 4 (limit exceeded), 5 (file error)