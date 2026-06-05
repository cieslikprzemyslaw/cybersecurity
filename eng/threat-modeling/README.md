# Threat Modeling

Threat modeling notes and mini-projects for reviewing application designs before implementation or testing.

Use this section to practise describing architecture, trust boundaries, data flows, assumptions, threats, validation ideas, and mitigations.

## Mini-Projects

| Project | Scope | Main artifacts |
|---|---|---|
| [Simple Web Application Threat Model](simple-web-application-threat-model/README.md) | STRIDE model for a basic React frontend, backend API, and database architecture. | [Threat register](simple-web-application-threat-model/threats.md), [STRIDE cheat sheet](simple-web-application-threat-model/stride-cheat-sheet.md), [Debrief](simple-web-application-threat-model/debrief.md) |

## File Role Guide

- `README.md` - project index, scope, and artifact links.
- `threats.md` - structured threat register with validation ideas and mitigations.
- `stride-cheat-sheet.md` - quick STRIDE reference.
- `debrief.md` - learning reflection, assumptions, and limitations.
- `source/` - editable threat modeling source files.
- `reports/` - exported reports.
- `assets/` - diagrams and supporting images.
