# Code for chemical-formula PR

This document contains the exact code to be submitted in the pull request, ready to copy and paste.

## Modified index.js

This is the complete replacement for `index.js` in the chemical-formula repository:

```javascript
'use strict';

const symbols = require('chemical-symbols');
const isFinite = require('lodash.isfinite');
const indexOf = require('lodash.indexof');

function strictParseInt(value) {
  if (/^(-|\+)?([0-9]+|Infinity)$/.test(value)) {
    return Number(value);
  }
  return NaN;
}

function getAtomicNumber(symbol) {
  const index = indexOf(symbols, symbol);
  return index > -1 ? index + 1 : -1;
}

function chemicalFormula(formula) {
  if (!formula || typeof formula !== 'string') {
    throw new Error('Invalid chemical formula');
  }

  const elements = {};

  function addElement(element, count) {
    elements[element] = (elements[element] || 0) + count;
  }

  function parseGroup(str, multiplier) {
    if (typeof multiplier === 'undefined') {
      multiplier = 1;
    }

    let i = 0;

    while (i < str.length) {
      const char = str.charAt(i);

      if (char === '(') {
        // Find matching closing parenthesis
        let depth = 1;
        let j = i + 1;
        while (j < str.length && depth > 0) {
          if (str.charAt(j) === '(') {
            depth++;
          }
          if (str.charAt(j) === ')') {
            depth--;
          }
          j++;
        }

        if (depth !== 0) {
          throw new Error('Unmatched parentheses in formula');
        }

        // Get the group content
        const groupContent = str.substring(i + 1, j - 1);
        i = j;

        // Parse the number after the closing parenthesis
        let numStr = '';
        while (i < str.length && /\d/.test(str.charAt(i))) {
          numStr += str.charAt(i);
          i++;
        }

        const groupMultiplier = numStr ? strictParseInt(numStr) : 1;
        if (!isFinite(groupMultiplier) || groupMultiplier < 1) {
          throw new Error('Invalid subscript in formula');
        }

        // Recursively parse the group with cascaded multiplier
        parseGroup(groupContent, multiplier * groupMultiplier);
      } else if (/[A-Z]/.test(char)) {
        // Element symbol (starts with uppercase)
        let element = char;
        i++;

        // Check for lowercase letter(s) following
        while (i < str.length && /[a-z]/.test(str.charAt(i))) {
          element += str.charAt(i);
          i++;
        }

        // Validate element symbol
        if (getAtomicNumber(element) === -1) {
          throw new Error('Unknown element: ' + element);
        }

        // Parse the subscript number
        let numStr = '';
        while (i < str.length && /\d/.test(str.charAt(i))) {
          numStr += str.charAt(i);
          i++;
        }

        const count = numStr ? strictParseInt(numStr) : 1;
        if (!isFinite(count) || count < 1) {
          throw new Error('Invalid subscript in formula');
        }

        // Add element with count multiplied by cascaded multiplier
        addElement(element, count * multiplier);
      } else {
        throw new Error('Invalid character in formula: ' + char);
      }
    }
  }

  parseGroup(formula);

  return elements;
}

module.exports = chemicalFormula;
```

## Test Cases to Add to test/index.js

Add this new test suite to `test/index.js`. Insert it after the existing "common organic compounds" test:

```javascript
test('nested subscripts in parentheses', function(t) {
  const COMPOUNDS = [
    // Metal sulfates - critical bug cases
    ['Al2(SO4)3', { Al: 2, S: 3, O: 12 }, 'aluminum sulfate'],
    ['Fe2(SO4)3', { Fe: 2, S: 3, O: 12 }, 'iron(III) sulfate'],
    ['Cr2(SO4)3', { Cr: 2, S: 3, O: 12 }, 'chromium(III) sulfate'],
    ['K2SO4', { K: 2, S: 1, O: 4 }, 'potassium sulfate (baseline - no parens)'],

    // Hydrated complexes - critical bug cases
    ['Fe(H2O)6', { Fe: 1, H: 12, O: 6 }, 'iron hexahydrate'],
    ['Co(H2O)6', { Co: 1, H: 12, O: 6 }, 'cobalt hexahydrate'],
    ['Cu(NH3)4', { Cu: 1, N: 4, H: 12 }, 'tetraamminecopper(II)'],
    ['Fe(CN)6', { Fe: 1, C: 6, N: 6 }, 'hexacyanoferrate'],

    // Hydroxides - common compounds
    ['Ca(OH)2', { Ca: 1, O: 2, H: 2 }, 'calcium hydroxide'],
    ['Mg(OH)2', { Mg: 1, O: 2, H: 2 }, 'magnesium hydroxide'],
    ['Al(OH)3', { Al: 1, O: 3, H: 3 }, 'aluminum hydroxide'],
    ['Ba(OH)2', { Ba: 1, O: 2, H: 2 }, 'barium hydroxide'],

    // Nitrates - common compounds
    ['Mg(NO3)2', { Mg: 1, N: 2, O: 6 }, 'magnesium nitrate'],
    ['Ca(NO3)2', { Ca: 1, N: 2, O: 6 }, 'calcium nitrate'],
    ['Ba(NO3)2', { Ba: 1, N: 2, O: 6 }, 'barium nitrate'],

    // Phosphates
    ['Ca3(PO4)2', { Ca: 3, P: 2, O: 8 }, 'calcium phosphate'],

    // Edge cases - parentheses without explicit multiplier
    ['Fe(H2O)', { Fe: 1, H: 2, O: 1 }, 'iron hydrate (no multiplier defaults to 1)'],
    ['Ca(OH)', { Ca: 1, O: 1, H: 1 }, 'calcium hydroxyl (no multiplier)'],

    // Nested parentheses
    ['Mg3(Fe(CN)6)2', { Mg: 3, Fe: 2, C: 12, N: 12 }, 'magnesium ferrocyanide (doubly nested)']
  ];

  t.plan(COMPOUNDS.length);

  forEach(COMPOUNDS, function(fixture) {
    t.deepEqual(chemicalFormula(fixture[0]), fixture[1], fixture[2]);
  });
});

test('error handling for invalid formulas', function(t) {
  t.plan(5);

  t.throws(function() {
    chemicalFormula('Ca(OH');
  }, /Unmatched parentheses/, 'unmatched opening parenthesis');

  t.throws(function() {
    chemicalFormula('CaOH)');
  }, /Invalid character/, 'unmatched closing parenthesis');

  t.throws(function() {
    chemicalFormula('');
  }, /Invalid chemical formula/, 'empty string');

  t.throws(function() {
    chemicalFormula('H2O!');
  }, /Invalid character/, 'invalid character at end');

  t.throws(function() {
    chemicalFormula('H2@O');
  }, /Invalid character/, 'invalid character in middle');
});
```

Make sure to add this at the top of `test/index.js` if not already present:

```javascript
const forEach = require('lodash.foreach');
```

## Complete test/index.js File

For reference, here's what the complete `test/index.js` should look like after modifications:

```javascript
'use strict';

const test = require('tape');
const isFunction = require('lodash.isfunction');
const forEach = require('lodash.foreach');

const chemicalFormula = require('../');

test('exports a function', function(t) {
  t.plan(1);
  t.ok(isFunction(chemicalFormula));
});

test('common organic compounds', function(t) {
  const COMPOUNDS = [
    ['C19H29COOH', { C: 20, H: 30, O: 2 }, 'abietic acid'],
    ['C12H10', { C: 12, H: 10 }, 'acenaphthene'],
    ['C12H6O2', { C: 12, H: 6, O: 2 }, 'acenaphthoquinone'],
    ['C6H5Br', { C: 6, H: 5, Br: 1 }, 'bromobenzene'],
    ['C3H4OH(COOH)3', { C: 6, H: 8, O: 7 }, 'citric acid'],
    ['HOCH2CH2OH', { H: 6, O: 2, C: 2 }, 'ethylene glycol'],
    ['C5H11NO2', { C: 5, H: 11, N: 1, O: 2 }, 'ethylene glycol'],
    ['CH3CH(CH3)CH3', { C: 4, H: 10 }, '2-methylpropene'],
    ['NH2CH(C4H5N2)COOH', { N: 3, H: 9, C: 6, O: 2 }, 'histidine'],
    ['H2O', { H: 2, O: 1 }, 'water']
  ];

  t.plan(COMPOUNDS.length);

  forEach(COMPOUNDS, function(fixture) {
    t.deepEqual(chemicalFormula(fixture[0]), fixture[1], fixture[2]);
  });
});

test('nested subscripts in parentheses', function(t) {
  const COMPOUNDS = [
    // Metal sulfates - critical bug cases
    ['Al2(SO4)3', { Al: 2, S: 3, O: 12 }, 'aluminum sulfate'],
    ['Fe2(SO4)3', { Fe: 2, S: 3, O: 12 }, 'iron(III) sulfate'],
    ['Cr2(SO4)3', { Cr: 2, S: 3, O: 12 }, 'chromium(III) sulfate'],
    ['K2SO4', { K: 2, S: 1, O: 4 }, 'potassium sulfate (baseline - no parens)'],

    // Hydrated complexes - critical bug cases
    ['Fe(H2O)6', { Fe: 1, H: 12, O: 6 }, 'iron hexahydrate'],
    ['Co(H2O)6', { Co: 1, H: 12, O: 6 }, 'cobalt hexahydrate'],
    ['Cu(NH3)4', { Cu: 1, N: 4, H: 12 }, 'tetraamminecopper(II)'],
    ['Fe(CN)6', { Fe: 1, C: 6, N: 6 }, 'hexacyanoferrate'],

    // Hydroxides - common compounds
    ['Ca(OH)2', { Ca: 1, O: 2, H: 2 }, 'calcium hydroxide'],
    ['Mg(OH)2', { Mg: 1, O: 2, H: 2 }, 'magnesium hydroxide'],
    ['Al(OH)3', { Al: 1, O: 3, H: 3 }, 'aluminum hydroxide'],
    ['Ba(OH)2', { Ba: 1, O: 2, H: 2 }, 'barium hydroxide'],

    // Nitrates - common compounds
    ['Mg(NO3)2', { Mg: 1, N: 2, O: 6 }, 'magnesium nitrate'],
    ['Ca(NO3)2', { Ca: 1, N: 2, O: 6 }, 'calcium nitrate'],
    ['Ba(NO3)2', { Ba: 1, N: 2, O: 6 }, 'barium nitrate'],

    // Phosphates
    ['Ca3(PO4)2', { Ca: 3, P: 2, O: 8 }, 'calcium phosphate'],

    // Edge cases - parentheses without explicit multiplier
    ['Fe(H2O)', { Fe: 1, H: 2, O: 1 }, 'iron hydrate (no multiplier defaults to 1)'],
    ['Ca(OH)', { Ca: 1, O: 1, H: 1 }, 'calcium hydroxyl (no multiplier)'],

    // Nested parentheses
    ['Mg3(Fe(CN)6)2', { Mg: 3, Fe: 2, C: 12, N: 12 }, 'magnesium ferrocyanide (doubly nested)']
  ];

  t.plan(COMPOUNDS.length);

  forEach(COMPOUNDS, function(fixture) {
    t.deepEqual(chemicalFormula(fixture[0]), fixture[1], fixture[2]);
  });
});

test('error handling for invalid formulas', function(t) {
  t.plan(5);

  t.throws(function() {
    chemicalFormula('Ca(OH');
  }, /Unmatched parentheses/, 'unmatched opening parenthesis');

  t.throws(function() {
    chemicalFormula('CaOH)');
  }, /Invalid character/, 'unmatched closing parenthesis');

  t.throws(function() {
    chemicalFormula('');
  }, /Invalid chemical formula/, 'empty string');

  t.throws(function() {
    chemicalFormula('H2O!');
  }, /Invalid character/, 'invalid character at end');

  t.throws(function() {
    chemicalFormula('H2@O');
  }, /Invalid character/, 'invalid character in middle');
});

// Keep any other existing tests below this point
```

## Commit Message

Use this exact commit message:

```
Fix nested subscripts in parentheses multiplication

The parser now correctly handles subscripts within parentheses that have
external multipliers. Previously, formulas like Al2(SO4)3 would return
{Al:2, O:3, S:3} but now correctly return {Al:2, S:3, O:12}.

The fix implements recursive parsing with proper multiplier cascading,
ensuring that subscripts inside parentheses are multiplied by the
parenthetical multiplier.

Fixes #287
```

Replace `[ISSUE_NUMBER]` with the actual issue number.

## Manual Testing Script

Use this to manually verify the fix works:

```bash
node -e "
const cf = require('./index.js');

console.log('=== Critical Bug Cases ===');
console.log('Al2(SO4)3:', JSON.stringify(cf('Al2(SO4)3')));
console.log('Expected:  {\"Al\":2,\"S\":3,\"O\":12}');
console.log('');

console.log('Fe(H2O)6:', JSON.stringify(cf('Fe(H2O)6')));
console.log('Expected: {\"Fe\":1,\"H\":12,\"O\":6}');
console.log('');

console.log('Fe(H2O):', JSON.stringify(cf('Fe(H2O)')));
console.log('Expected: {\"Fe\":1,\"H\":2,\"O\":1}');
console.log('');

console.log('=== Other Common Compounds ===');
console.log('Ca(OH)2:', JSON.stringify(cf('Ca(OH)2')));
console.log('Mg(NO3)2:', JSON.stringify(cf('Mg(NO3)2')));
console.log('Ca3(PO4)2:', JSON.stringify(cf('Ca3(PO4)2')));
console.log('');

console.log('=== Nested Parentheses ===');
console.log('Mg3(Fe(CN)6)2:', JSON.stringify(cf('Mg3(Fe(CN)6)2')));
console.log('Expected:       {\"Mg\":3,\"Fe\":2,\"C\":12,\"N\":12}');
console.log('');

console.log('=== Baseline (should still work) ===');
console.log('H2O:', JSON.stringify(cf('H2O')));
console.log('NaCl:', JSON.stringify(cf('NaCl')));
console.log('H2SO4:', JSON.stringify(cf('H2SO4')));
"
```

Expected output:

```
=== Critical Bug Cases ===
Al2(SO4)3: {"Al":2,"S":3,"O":12}
Expected:  {"Al":2,"S":3,"O":12}

Fe(H2O)6: {"Fe":1,"H":12,"O":6}
Expected: {"Fe":1,"H":12,"O":6}

Fe(H2O): {"Fe":1,"H":2,"O":1}
Expected: {"Fe":1,"H":2,"O":1}

=== Other Common Compounds ===
Ca(OH)2: {"Ca":1,"O":2,"H":2}
Mg(NO3)2: {"Mg":1,"N":2,"O":6}
Ca3(PO4)2: {"Ca":3,"P":2,"O":8}

=== Nested Parentheses ===
Mg3(Fe(CN)6)2: {"Mg":3,"Fe":2,"C":12,"N":12}
Expected:       {"Mg":3,"Fe":2,"C":12,"N":12}

=== Baseline (should still work) ===
H2O: {"H":2,"O":1}
NaCl: {"Na":1,"Cl":1}
H2SO4: {"H":2,"S":1,"O":4}
```

## Dependencies Check

Verify the new code only uses existing dependencies:

```bash
# Check what we're using
grep "require(" index.js

# Should output:
# const symbols = require('chemical-symbols');
# const isFinite = require('lodash.isfinite');
# const indexOf = require('lodash.indexof');

# Note: We removed lodash.forown since it's no longer needed
```

The new implementation actually removes a dependency (`lodash.forown`) since we don't need it with the recursive approach. This is a bonus - fewer dependencies!

## Pre-Submit Checklist

Before submitting the PR, verify:

```bash
# 1. Tests pass
npm test
# Expected: All tests pass (original + new ones)

# 2. Linter passes
npm run lint
# Expected: No errors

# 3. Coverage is good
npm run coverage
# Expected: High coverage percentage

# 4. Manual tests work
# Run the manual testing script above
# Expected: All outputs match expected values

# 5. No unintended changes
git diff index.js
git diff test/index.js
# Review carefully - only the intended changes should be present
```

## Ready to Submit

Once all checks pass:

1. Stage files: `git add index.js test/index.js`
2. Commit: `git commit -m "[message from above]"`
3. Push: `git push origin fix/nested-subscripts-in-parentheses`
4. Create PR on GitHub
5. Use PR description from CONTRIBUTION_PLAN.md Step 8.1

## Notes

- The new implementation is simpler and more maintainable than the original
- It uses fewer dependencies (removed lodash.forown)
- It handles nested parentheses naturally through recursion
- All existing functionality is preserved
- Error handling is improved with clearer messages
