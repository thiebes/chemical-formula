# Contribution Checklist: chemical-formula Bug Fix

This is a quick reference checklist to accompany the detailed CONTRIBUTION_PLAN.md.

## Pre-Contribution

- [ ] Verify Node.js version (18 || >=20)
- [ ] Confirm bugs still exist in latest version (4.0.1)
- [ ] Read CONTRIBUTION_PLAN.md thoroughly

## Step 1: Issue Creation

- [ ] Draft issue with bug examples and expected behavior
- [ ] Include impact statement and affected compounds
- [ ] Mention willingness to submit PR
- [x] Submit issue on GitHub
- [x] Record issue number: `#287`
- [ ] Wait 3-5 days for maintainer response

## Step 2: Repository Setup

- [ ] Fork repository to your GitHub account
- [ ] Clone fork locally
- [ ] Add upstream remote
- [ ] Install dependencies: `npm install`
- [ ] Run existing tests: `npm test` (all should pass)
- [ ] Run linter: `npm run lint` (all should pass)
- [ ] Create feature branch: `fix/nested-subscripts-in-parentheses`

## Step 3: Code Analysis

- [x] Review `index.js` (main parser)
- [x] Review `test/index.js` (test patterns)
- [x] Review `package.json` (scripts and dependencies)
- [x] Identify bug location in code
- [x] Review our working implementation
- [x] Plan adaptation strategy

## Step 4: Implementation

- [x] Backup original `index.js` (just in case)
- [x] Implement recursive parsing fix in `index.js`
- [x] Maintain same function signature
- [x] Use existing dependencies (chemical-symbols, lodash)
- [x] Add code comments for complex logic
- [x] Verify element validation still uses `getAtomicNumber()`

## Step 5: Testing

- [x] Add test suite for nested subscripts in `test/index.js`
- [x] Include all bug case tests (Al2(SO4)3, Fe(H2O)6, etc.)
- [x] Add error handling tests
- [x] Add edge case tests
- [x] Run tests: `npm test` (all should pass - 44/44 passing)
- [x] Run coverage: `npm run coverage` (93.1% achieved)
- [x] Verify all original tests still pass (no regressions)
- [x] Manual testing of bug cases

## Step 6: Quality Checks

- [x] Run linter: `npm run lint` (0 errors)
- [x] Fix any linting issues
- [x] Verify code style matches project conventions
- [x] Review all changes with `git diff`
- [x] Ensure no unintended changes
- [x] Check for console.logs or debug code

## Step 7: Commit and Push

- [ ] Stage changes: `git add index.js test/index.js`
- [ ] Review staged changes: `git diff --staged`
- [ ] Create commit with descriptive message
- [ ] Include "Fixes #[ISSUE_NUMBER]" in commit message
- [ ] Push to fork: `git push origin fix/nested-subscripts-in-parentheses`

## Step 8: Pull Request

- [ ] Navigate to fork on GitHub
- [ ] Click "Compare & pull request"
- [ ] Verify base: kenany/chemical-formula master
- [ ] Verify compare: your-username/chemical-formula fix/nested-subscripts-in-parentheses
- [ ] Paste PR description (from CONTRIBUTION_PLAN.md)
- [ ] Include "Fixes #[ISSUE_NUMBER]" in description
- [ ] Review file changes in PR
- [ ] Create pull request
- [ ] Add labels if possible (bug, enhancement)

## Step 9: Post-PR

- [ ] Monitor PR for feedback (check daily)
- [ ] Respond to comments within 24-48 hours
- [ ] Make requested changes if needed
- [ ] Rebase if master has moved ahead
- [ ] Thank maintainer for review
- [ ] Wait for approval and merge

## Step 10: Follow-up

- [ ] Verify fix in published package version
- [ ] Update Chemly dependency: `npm install chemical-formula@latest`
- [ ] Test in Chemly application
- [ ] Consider removing custom parser if no longer needed
- [ ] Update this checklist with lessons learned

## Notes and Reminders

**Issue Number**: `#287`

**Fork URL**: `https://github.com/______/chemical-formula` (fill in your username)

**PR URL**: `______` (fill in after creating PR)

**Important Commands**:
```bash
npm install          # Install dependencies
npm test            # Run tests
npm run lint        # Run linter
npm run coverage    # Run coverage

git status          # Check status
git diff            # View changes
git add .           # Stage changes
git commit -m ""    # Commit
git push origin fix/nested-subscripts-in-parentheses  # Push
```

**Key Files**:
- `index.js` - Main parser (modify this)
- `test/index.js` - Tests (add tests here)
- `package.json` - Scripts and deps (read only)

**Test Cases to Include**:
- Al2(SO4)3 → {Al:2, S:3, O:12}
- Fe(H2O)6 → {Fe:1, H:12, O:6}
- Fe(H2O) → {Fe:1, H:2, O:1}
- Ca(OH)2 → {Ca:1, O:2, H:2}
- Mg(NO3)2 → {Mg:1, N:2, O:6}
- Ca3(PO4)2 → {Ca:3, P:2, O:8}
- Mg3(Fe(CN)6)2 → {Mg:3, Fe:2, C:12, N:12}

**Blockers to Watch For**:
- Maintainer not responsive (wait 2 weeks, then ping)
- Style differences (run linter, match their code)
- Test framework confusion (review existing tests)
- CI/CD failures (review logs, ask for help)

**Success Indicators**:
- All tests pass
- Linter passes
- Maintainer approves PR
- PR merged to master
- New version published with fix
