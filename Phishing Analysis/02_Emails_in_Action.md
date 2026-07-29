## Phishing Analysis – Real-World Examples & Techniques

Went through six different phishing samples in this room, each demonstrating a different combination of tricks. Grouping the recurring techniques together with examples from each case, and explaining anything that isn't obvious at first glance.

### 1. Fake PayPal Transaction Email
**Techniques:** spoofed sender address, URL shortening, branded HTML

- The subject line used a fake transaction to create urgency — the classic "something happened to your money, act now" hook.
- The **From** address was the giveaway: it displayed as `service@paypal.com`, but the actual underlying address was `gibberish@sultanbogor.com`. Attackers can set any display name they want — the real sender only shows up in the raw headers.
- The single interactive element was a "Cancel the order" button, which pointed to a **shortened URL**. Shortened links (like bit.ly-style redirects) hide the real destination until you actually click — which is risky to do. Tools like [WhereGoes](https://wheregoes.com/) let you resolve a shortened link's final destination safely, without visiting it yourself.

### 2. Fake Shipping/Tracking Notification
**Techniques:** spoofed sender address, pixel tracking, link manipulation

- Subject line used a fake tracking number to bait a click.
- Display name said "Distribution Center" but the real address was `contact@beginpro.club` — same spoofing pattern as above.
- This email also had a **tracking pixel** — a tiny (often 1x1 pixel), invisible image embedded in the email. When your email client loads it, the image request pings the sender's server, quietly confirming that you opened the email and when. This is exactly why Yahoo auto-blocked the images and disabled hovering over links — many providers do this specifically to stop spammers from confirming an email was read/is a live target.
- The link's true destination stayed hidden until digging into the raw HTML source of the email.

### 3. Multi-Stage Redirect → Fake Login Portal (OneDrive/Adobe/Microsoft impersonation)
**Techniques:** artificial urgency, brand impersonation (layered), link redirection chain, credential harvesting

- Used a fake "fax document" with an expiration date on the same day it was sent — pure urgency pressure.
- Clicking "Download Document Here" didn't lead straight to malware — instead it walked through a **chain of redirects**: a fake OneDrive page → a fake Adobe page → and finally a fake login portal asking for email credentials (e.g. Outlook).
- This chaining is deliberate: each hop uses a different trusted brand's look-and-feel to keep building false confidence, and it also helps dodge basic email filters that only check the first link.
- **Credential harvesting** = the actual point of the whole chain — the fake login page isn't authenticating you to anything real; whatever you type (username/password) gets sent straight to the attacker, and you'll get a generic error regardless of whether you typed something real.
- Worth noting: this sample had obvious typos/bad formatting, but the notes flagged that AI tools now let attackers produce polished, typo-free phishing pages — so grammar mistakes are becoming a less reliable tell over time.

### 4. Fake Netflix Billing Issue
**Techniques:** spoofed sender, urgency, brand impersonation, poor grammar, malicious attachment

- Display name "Netlfix billing" (deliberate misspelling) doesn't match the actual sending domain.
- Claimed the account was suspended — urgency again, this time tied to fear of losing access to a paid service.
- Instead of a link in the email body, this one used a **PDF attachment** containing the malicious link ("Update Payment Account") — attackers do this because raw links in email bodies are easier for spam filters to scan and flag; burying the link inside an attachment is a way to dodge that.
- Two extra red flags: an oddly-formatted phone number, and use of a legitimate-looking Netflix help center domain to build false trust even while the actual payment link was fraudulent.

### 5. Fake Apple Purchase Notification
**Techniques:** spoofed sender, BCC recipient, urgency, poor grammar, unusual attachment format

- Display name "Apple Support" again mismatched the real sender, with typos even in the From/To fields.
- The victim was **BCCed** rather than the direct recipient — Blind Carbon Copy hides the full list of who else received the email, so the victim can't see it was mass-sent to many other people at once, which would otherwise be a giveaway that it's not a personal notification.
- The email body was completely blank — all the "work" of the attack was in a **.dot file attachment** (a Microsoft Word Template file — an unusual choice for something claiming to be a receipt, which is itself suspicious). Clicking an embedded image inside the document redirected to a phishing site with a long, overly complex URL — deliberately padded with familiar-looking words like "apps" and "ios" to look more legitimate at a glance.

### 6. Fake DHL Shipping Notification with Malicious Excel Attachment
**Techniques:** spoofed sender, brand impersonation, malicious Excel attachment

- Display name "DHL Express" didn't match the real sending domain.
- Email body was thin — most of the attack lived in an attached `.xlsx` file.
- The document had inconsistent geography as a red flag: sender domain was German, the invoice was addressed to a city in India, but the actual document content was written in Mandarin. Real, legitimate documents don't usually have this kind of geographic mismatch.
- A link inside the spreadsheet attempted to download and run `regasms.exe` — a malicious executable. In this case it errored out in testing, but the intent was clear: this is a classic pattern for delivering a payload that could go on to:
  - **Establish persistence** — set up a backdoor or scheduled task so the attacker keeps access after a reboot
  - **Exfiltrate data** — steal files, credentials, or saved browser passwords
  - **Deploy ransomware** — encrypt the system and demand payment to unlock it

### Recurring Patterns Across All Six Samples
- **Sender spoofing** — display name looks trustworthy, but the actual email address never matches
- **Manufactured urgency** — suspended accounts, expiring links, unauthorized purchases — all designed to make you act before thinking
- **Brand impersonation** — real company logos/HTML templates borrowed to build instant trust
- **Hiding the real destination** — via shortened links, redirect chains, or burying links inside attachments (PDF, Word, Excel) instead of the email body
- **The payload isn't always malware** — several of these existed purely to steal credentials through fake login pages, not to install anything
