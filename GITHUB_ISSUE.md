# GitHub Issue for chemical-formula

Submit this to: https://github.com/kenany/chemical-formula/issues

---

**Title**: Bug: Incorrect element counts for nested subscripts in parentheses (e.g., Al2(SO4)3)

**Body**:

## Description

The library incorrectly calculates element counts when chemical formulas contain subscripts within parentheses that are then multiplied by an external subscript. The internal subscripts are not properly multiplied by the parenthetical multiplier.

## Steps to Reproduce

```javascript
const chemicalFormula = require('chemical-formula');

// Test case 1: Aluminum sulfate
console.log(chemicalFormula('Al2(SO4)3'));
// Current output: {Al: 2, S: 3, O: 3}
// Expected output: {Al: 2, S: 3, O: 12}

// Test case 2: Iron hexahydrate
console.log(chemicalFormula('Fe(H2O)6'));
// Current output: {Fe: 1, H: 6, O: 6}
// Expected output: {Fe: 1, H: 12, O: 6}

// Test case 3: Iron hydrate without explicit multiplier
console.log(chemicalFormula('Fe(H2O)'));
// Current output: {Fe: 1, H: null}
// Expected output: {Fe: 1, H: 2, O: 1}
```

## Expected Behavior

When a subscript appears inside parentheses, it should be multiplied by the parenthetical multiplier:
- In `(SO4)3`: The `O4` becomes `O12` (4 × 3)
- In `(H2O)6`: The `H2` becomes `H12` (2 × 6) and `O` becomes `O6` (1 × 6)
- In `(H2O)` with no explicit multiplier: Treat as `(H2O)1`

## Chemical Accuracy Reference

These test cases can be verified against standard chemistry references:
- Al2(SO4)3 (aluminum sulfate): 2 Al + 3 S + 12 O
- Fe(H2O)6 (iron hexahydrate): 1 Fe + 12 H + 6 O
- Ca(OH)2 (calcium hydroxide): 1 Ca + 2 O + 2 H
- Mg(NO3)2 (magnesium nitrate): 1 Mg + 2 N + 6 O

## Impact

This bug affects many common chemical compounds including:
- All metal sulfates: Al2(SO4)3, Fe2(SO4)3, Cr2(SO4)3, K2SO4
- Hydrated metal complexes: Fe(H2O)6, Co(H2O)6, Cu(NH3)4
- Hydroxides: Ca(OH)2, Mg(OH)2, Al(OH)3, Ba(OH)2
- Nitrates: Mg(NO3)2, Ca(NO3)2, Ba(NO3)2
- Phosphates: Ca3(PO4)2

This makes the library unsuitable for chemistry education, stoichiometry calculations, and molecular weight calculations.

## Environment

- chemical-formula version: 4.0.1
- Node.js version: 22.17.0
- Platform: win32

## Proposed Solution

I have a working fix that correctly handles nested subscripts in parentheses through recursive parsing. I would be happy to submit a pull request with:
1. The corrected parsing logic
2. Comprehensive test cases covering the bug scenarios
3. Additional tests for edge cases (nested parentheses, large subscripts, etc.)

Would you be open to reviewing a PR for this fix?
