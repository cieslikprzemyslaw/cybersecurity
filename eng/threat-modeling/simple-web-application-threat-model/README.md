# Simple Web Application Threat Model

This mini-project models a basic web application using OWASP Threat Dragon and the STRIDE methodology.

The architecture contains:

- User
- Browser
- React Frontend
- Backend API
- Database

The model includes a client-side trust boundary and a server-side trust boundary. The main data flows cover user interaction, frontend execution, HTTPS API requests and responses, database queries, and query results.

![Web application threat model](assets/web-application-architecture.png)

## Project purpose

The goal of this exercise was to practise identifying potential security threats before testing a real implementation. The model does not confirm that any vulnerability exists. Each threat remains open because the architecture is hypothetical and no source code, configuration, infrastructure, or running application was reviewed.

## Summary

- Total threats: 8
- Open threats: 8
- High severity: 5
- Medium severity: 3
- Mitigated threats: 0

## Identified threats

1. Modified API Request
2. Broken Object-Level Authorization
3. Session or Token Theft
4. Excessive Data Returned by API
5. Missing Audit Logging
6. Sensitive Information Exposed in Browser Console
7. Injection in Database Query
8. Sensitive Data Returned from Database

See [threats.md](threats.md) for the full threat descriptions, validation ideas, and mitigations.

## Files

- `assets/web-application-architecture.png` - exported diagram
- `reports/simple-web-application-threat-model.pdf` - Threat Dragon PDF report
- `reports/simple-web-application-threat-model.html` - Threat Dragon HTML report
- `source/threat-dragon-model.json` - editable Threat Dragon source model
- `stride-cheat-sheet.md` - short STRIDE reference
- `debrief.md` - learning reflection and limitations
- `threats.md` - structured threat register

## Status

This is a hypothetical learning model. The threats have not been validated against a real implementation.
