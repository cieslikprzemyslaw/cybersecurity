# A07 Practice Summary

## Completed Coverage

- IAAA distinctions: identification, authentication, authorization, accountability.
- Password reset as an alternative authentication mechanism.
- Reset-token binding to the correct account.
- Evidence required to confirm account takeover.
- MFA-pending versus fully authenticated session state.
- Server-side enforcement of MFA on protected resources.
- Frontend controls versus backend security boundaries.
- Session rotation and logout invalidation at awareness/review level.
- Enumeration and rate limiting at awareness/review level.

## Strongest Practical Lessons

1. Identity values supplied by the browser are claims, not trusted proof.
2. Recovery evidence must authorise one specific identity and action.
3. A displayed security step does not prove that the backend enforces it.
4. A redirect or unusual response is evidence of behaviour, not always final proof of impact.
5. Account takeover should be claimed only after controlled evidence confirms access.

## Final Assessment

**PASS** for the practical foundation expected from a Frontend Developer transitioning into AppSec.
