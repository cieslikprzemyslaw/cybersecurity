# Threat Modeling Mini-Project Debrief

## What I modelled

I created a simple web application threat model with the following architecture:

- User
- Browser
- React Frontend
- Backend API
- Database

I added two trust boundaries:

- client-side boundary
- server-side boundary

The main data flows were:

- user interaction with the browser
- browser loading and executing the React application
- React rendering and updating the DOM
- HTTPS API requests from the frontend to the backend
- API responses from the backend to the frontend
- database queries from the backend
- query results returned from the database

## Why I used STRIDE

I used STRIDE to organise potential threats into the following categories:

- Spoofing
- Tampering
- Repudiation
- Information Disclosure
- Denial of Service
- Elevation of Privilege

This helped me review the whole architecture instead of focusing only on injection or authentication issues.

## Threats identified

1. Modified API Request
2. Broken Object-Level Authorization
3. Session or Token Theft
4. Excessive Data Returned by API
5. Missing Audit Logging
6. Sensitive Information Exposed in Browser Console
7. Injection in Database Query
8. Sensitive Data Returned from Database

## Key security assumptions

- The user and browser are untrusted.
- Client-side requests can be modified before they reach the backend.
- The backend must independently identify the authenticated user and enforce authorisation.
- The database should be accessed through the backend.
- API responses should follow data minimisation and least privilege.
- HTTPS protects data in transit but does not replace application-layer security controls.

## What I learned

The most important lesson was that threat modeling does not confirm that a vulnerability exists. It identifies where a weakness could appear and what should be reviewed or tested later.

I also learned that:

- a valid session does not automatically mean that a user is authorised to access every object
- parameterised queries prevent injection but do not prevent excessive data exposure
- audit logging supports accountability, but logs must not contain secrets or unnecessary personal data
- API responses should follow data minimisation and least privilege
- browser console logging can expose sensitive information
- trust boundaries help identify where data moves between components with different levels of trust
- the quality of a threat model depends heavily on how accurately the architecture and data flows are described

## Validation ideas

The threats could be validated through:

- manual code review
- SAST
- DAST
- API testing
- session and cookie configuration review
- authorisation testing
- response schema review
- database query review
- audit log review

## Limitations

This was a hypothetical learning model and was not created from a real application implementation.

The identified threats were not tested against source code, infrastructure, configuration, or a running system. All threats therefore remain open and should be treated as potential risks rather than confirmed vulnerabilities.

## Final reflection

This exercise helped me understand the difference between:

- architecture
- trust boundaries
- threats
- vulnerabilities
- validation
- mitigations

It also showed me that threat modeling is most useful before development or during design review, because it helps identify security requirements before weaknesses reach production.
