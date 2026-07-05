# OpenOutreach

Free, self-hosted mail merge. Build a personalized email in your browser, download a Python script, run it from your own machine through your own Gmail account. No SaaS subscription, no third party holding your contact list, no vendor lock-in.

![OpenOutreach](openoutreach-brand/logo_horizontal.png)

---

## What this is (and isn't)

OpenOutreach is a lightweight mail-merge tool for **warm, expected, low-to-medium volume email** — applicant communication, event updates, community newsletters, personal outreach to people who already know you. It is not a cold-email growth platform: it has no mailbox warmup, IP rotation, or deliverability infrastructure, and Gmail itself caps sending at roughly 500/day on a personal account (~2,000/day on Workspace). If you need to blast thousands of cold prospects a day, you want a dedicated cold-email platform instead.

What you get:
- A single HTML file — the **builder** — where you write your email and preview it live
- A generated **Python script** that sends it, personalized per recipient from a CSV
- Support for **any column in your CSV** as a placeholder, not just name/email
- A built-in **compliance footer** and **unsubscribe suppression list**
- A companion script that **automatically detects unsubscribe replies**

---

## Quick start

1. **Open the builder** — double-click `openoutreach.html`. It runs entirely in your browser; no install, no server.
2. **Write your email** — fill in the subject, main message, and sign-off. Use `{{name}}`, `{{email}}`, or any custom column from your CSV as a placeholder.
3. **Check the live preview** on the right.
4. **Download the mailer script** — click "Download mailer script." This saves a `.py` file with your content baked in.
5. **Get a Gmail App Password** (see below) and set two environment variables.
6. **Put your CSV and logo next to the downloaded script**, then run it.

```bash
export GMAIL_ADDRESS="you@example.com"
export GMAIL_APP_PASSWORD="xxxxxxxxxxxxxxxx"
python openoutreach_send.py
```

That's it — it'll send one personalized email per row in your CSV.

---

## Step-by-step setup

### 1. Get a Gmail App Password

App Passwords let a script sign in to Gmail without using your real password.

1. Go to [myaccount.google.com/security](https://myaccount.google.com/security) and turn on **2-Step Verification** if it isn't already on.
2. Go to [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).
3. Type a name (e.g. `openoutreach`) and click **Create**.
4. Copy the 16-character password shown — you won't see it again.

**Using a Google Workspace (company) account?** App Passwords are sometimes disabled by admin policy. If you're the admin: go to [admin.google.com](https://admin.google.com) → **Security → Authentication → 2-step verification** → enable "Allow users to use App Passwords." If you're not the admin, ask whoever is, or switch to OAuth2/Gmail API (not covered by this script).

### 2. Set your credentials as environment variables

Never put your password directly in the script. Instead, set it as an environment variable each time you run it:

**macOS / Linux (bash/zsh):**
```bash
export GMAIL_ADDRESS="you@example.com"
export GMAIL_APP_PASSWORD="abcdefghijklmnop"
```

**Windows (PowerShell):**
```powershell
$env:GMAIL_ADDRESS="you@example.com"
$env:GMAIL_APP_PASSWORD="abcdefghijklmnop"
```

These only last for your current terminal session — you'll need to set them again if you close the terminal (or add them to your shell profile if you send emails often).

### 3. Prepare your files

Everything needs to sit in the **same folder**:

```
your-folder/
├── openoutreach_send.py     ← downloaded from the builder
├── applicants.csv           ← your real recipient list
├── logo.png                 ← your logo, shown at the top of the email
└── unsubscribed.csv         ← created automatically on first run
```

### 4. Run it

```bash
python openoutreach_send.py
```

You'll see progress printed for each recipient:
```
Found 42 applicant(s). Sending emails...
  sent to jane@example.com
  sent to john@example.com
  skipped (unsubscribed): blocked@example.com
...
Done. Sent: 40/42  Skipped (unsubscribed): 1
```

---

## The builder, field by field

### Content

| Field | What it does |
|---|---|
| **Subject line** | The email subject. Supports placeholders. |
| **Main message** | The full email body. Write everything here — greeting, details, links. Line breaks are preserved exactly. |
| **Sign-off** | A separate closing block (e.g. "Best regards, Hiring Team"), kept visually distinct from the main message. |

### Using your CSV columns

Any field above can reference a column from your CSV using `{{column_name}}`:

```
Hi {{name}},

Congratulations! You've been shortlisted for {{role}} at {{college}}.
```

- `{{name}}` and `{{email}}` are **always available**, even if your CSV doesn't explicitly need them elsewhere.
- Any other column — `{{role}}`, `{{college}}`, `{{recruiter}}`, whatever you have — works automatically as long as your CSV has a column with that exact name.
- If a placeholder has no matching column, it's left as literal text (`{{typo}}`) instead of silently breaking, so mistakes are easy to spot.

**Sample values (for preview only):** since the builder doesn't read your real CSV, type `key=value` pairs here (one per line) to see how placeholders render with realistic data. This has no effect on the actual script — that always reads your real CSV.

### Delivery settings

| Field | What it does |
|---|---|
| **Applicant list file** | Filename of your CSV. Must have at minimum a `name` and `email` column. |
| **Logo file** | Filename of the image shown at the top of the email. |
| **Delay between emails** | Seconds to wait between each send — keeps you under Gmail's rate limits. Default 2s is a reasonable starting point. |
| **Script file name** | What the downloaded `.py` file is called. |

### Compliance (unsubscribe)

| Field | What it does |
|---|---|
| **Include compliance footer** | Toggles a footer on every email with your org identity and an opt-out instruction. On by default — recommended for any outbound email. |
| **Organization name / Mailing address** | Shown in the footer. A physical address is a CAN-SPAM requirement in the US if this is used for anything resembling marketing. |
| **Suppression list file** | Filename of a plain-text list (one email per line) of people who've opted out. The script checks this before every send and skips matches automatically. |

---

## The unsubscribe workflow

1. A recipient replies to your email with the word **"Unsubscribe"** in the subject or body (the footer tells them to do exactly this).
2. Before your next batch, run the companion checker script:
   ```bash
   python check_unsubscribes.py
   ```
   This logs into your inbox via IMAP, finds any message with "unsubscribe" in the subject, and appends the sender's address to your suppression file.
3. Run your mailer again — anyone in the suppression list is automatically skipped and logged as `skipped (unsubscribed)`.

This is a lightweight, reply-based opt-out — it doesn't require hosting a web server or building a click-tracking link. It's appropriate for the low/medium-volume, relationship-based use cases this tool targets. If you're sending anything closer to bulk marketing, get a legal opinion on whether this satisfies your jurisdiction's requirements (CAN-SPAM, GDPR, etc.) before relying on it — this isn't legal advice.

---

## FAQ / troubleshooting

**"The setting that you are looking for is not available for your account" when creating an App Password**
Your account is a Google Workspace (company/school) account with App Passwords disabled by policy. See the Workspace note above.

**Emails aren't sending / authentication error**
Double check `GMAIL_ADDRESS` and `GMAIL_APP_PASSWORD` are set correctly in your *current* terminal session — they don't persist across terminal restarts unless added to your shell profile.

**Emails landing in spam**
This tool sends through your real Gmail account with no special deliverability infrastructure. Keep volume modest (well under Gmail's daily caps), avoid spam-trigger language, and make sure your account has a sending history — a brand-new account blasting hundreds of emails on day one is much more likely to get flagged.

**Can I use Outlook or another provider instead of Gmail?**
Not out of the box — the generated script is hardcoded to `smtp.gmail.com`. You can edit `SMTP_HOST` and `SMTP_PORT` in the downloaded script and use provider-specific credentials, but this hasn't been tested against other providers.

**Can I attach a file?**
Not currently — this version sends HTML email with an inline logo but no attachments, by design (keeps deliverability simpler and avoids the most common spam triggers).

---

## License

MIT — use it, modify it, ship it in your own product. Attribution appreciated but not required.
