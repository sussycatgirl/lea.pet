---
title: "Using IMAP and SMTP with Office 365 mail accounts"
date: 2023-06-07T09:09:02+02:00
draft: false
author: Lea
author_url: https://lea.pet/@lea
---

Leaving this here mainly as a reminder to myself.<!--more--> These options work in both Thunderbird and K-9 Mail, but any client that supports OAuth2 should work.

### IMAP
```
Server/Port: outlook.office365.com:993
Username: (Probably your email address)
Authentication method: OAuth2
```

### SMTP
```
Server/Port: smtp.office365.com:587
Username: (Same as for IMAP)
Security: STARTTLS
Authentication method: OAuth2
```
