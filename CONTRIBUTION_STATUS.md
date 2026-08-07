# Contribution Status

## Current Status: ISSUE SUBMITTED - AWAITING MAINTAINER RESPONSE

**Decision**: Keep issue #287 focused on parentheses bug only. Hydrate notation (e.g., CuSO4·5H2O) is a separate feature request that can be addressed later if needed.

### Issue Details
- **Issue Number**: #287
- **Issue URL**: https://github.com/kenany/chemical-formula/issues/287
- **Submitted Date**: 2025-10-15
- **Status**: Open, awaiting maintainer response

### Next Steps

According to the contribution plan, we should:

1. **Wait 3-5 business days** for maintainer response
2. Monitor the issue for:
   - Maintainer acknowledgment of the bug
   - Confirmation they're open to a PR
   - Any specific requirements or preferences

### While Waiting

We can prepare but should NOT submit a PR until the maintainer responds:
- ✓ Issue submitted with clear reproduction steps
- ✓ Bug verified in version 4.0.1
- ✓ Code fix ready in CONTRIBUTION_CODE.md
- ⏳ Waiting for maintainer green light

### When to Proceed

Proceed to Step 2 (fork and implement) when maintainer:
- Confirms the bug exists
- Indicates they're open to reviewing a PR
- Provides any specific guidance or requirements

### If No Response

- After 1 week: Politely ping on the issue
- After 2 weeks: Check maintainer's recent GitHub activity
- After 3 weeks: Consider alternative approaches (discussed in CONTRIBUTION_PLAN.md)

### Checklist Progress

Completed:
- [x] Verify Node.js version (22.17.0)
- [x] Confirm bugs exist in version 4.0.1
- [x] Read CONTRIBUTION_PLAN.md
- [x] Draft issue with examples
- [x] Submit issue (#287)
- [x] Record issue number

Waiting:
- [ ] Wait 3-5 days for response
- [ ] Maintainer confirms bug
- [ ] Maintainer open to PR

### Bug Verification Results

Tested on 2025-10-15 with Node.js v22.17.0:

```
Al2(SO4)3: {"Al":2,"O":3,"S":3}
Expected:  {"Al":2,"S":3,"O":12}
❌ WRONG - O should be 12, not 3

Fe(H2O)6: {"Fe":1,"H":6,"O":6}
Expected: {"Fe":1,"H":12,"O":6}
❌ WRONG - H should be 12, not 6

Fe(H2O): {"Fe":1,"H":null}
Expected: {"Fe":1,"H":2,"O":1}
❌ WRONG - H is null, O is missing
```

All three critical bugs confirmed.

### Resources

- Issue template: GITHUB_ISSUE.md
- Implementation code: CONTRIBUTION_CODE.md
- Detailed plan: CONTRIBUTION_PLAN.md
- Quick checklist: CONTRIBUTION_CHECKLIST.md
- This status: CONTRIBUTION_STATUS.md
