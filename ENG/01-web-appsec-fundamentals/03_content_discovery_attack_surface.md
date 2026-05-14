# Content Discovery and Attack Surface

## 1. Overview

This note covers content discovery and attack surface mapping.

A web application is often bigger than what is visible in the navigation menu. Some resources may not be linked in the UI, but they may still exist and respond to requests.

Content discovery helps find those resources.

Attack surface mapping helps understand where the application can be attacked or abused.

The main idea:

> If a resource exists and is reachable, it must be protected properly.

## 2. What is Content Discovery?

Content Discovery is the process of finding application resources that are not always visible in the UI.

Examples of discoverable content:

- hidden endpoints,
- directories,
- files,
- admin panels,
- backup files,
- old application versions,
- API routes,
- configuration files,
- upload folders,
- `robots.txt`,
- `sitemap.xml`,
- HTML comments,
- JavaScript files,
- links inside JavaScript bundles.

Example paths:

```text
/admin
/api
/backup
/uploads
/dev
/test
/config
```

Content discovery is useful because applications often expose more than developers expect.

## 3. Why the UI Does Not Show the Whole Application

The UI does not always show the full application.

An application may have endpoints that:

- are called only by JavaScript,
- are not linked in the menu,
- are intended for admins,
- are old but still deployed,
- were used for testing,
- were accidentally left in production,
- are used by mobile apps,
- are used by internal tools,
- are part of an API,
- are accessible through predictable names.

A missing link in the UI does not mean the resource is private.

For example, the menu may not show:

```text
/admin
```

but the URL may still exist.

## 4. Hidden URLs Are Not Security

A hidden URL is not a real security control.

Example:

```text
/admin
/backup
/dev
/api/users
/uploads
```

Even if there is no visible link to those paths, they may still be discovered through:

- `robots.txt`,
- `sitemap.xml`,
- DevTools,
- Burp history,
- JavaScript files,
- HTML comments,
- wordlists,
- search engines,
- archived pages,
- guessing common names.

This is related to the idea of security by obscurity.

Security by obscurity means relying on something being hidden rather than properly protected.

Hiding can reduce accidental discovery, but it is not enough. Sensitive resources still need proper server-side access control.

## 5. robots.txt

`robots.txt` tells search engine crawlers which paths they should not index.

Example:

```text
User-agent: *
Disallow: /admin
Disallow: /backup
```

From an AppSec perspective, this file can be interesting because it may reveal sensitive or hidden paths.

Important point:

> robots.txt does not protect anything. It only asks crawlers not to index certain paths.

If `/admin` is listed in `robots.txt`, I should not assume it is secure. I should check how the server responds when that path is requested.

Possible secure behaviours:

- redirect to login,
- `401 Unauthorized`,
- `403 Forbidden`,
- sometimes `404 Not Found`,
- proper access control after authentication.

## 6. sitemap.xml

`sitemap.xml` may contain a list of URLs in the application.

It can reveal:

- pages not visible in the menu,
- old pages,
- unusual paths,
- application sections,
- test pages,
- content types,
- endpoints or public routes.

Example:

```text
/sitemap.xml
```

Sitemap files are useful during application mapping because they provide a quick overview of known URLs.

However, just because a URL appears in a sitemap does not mean it is safe. Each URL still needs proper access control and secure behaviour.

## 7. Favicon Fingerprinting

A favicon can sometimes help identify a technology, framework, admin panel, CMS or self-hosted application.

This is called favicon fingerprinting.

For example, if an application uses a default favicon from a known tool, it may reveal what technology is running behind the page.

This can help with further investigation because known technologies may have:

- default paths,
- default admin panels,
- known configuration files,
- known version-specific vulnerabilities,
- common misconfigurations.

Favicon fingerprinting is not proof of a vulnerability by itself. It is only a clue.

## 8. Manual Application Mapping

Manual mapping means browsing the application and recording how it works.

During manual mapping, I should look for:

- pages,
- forms,
- buttons,
- links,
- API requests,
- parameters,
- cookies,
- headers,
- redirects,
- upload features,
- login and logout flows,
- password reset flows,
- admin-related functionality,
- unusual server responses.

Manual mapping is important because it gives business context.

For example, I should understand:

- What does the application do?
- What data does it process?
- Where does it accept user input?
- Which actions change data?
- Which features require authentication?
- Which features require special permissions?
- Which requests include user or resource IDs?

Automated tools can find paths, but manual mapping helps understand meaning and risk.

## 9. Automated Discovery

Automated discovery uses tools and wordlists to find common paths.

Examples of tools:

- Gobuster,
- ffuf,
- dirsearch,
- Burp Intruder,
- Burp Content Discovery features.

Example paths a wordlist may test:

```text
/admin
/api
/backup
/uploads
/dev
/test
/config
```

Automated tools are useful, but they are not magic.

They only find what they try.

Results depend on:

- the wordlist,
- the target configuration,
- status codes,
- redirects,
- rate limits,
- authentication,
- file extensions,
- naming conventions.

A good approach combines automated discovery with manual thinking.

## 10. What is Attack Surface?

Attack surface means all the places where an application can be attacked, abused or misused.

Examples of attack surface:

- login,
- registration,
- password reset,
- file upload,
- forms,
- API endpoints,
- URL parameters,
- cookies,
- headers,
- admin panels,
- public directories,
- third-party integrations,
- old endpoints,
- subdomains,
- webhooks,
- search features,
- payment or checkout flows.

The larger the attack surface, the more places need:

- validation,
- authentication,
- authorization,
- secure configuration,
- logging,
- error handling,
- rate limiting.

Attack surface mapping helps decide where to focus testing.

## 11. Interesting Paths to Investigate

Some paths are especially interesting during AppSec testing.

### `/api`

APIs may expose data or actions used by the frontend or mobile app.

Potential risks:

- IDOR,
- BOLA,
- Broken Access Control,
- excessive data exposure,
- missing authentication,
- missing authorization,
- weak input validation.

### `/admin`

Admin panels are sensitive because they often provide privileged actions.

Correct server behaviour may include:

- redirect to login,
- `401 Unauthorized` if not logged in,
- `403 Forbidden` if logged in but not admin,
- proper authorization checks,
- sometimes `404 Not Found` to avoid revealing the resource.

### `/backup`

Backup paths can be dangerous if exposed publicly.

Possible files:

```text
backup.zip
db.sql
config.bak
site.old
database.dump
```

Potential risks:

- source code exposure,
- database dump exposure,
- credentials leakage,
- configuration disclosure,
- old vulnerable code.

### `/uploads`

Upload directories are interesting because file upload features can introduce serious vulnerabilities.

Potential risks:

- public access to uploaded files,
- dangerous file types,
- weak file validation,
- path traversal,
- stored XSS,
- malware upload,
- overwriting files.

### `/dev` and `/test`

Development or test paths may expose unfinished or less secure functionality.

Potential risks:

- debug information,
- test accounts,
- verbose errors,
- internal tools,
- weaker access controls,
- temporary endpoints left in production.

## 12. Key Takeaways

- An application is more than visible links.
- A hidden URL is not a real security control.
- `robots.txt` and `sitemap.xml` can reveal interesting paths.
- Favicon fingerprinting can provide technology clues.
- Manual mapping helps understand business context.
- Automated discovery helps find common paths.
- Tools are useful, but they depend on wordlists and configuration.
- Attack surface includes every place where input, access or actions exist.
- Admin panels, APIs, backups and uploads are especially interesting.
- Sensitive resources must be protected with proper authentication and authorization.

## 13. Practical Discovery Checklist

When doing content discovery, I should check:

- Is there a `robots.txt` file?
- Is there a `sitemap.xml` file?
- Are there interesting paths in JavaScript files?
- Are there comments in HTML?
- Are there API calls visible in DevTools or Burp?
- Are there hidden forms or parameters?
- Are there admin-related paths?
- Are there backup files?
- Are there upload directories?
- Are old or test paths still available?
- What status codes do interesting paths return?
- Does the server require authentication?
- Does the server check authorization?
- Does the response expose sensitive data?

Practical mindset:

> Finding a hidden path is not the vulnerability by itself. The vulnerability appears when the path exposes data, allows unauthorized actions or is misconfigured.
