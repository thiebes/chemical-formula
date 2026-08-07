# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

chemical-formula is a Node.js library that parses chemical formulas to count each element in a compound. The library exports a single function that takes a formula string like "H2O" or "C3H4OH(COOH)3" and returns an object with element counts.

## Development Commands

Test:
- `npm test` - Run all tests using tape
- `tape test/index.js` - Run tests directly

Lint:
- `npm run lint` - Check code style (uses @kenan/eslint-config)

Coverage:
- `npm run coverage` - Run tests with nyc coverage

Release:
- `npm run release` - Run semantic-release (automated versioning)

## Core Architecture

The main parsing logic is in [index.js](index.js) as a single exported function with nested helper functions. The parser uses a recursive descent approach with depth tracking:

### Main Functions

1. **strictParseInt(value)** - Validates integer strings with strict regex checking before parsing. Only accepts valid integers, not malformed numeric strings.

2. **getAtomicNumber(symbol)** - Validates element symbols against the chemical-symbols package using lodash.indexof. Returns -1 for unknown elements.

3. **chemicalFormula(formula)** - Main parser with nested helper functions:
   - **addElement(element, count)** - Accumulates element counts, summing if element already exists
   - **parseGroup(str, multiplier)** - Recursive parser handling elements, subscripts, and parentheses

### Parsing Strategy

1. **Element Symbol Recognition**: Parses uppercase letter followed by any lowercase letters. Two-character symbols (like "Br" or "Ca") are automatically detected. Validates each symbol via getAtomicNumber().

2. **Parenthesis Handling**: Uses depth-tracking counter to find matching closing parenthesis. Handles nested parentheses by incrementing/decrementing depth. Extracts group content and recursively calls parseGroup() with cascaded multiplier.

3. **Subscript Processing**: Parses digit strings following elements or closing parentheses using strictParseInt(). Validates with isFinite() to ensure valid positive integers.

4. **Multiplier Cascading**: Critical feature for nested structures. When parsing (SO4)3, the parseGroup function receives multiplier=3, so each element count is multiplied: O gets 4 * 3 = 12. For doubly nested structures like Mg3(Fe(CN)6)2, multipliers cascade: C gets 1 * 2 * 6 = 12.

5. **Result Accumulation**: Maintains plain object with element symbols as keys. Uses addElement() to sum counts as they're discovered, enabling single-pass parsing.

### Error Handling

- **Unmatched parentheses**: Detected via depth counter validation
- **Unknown elements**: Thrown when getAtomicNumber() returns -1
- **Invalid subscripts**: Validated with strictParseInt() and isFinite()
- **Invalid characters**: Caught by character type testing (must be A-Z, a-z, 0-9, or parentheses)
- **Leading digits**: Rejected as invalid characters (e.g., "2O" throws error)

## Test Structure

Tests are in [test/index.js](test/index.js) using tape. Five test suites with 40+ test cases:

1. **Function Export Validation** (1 test)
   - Validates main export is a function

2. **Common Organic Compounds** (10 tests)
   - Real chemical formulas: citric acid, ethylene glycol, histidine, water, etc.
   - Tests complex element counts and multi-character symbols
   - Example: C3H4OH(COOH)3 → {C: 6, H: 8, O: 7}

3. **Nested Subscripts in Parentheses** (20 tests)
   - Critical test suite for multiplier cascading functionality
   - Metal sulfates: Al2(SO4)3, Fe2(SO4)3 (tests subscript multiplication)
   - Hydrated complexes: Fe(H2O)6, Cu(NH3)4
   - Hydroxides: Ca(OH)2, Mg(OH)2, Al(OH)3
   - Nitrates: Mg(NO3)2, Ca(NO3)2, Ba(NO3)2
   - Phosphates: Ca3(PO4)2
   - Edge cases: Parentheses without explicit multiplier
   - Doubly nested: Mg3(Fe(CN)6)2 (tests cascading multipliers)

4. **Error Handling** (5 tests)
   - Unmatched opening/closing parentheses
   - Empty strings
   - Invalid characters (!, @) at different positions
   - Specific error message validation

5. **Invalid Formulas** (9 tests)
   - Leading digits: 0C, 2O, 13Li
   - Digits before parentheses: 2(NO3)
   - Subscripts on parentheses: H(2), Ba(12), Cr(5)3
   - Multi-digit invalid subscripts: Pb(13)2, Au(22)11

When adding tests, follow the existing pattern of arrays with [formula, expectedResult, description] tuples. For error tests, use t.throws() with regex to match expected error messages.

## Dependencies

### Production Dependencies
- **chemical-symbols** (^3.0.0) - Provides periodic table data for element validation
- **lodash.indexof** (^4.0.5) - Array indexOf utility for element symbol lookup
- **lodash.isfinite** (^3.3.2) - Validates parsed numeric values

### Development Dependencies
- **tape** (^5.9.0) - TAP-compatible testing framework
- **nyc** (^17.1.0) - Code coverage reporting
- **eslint** (^8.57.1) with **@kenan/eslint-config** (^11.1.18) - Code linting
- **semantic-release** (^24.2.9) - Automated versioning and publishing
- Test helpers: lodash.foreach, lodash.isfunction

## CI/CD

The project uses GitHub Actions with two workflows:

### nodejs.yml (CI Pipeline)
Runs on push, pull requests, and merge groups:
- Matrix: Node.js 18 and 20 on ubuntu-latest
- Steps: Setup Node, update npm, install dependencies, run coverage and linting
- No lockfile: Dependencies installed without package-lock.json

### release.yml (Release Pipeline)
Runs on master branch pushes only:
- Node.js 22.20.0 on ubuntu-latest
- Runs semantic-release with GITHUB_TOKEN and NPM_TOKEN
- Automatic versioning, changelog generation, and npm publishing

## Release Process

This project uses semantic-release with conventional commits. The [.releaserc.json](.releaserc.json) configuration automatically:
- Analyzes commits for version bumps
- Generates [CHANGELOG.md](CHANGELOG.md)
- Creates GitHub releases
- Publishes to npm
- Commits release artifacts

Commit messages must follow conventional commits format (feat:, fix:, etc.) for proper versioning.
