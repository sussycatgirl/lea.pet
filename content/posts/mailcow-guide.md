---
title: "Mailcow Setup Guide"
date: 2022-12-20T16:17:37+01:00
draft: false
hidden: true
author: Lea
author_url: https://lea.pet/@lea
---

This post is primarily intended for users of my private mailserver, [mail.amogus.cloud](https://mail.amogus.cloud), but some of the information, especially the DNS part, might also be useful to others.

<!--more-->

# Using an external mail client

Instead of the webmail, you can also use other mail clients (Thunderbird, K-9 Mail, Gmail, etc) to access your account.

Many mail clients are able to automatically fetch the server configuration. If your client supports this, you will only need to provide your **primary email address** and **account password**.

Otherwise you need to provide the following configuration:

### Incoming server (IMAP)
> If prompted whether to use IMAP or POP3, choose IMAP.
```
Server: vps.srv.janderedev.xyz
Port: 993
Security: SSL/TLS
Username: (your account's main address)
Password: (your account password)
```

### Outgoing server (SMTP)
```
Server; vps.srv.janderedev.xyz
Port: 465
Security: SSL/TLS
Username: (your account's main address)
Password: (your account password)
```

## Nextcloud

If you use [my Nextcloud server](https://amogus.cloud), you might have noticed that it has email integration. Since your mail account and Nextcloud/Authentik account are completely separate, mail integration can't be configured automatically.

If you want to access your emails from Nextcloud, navigate to the [mail tab](https://amogus.cloud/apps/mail/) and add a new mail account with your username/password combination.

Nextcloud supports automatic configuration, so you won't need to input the configuration details above.

You can also add mail accounts from other providers here, but keep in mind that you are entrusting me with full access to all your emails. Nextcloud might also start categorizing and tagging your emails automatically, which can be rather annoying to clean up if not wanted.

(Also, Nextcloud's webmail isn't that good anyway)

# Setting up your own domain

> This segment is only relevant if you want to use and manage your own domain on my mailserver. Please ask me to add the domain and assign an administrator account to you first, then refer to this section for DNS setup.

## DNS Records

Please replace `example.com` with your own domain name. If you are using Cloudflare, disable the proxy for the CNAME records.

```
Type   Domain                       Record value

MX     example.com                  vps.srv.janderedev.xyz
CNAME  autodiscover.example.com     vps.srv.janderedev.xyz
CNAME  autoconfig.example.com       vps.srv.janderedev.xyz
TXT    example.com                  v=spf1 mx -all
TXT    _dmarc.example.com           v=DMARC1; p=reject; rua=mailto:dmarc@janderedev.xyz
TXT    dkim._domainkey.example.com  (read section below)
```

### DKIM

The DKIM record contains an unique public key that is different for every domain.

To retrieve the correct value of this record, log in with the provided administrator credentials, navigate to the "Domains" tab and click the blue "DNS" button next to your domain.

Ignore the other data and scroll down all the way to the bottom. The last entry should contain a long string of characters, similar to this:

> `v=DKIM1;k=rsa;t=s;s=email;p=MIIBIjAN [...]`

Copy this entire string, including the `v=DKIM1; [...]` part, and insert it as the value of the `dkim._domainkey.example.com` record.
