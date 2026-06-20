# Privacy Policy — My Money and (AI)

_Last updated: 2026-06-20_

**App:** My Money and (AI)
**Developer contact / privacy point of contact:** info.ai.mymoney@gmail.com

My Money and (AI) ("the app", "we", "us") is a personal-finance app that reads the **transactional** SMS already on your phone and turns it into a private spending dashboard. This document explains, in plain language, exactly what data the app accesses, where it lives, how long it is kept, how to delete it, and what (if anything) leaves your device.

The short version:

> **Your data stays on your phone.** The app has no server and no "My Money cloud." The only time anything leaves the device is when *you* trigger it — sharing a backup file, or asking the optional AI assistant a question (and even then we send only aggregated, anonymised figures over HTTPS).

---

## 1. What data the app reads

### 1.1 SMS messages (personal & sensitive data)

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

There is no upload of your inbox, ever. Your full SMS history stays in Android's SMS provider exactly where it was. The app never sends, writes, modifies, or deletes any SMS.

### 1.2 Background access (important disclosure)

So that new transactions appear without you opening the app, My Money and (AI) also scans for transactional SMS **in the background** — on a periodic schedule and after device restart (this is why the app requests `RECEIVE_BOOT_COMPLETED` and runs a scheduled background task). This background processing does the **same** local, on-device parsing described in 1.1: it reads only transactional messages, writes only the derived transaction rows to the local database, and **transmits nothing**. You are asked to grant SMS access through an in-app disclosure before any access begins, and you can revoke the permission at any time in Android Settings — background scanning then stops.

### 1.3 What we *don't* read

- Contacts, call logs, calendar, photos, location, microphone, camera — none of these are requested.
- Notifications from other apps — never read.
- Browsing history — never read.
- Persistent device identifiers (IMEI, IMSI, SIM serial, MAC, etc.) — never read or collected.

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

The AI Assistant is **off by default**. To enable it you paste your own API key from a provider you choose (Anthropic, OpenAI, etc.) in **Settings → AI Assistant**. We do not run an AI service of our own; your key calls your provider directly over **HTTPS**.

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
| `READ_SMS`, `RECEIVE_SMS` | Read transactional SMS to build the dashboard (foreground and background). | No — without it the app cannot do its core job. You can revoke any time; existing data stays on your phone. |
| `RECEIVE_BOOT_COMPLETED`, `WAKE_LOCK` | Re-arm the periodic background transaction scan after a restart. | Functional; tied to background scanning. |
| `SCHEDULE_EXACT_ALARM`, `USE_EXACT_ALARM` | Deliver bill/reminder notifications on time. | Yes — disable notifications to stop. |
| `POST_NOTIFICATIONS` | Show transaction and reminder notifications (Android 13+). | Yes — can be disabled in Settings → Notifications. |
| Storage (file picker) | Share / restore the `.mmbak` backup file. | Only used when you tap Export / Restore. |
| `USE_BIOMETRIC`, `USE_FINGERPRINT` | Optional app/database unlock. | Yes. |
| `INTERNET`, `ACCESS_NETWORK_STATE` | Only used when you have configured the AI assistant; otherwise the app makes zero network requests. | Yes — the app runs fully offline without an AI key. |

We use Android **runtime permission requests**, preceded by an in-app disclosure, before accessing SMS data.

---

## 5. Data security

- SMS parsing and all transaction storage happen **on-device**; the transaction database sits in the app's private, sandboxed storage and is not readable by other apps.
- Optional backups can be **encrypted** with AES-256-GCM (key derived via PBKDF2-HMAC-SHA256, 200 000 iterations, per-file salt).
- The only outbound network call (optional AI assistant) uses **HTTPS / modern TLS**.
- We do not transmit your SMS, account numbers, or government identifiers off the device under any circumstances.

---

## 6. Data retention and deletion

- **Retention:** Transaction data derived from SMS is retained locally on your device only, for as long as the app is installed, so your dashboard and history remain available offline. We hold **no copy** on any server, because there is no server.
- **Delete within the app:** You can wipe data at any time from **Settings** — "Delete all transactions" / "Clear all transaction data" / "Reload messages from SMS" — which remove the derived rows and cache from the local database.
- **Delete by uninstalling:** Uninstalling the app removes the entire local database (`my_money.db`) and all app data from your device.
- **Backups you exported** (`.mmbak`) live wherever *you* saved them (Drive, Gmail, PC) and are under your sole control; delete them in that location.
- Because no personal or sensitive data ever reaches us, a "delete my data" request has nothing for us to delete server-side. You can still contact us at the email below with any question.

---

## 7. Accounts

My Money and (AI) does **not** offer or require any user account, sign-in, or registration. There is no username, password, or profile stored by us, and therefore no online account to delete. All data is local to your device.

---

## 8. No tracking, no ads, no analytics, no third-party SDKs

- The app does not include any third-party analytics SDK (Firebase Analytics, Crashlytics, AppsFlyer, Mixpanel, Sentry, etc.).
- No advertising SDK is integrated and **no ads are served**; the app does not collect or use the Android Advertising ID (AAID) or the App Set ID.
- There is no crash-reporting service that uploads stack traces.
- We do not measure how often you open the app, which screens you visit, or what you tap.

If no AI key is configured, the app makes **zero outbound network requests**.

---

## 9. Financial and sensitive data

Financial information derived from your SMS is treated as sensitive. It is never publicly disclosed, never sold, and never shared with third parties for advertising or any other purpose. We do **not sell** personal or sensitive user data — there is no exchange or transfer of such data to any third party for monetary or other consideration.

---

## 10. International users and your rights

The app processes data **locally on your device** and we operate no server that receives your personal data, so we are not a data controller of any centrally-held personal data. Where applicable laws (such as the EU/UK GDPR or India's data-protection law) grant you rights over your personal data — including access, correction, and deletion — you can exercise them directly on the device (the data is in your hands), and you may contact us at the email below with any question. We comply with applicable privacy and data-protection laws in the jurisdictions where the app is offered.

---

## 11. Children

The app is not directed at children under 13. We do not knowingly collect data from anyone — but in particular not from children.

---

## 12. Changes to this policy

If we change anything material — for example, if a future version of the app introduces a new permission, a new outbound network call, an advertising or analytics SDK, or a new type of stored data — this document will be updated and the "Last updated" date at the top will change. You'll see the new policy on the next app update.

---

## 13. Contact

Questions, concerns, or a data-deletion request?

**info.ai.mymoney@gmail.com**

You can also raise an issue or feedback on our public tracker: https://github.com/infoaimymoney-lgtm/mymoney-support/issues

A data-deletion request is straightforward: there's nothing on a server for us to delete. Uninstalling the app removes the local database. Any `.mmbak` backup files you exported live wherever *you* saved them (Drive, Gmail, PC) and are under your control.

---

_My Money and (AI) is built and maintained as a personal-finance tool. It does not exist to monetise your data. The privacy guarantees above aren't legal fine print — they are how the app actually works, and you can audit it: tap **AI Assistant → shield icon** to see exactly what the LLM gets._
