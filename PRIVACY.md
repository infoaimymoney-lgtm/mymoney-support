# Privacy Policy — My Money

_Effective 2026-05-25_

My Money is a personal finance app that reads the **transactional** SMS already on your phone and turns it into a private spending dashboard. This document explains, in plain language, exactly what data the app touches, where it lives, and what (if anything) leaves your device.

The short version:

> **Your data stays on your phone.** The only time anything leaves the device is when *you* trigger it — sharing a backup file, or asking the AI assistant a question (and even then we send only aggregated, anonymised figures).

---

## 1. What data the app reads

### 1.1 SMS messages

The app needs `READ_SMS` and `RECEIVE_SMS` permissions to do its core job: turn bank, UPI, credit-card, and mutual-fund SMS into a clean transaction list.

It reads **only messages that look transactional**. Each SMS is parsed locally on your phone with a deterministic rules engine — there is no cloud parser. A message is kept only if it contains:

- a debit, credit, or transfer **amount** (₹), AND
- an action verb a bank/UPI/AMC would use (`debited`, `credited`, `spent`, `received`, `purchased`, `NAV`, `EMI`, `IMPS`, `UPI`, `NEFT`, etc.).

The following kinds of SMS are **dropped at parse time** and never written to the database, sent to the LLM, or included in backups:

- OTPs and one-time verification codes
- Balance-update pings without an action
- Promotional / marketing offers
- Personal messages from individuals
- Anything without a recognisable rupee amount

There is no upload of your inbox, ever. Your full SMS history stays in Android's SMS provider exactly where it was.

### 1.2 What we *don't* read

- Contacts, call logs, calendar, photos, location, microphone, camera — none of these are requested.
- Notifications from other apps — never read.
- Browsing history — never read.

---

## 2. Where your data lives

### 2.1 Local SQLite database

Every transaction the app derives from SMS is written to a single SQLite file (`my_money.db`) inside the app's private storage directory on your phone. Other apps on the device cannot read it (Android sandboxing). The database holds:

- Parsed transaction rows (date, amount, direction, parsed category, parsed merchant name, your manual labels)
- A link back to the source SMS id (so reloads stay consistent)
- Your goals, reminders, custom categories, and per-message overrides

Nothing in this database is uploaded to a remote server. There is no "My Money cloud."

### 2.2 Backups

You can choose to save a backup file (`.mmbak`) from **Settings → Backup to a file**. The backup is created in your phone's temp directory, then handed to the Android share sheet so *you* pick where it goes — Google Drive, Gmail, a USB cable transfer to your PC, anywhere.

- **Encrypted backups (recommended)**: You set a passphrase. The file is encrypted with **AES-256-GCM**, with the key derived from your passphrase via **PBKDF2-HMAC-SHA256 (200 000 iterations + per-file salt)**. We do not store, log, or transmit the passphrase. If you forget it, the file cannot be recovered — by us, by anyone.
- **Plain backups (optional)**: The file is a copy of the SQLite DB. Encrypted format is strongly recommended; the plain option exists for convenience when restoring locally.

Where the backup ultimately lives (your Drive, your email, your PC) is governed by *that* service's privacy policy, not ours.

---

## 3. AI Assistant (optional)

The AI Assistant is **off by default**. To enable it you paste your own API key from a provider you choose (Anthropic, OpenAI, etc.) in **Settings → AI Assistant**. We do not run an AI service of our own; your key calls your provider directly.

### 3.1 What is sent to the LLM

When you ask the assistant a question, a prompt is built locally on your phone and sent to your chosen provider. The prompt contains **only aggregated, anonymised figures**:

- Total spend, income, net, savings rate for the active window
- Spend totals **per category name** (e.g. "Food & Dining: ₹4,200")
- Spend totals **per month** (e.g. "2026-04: ₹38,000")
- Spend totals **per manual label name** (only labels *you typed*)
- Counts of bills, reminders, goal progress percentages, anonymised goal tags ("Goal A", "Goal B")

That's it. You can audit the exact prompt yourself, any time, from the AI Assistant screen — tap the shield icon (top right) to open **"What data is sent to your LLM"**. The prompt shown there is the prompt that will actually be sent.

### 3.2 What is NEVER sent to the LLM

- **SMS body text** — not a single character.
- **SMS sender ID** (e.g. `AX-AXISBK-S`).
- **Receiver / merchant name** (e.g. "Swiggy", "Mr. Patel"), in any field.
- **Transaction ids**.
- **Account or card numbers**, masked or unmasked.
- **PRAN, EPFO, or any government identifier**.
- **Goal titles or reminder titles** (sent as anonymised tags only).
- **Your name, email address, or phone number.**

### 3.3 What your LLM provider sees

Whatever you send via your API key is governed by your provider's privacy and retention terms. We have no relationship with that provider; the request goes from your phone to them, not via us. Treat the aggregated prompt above as if it were posted on a piece of paper to your provider's office — that is roughly the level of detail involved.

---

## 4. Permissions explained

| Permission | Why the app asks for it | Optional? |
|---|---|---|
| `READ_SMS`, `RECEIVE_SMS` | Read transactional SMS to build the dashboard. | No — without it the app cannot do its core job. You can revoke any time; existing data stays on your phone. |
| Storage (file picker) | Share / restore the `.mmbak` backup file. | Only used when you tap Export / Restore. |
| Foreground notifications | Daily summary; bill reminders. | Yes — can be disabled in Settings → Notifications. |
| Biometrics (`USE_BIOMETRIC`) | Optional unlock of the app or encrypted DB (future feature, currently unused). | Yes. |
| Internet | Only used when you have configured the AI assistant; otherwise the app makes zero network requests. | Yes — the app runs fully offline. |

---

## 5. No tracking, no ads, no analytics

- The app does not include any third-party analytics SDK (Firebase Analytics, Crashlytics, AppsFlyer, Mixpanel, Sentry, etc.).
- There are no ads.
- There is no crash-reporting service that uploads stack traces.
- We do not measure how often you open the app, which screens you visit, or what you tap.

If the app is offline-only on your device (no AI key configured), the app makes **zero outbound network requests**.

---

## 6. Children

The app is not directed at children under 13. We do not knowingly collect data from anyone — but in particular not from children.

---

## 7. Changes to this policy

If we change anything material — for example, if a future version of the app introduces a new permission, a new outbound network call, or a new type of stored data — this document will be updated and the effective date at the top will change. You'll see the new policy on the next app update.

---

## 8. Contact

Questions, concerns, or a data-deletion request?

**info.ai.mymoney@gmail.com**

A data-deletion request is straightforward: there's nothing on a server for us to delete. Uninstalling the app removes the local database. Any `.mmbak` backup files you exported live wherever *you* saved them (Drive, Gmail, PC) and are under your control.

---

_My Money is built and maintained as a personal-finance tool for the developer's own use and for friends and family. It does not exist to monetise your data. The privacy guarantees above aren't legal fine print — they are how the app actually works, and you can audit it: tap **AI Assistant → shield icon** to see exactly what the LLM gets._
