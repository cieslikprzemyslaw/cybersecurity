# A10 Review Checklist

Use only the sections relevant to the reviewed feature.

## 1. Normal flow

- What should normally happen?
- Which backend operation is authoritative?
- Which state should change?
- Which state must not change?

## 2. Security invariant

- What rule must always remain true?
- Must access be denied unless authorization is confirmed?
- Must one sensitive operation happen only once?
- Must a database record and file exist together?
- Must an audit event exist for a privileged change?

## 3. Validation and expected failures

- Are missing fields rejected?
- Are empty values rejected where inappropriate?
- Are `null` values rejected?
- Are arrays or objects rejected where strings are expected?
- Are unknown enum values rejected?
- Are malformed IDs rejected?
- Are oversized bodies rejected?

## 4. Exceptional conditions

- What happens if the database fails?
- What happens if file storage fails?
- What happens if authorization cannot be verified?
- What happens if the session store fails?
- What happens if a dependency times out?
- What happens if the client disconnects?
- What happens if two requests arrive together?

## 5. Fail-open / fail-closed

- Does the application allow a sensitive operation when a security check fails?
- Does it deny access when authorization cannot be confirmed?
- Does fallback reduce security?
- Is uncertainty treated as permission?

## 6. Partial state

- Which steps completed before the error?
- Which changes were committed?
- Were files, messages, emails or external API calls already created?
- Is cleanup or rollback required?
- Does the user see a state that disagrees with the backend?

## 7. Retry and timeout

- Is timeout treated as unknown outcome?
- Could retry duplicate records, emails, payments, uploads or activity logs?
- Does the frontend blindly retry sensitive actions?

## 8. Error responses

- Is the status code appropriate?
- Is the client message generic but understandable?
- Is request/correlation ID included where useful?
- Are stack traces, SQL queries, paths, secrets and config hidden?

## 9. Internal evidence

- Is the error logged with safe context?
- Does the log include correlation ID?
- Are repeated failures monitored?
- Would the team know that cleanup or rollback failed?

## 10. Frontend behaviour

- Does the UI avoid showing success without confirmation?
- Are optimistic updates reverted or refreshed after failure?
- Does the UI handle unknown outcome separately from confirmed failure?
- Does disabled button support UX rather than security?
