# Cloud Storage Upload Notes

Cloud storage changes the risk model. RCE may be less likely, but public exposure and stored XSS may still matter.

Check:

- bucket/object permissions
- public access
- predictable URLs
- user-controlled Content-Type metadata
- same-origin risks

## Main Takeaway

Cloud upload security is often about access control, metadata, object naming and browser rendering behaviour.
