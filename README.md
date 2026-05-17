# Frontend AppSec Notes

This repository contains my personal web security and application security learning notes from the perspective of a Frontend Engineer.

The purpose of this repository is to build stronger security awareness around web applications, APIs, authentication flows, authorization checks, secure coding, and developer-friendly remediation.

The focus is defensive and developer-oriented: understanding risks, improving secure implementation, and writing clearer remediation notes.

This is not a collection of full walkthroughs, exploit guides, or lab answers. It is a structured learning space for notes, cheatsheets, summaries, and practical reflections that help connect AppSec concepts with real development work.

## Why This Repository Exists

As a Frontend Engineer, I regularly work with user interfaces, APIs, CMS-driven platforms, client-side behaviour, production issues, and frontend/backend data flow.

Learning AppSec helps me look at those same systems through a security lens:

- What can the user control?
- Where does that input go?
- What does the backend trust?
- What happens if a request is modified?
- Is this an authentication issue or an authorization issue?
- What would a safe implementation look like?
- How could this be tested to prevent regression?

The goal is to better understand how web security issues happen and how developers can prevent them during normal engineering work.

## Learning Approach

My learning approach is intentionally practical:

- **10% theory** to understand the risk
- **70% hands-on labs** to practise safely
- **20% notes and review** to turn practice into understanding

For each topic, I aim to go beyond “what payload works” and focus on:

- root cause
- impact
- trust boundaries
- developer mistakes
- secure implementation
- remediation
- regression testing

## Roadmap

The roadmap is focused on web application security fundamentals first, then secure coding and developer workflow.

### 1. Web Foundations

Core web concepts that matter for AppSec:

- HTTP requests and responses
- cookies and sessions
- browser/server communication
- frontend/backend data flow
- content discovery
- user-controlled input

### 2. Burp Suite and Manual Testing

Practical request/response testing:

- Proxy
- Repeater
- basic Intruder usage for labs
- modifying parameters, cookies, headers, methods, and request bodies
- comparing responses
- understanding how applications behave when requests are changed

### 3. Authentication and Authorization

Understanding identity and access control:

- authentication vs authorization
- login logic
- username enumeration
- authentication bypass patterns
- session handling
- broken access control
- IDOR
- horizontal and vertical privilege issues

### 4. Client-Side and Browser-Based Issues

Frontend-adjacent security topics:

- reflected XSS
- stored XSS
- DOM XSS
- sources and sinks
- output encoding
- safe rendering
- dangerous DOM APIs
- browser trust boundaries

### 5. Injection and Server-Side Input Handling

Input that changes backend behaviour:

- SQL Injection
- Boolean-based blind SQLi
- time-based blind SQLi
- UNION-based SQLi
- path traversal
- file upload issues
- command injection basics
- SSRF basics

### 6. API and Configuration Security

Web application and API security topics:

- API authorization
- object-level access control
- CORS misconfiguration
- security headers
- error handling
- sensitive data exposure
- rate limiting basics

### 7. Secure Coding and Review

Developer-focused AppSec practices:

- secure code review
- SAST basics
- DAST basics
- dependency scanning
- secrets scanning
- threat modelling basics
- remediation planning
- writing clear findings

## Standard Workflow for Each Topic

For each topic, I try to follow the same workflow:

1. Read enough theory to understand the risk.
2. Practise in a legal lab or local test environment.
3. Test manually first before relying on automation.
4. Write notes in my own words.
5. Identify the root cause and impact.
6. Write a developer-friendly fix.
7. Add a checklist or takeaway for future reference.

## Note Template

Most notes will follow a structure similar to this:

```md
# Topic Name

## TL;DR

Short explanation of the issue.

## What I Practised

Brief summary of the lab or exercise without publishing sensitive answers.

## Key Concept

What the vulnerability is and why it happens.

## What the User Controls

Parameter, path, body, cookie, header, file, URL, or DOM source.

## Where the Input Goes

SQL query, HTML response, DOM sink, filesystem path, backend request, authorization check, etc.

## Impact

Realistic impact without exaggeration.

## Developer Remediation

How to fix or prevent the issue.

## Review Checklist

Questions to ask during code review or testing.

## Main Takeaway

One practical lesson from the topic.
```

## Repository Structure

This repository is organised by topic, not by file type.

Each topic may contain notes, cheatsheets, lab summaries, remediation notes, and review checklists in one place. This makes it easier to study a topic end-to-end instead of jumping between separate `notes` and `cheatsheets` folders.

The repository contains both English and Polish notes:

- [English notes index](eng/README.md)
- [Polish notes index](pl/README.md)

Current structure:

- `eng/fundamentals/` - reusable foundations before vulnerability-specific study.
- `eng/key-web-vulnerabilities/` - the Key Web Vulnerabilities series.
- `pl/` - Polish version following the same structure.
- `labs/` folders - short summaries of legal training labs.
- `overview.md` files - concise source-of-truth notes for each topic.
- `cheat-sheet.md` files - practical review and testing workflows.

The Polish version follows the same high-level structure as the English version.

## Ethics and Scope

All notes and examples in this repository are for learning and professional development.

Practice should be limited to:

- legal labs
- intentionally vulnerable applications
- local test environments
- authorised learning platforms
- explicitly permitted testing scopes

Do not test real systems without permission, clear scope, and safe harbour.

### This repository does not include real target data, credentials, private information, or instructions for testing systems without permission.

