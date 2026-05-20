# Java Upload Risks

Modern Java/Spring applications are usually routed through controllers. Uploaded files do not normally execute just because they exist on disk.

Risky cases can involve `.jsp` upload into a web-accessible executable location, depending on Tomcat/application server configuration.

## Main Takeaway

In Java, RCE through upload is configuration-dependent, but path traversal, stored XSS and unsafe file serving remain important.
