# Apache / Nginx Upload Configuration Notes

Uploads are safer when stored outside the executable web root.

Risky pattern:

```text
/var/www/html/uploads/shell.php
```

## Main Takeaway

Web server configuration can turn a weak upload feature into RCE.
