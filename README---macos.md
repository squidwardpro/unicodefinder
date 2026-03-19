# Email Encoding & Phishing Safety Test

An internal tool for identifying hidden malicious content, Unicode obfuscation, and phishing indicators in real-world emails.

---

## Purpose

Modern phishing attacks often use techniques such as Unicode homoglyphs (e.g., Cyrillic `а` vs Latin `a`), hidden or zero-width characters, encoded or misleading URLs, and spoofed sender information. This test helps ensure our systems — and our team — can detect these threats effectively.

---

## Safety Guidelines

**Before submitting any email for scanning, follow these rules:**

- Do **not** submit emails containing passwords, credentials, or personal data.
- Do **not** submit confidential or privileged communications.
- Do **not** include attachments.
- Prefer low-risk emails such as newsletters, automated notifications, or marketing messages.
- When in doubt, redact any sensitive content before submission.

---

## Test Overview

```
Select a recent email → Paste into scanner → Analyse encoding → Review results
```

---

## Instructions

### Step 1 — Retrieve a Recent Email

1. Open your email client (e.g., Gmail, Outlook).
2. Select a recent, non-sensitive email.
3. Copy the relevant content, including sender, subject line, body text, and any visible links.

### Step 2 — Submit to Internal Scanner

1. Open the internal scanning tool at:

   ```
   https://squidward.pro
   ```

   > **Verification:** This is a company-managed internal tool maintained by the security team. If you have any concerns about the legitimacy of this URL, confirm with your team lead or the security channel before proceeding.

2. Paste the copied email content into the input field.
3. Click **Scan**.

### Step 3 — Review Analysis Results

The scanner checks for the following:

**Unicode Anomalies** — Homoglyph attacks (Cyrillic vs Latin characters), zero-width or invisible characters, and mixed encoding patterns.

**Link Inspection** — Mismatched display vs actual URLs, suspicious domains, and URL encoding tricks.

**Header & Metadata Checks** — Sender spoofing indicators, domain inconsistencies, and reply-to mismatches.

**Phishing Heuristics** — Urgency language, credential harvesting patterns, and known phishing signatures.

---

## What to Look For

Be alert for characters that appear normal but are substituted from other scripts, links that redirect to unexpected destinations, slight misspellings in domains (e.g., `micros0ft.com`), and unusual formatting or spacing in the email body.

---

## Expected Output

The tool returns a risk score (Low / Medium / High), a list of detected encoding anomalies, extracted and analysed links, potential phishing indicators, and recommendations for next steps.

---

## Privacy & Data Handling

- Submitted email content is processed in an isolated environment and is **not stored permanently**.
- Logs are anonymised where possible.
- If you have questions about how submitted data is handled, contact the security team directly.

---

## Developer Notes

When working with encoding detection, ensure proper handling of UTF-8 and UTF-16 decoding as well as Unicode normalisation (NFC/NFKC). Useful libraries include `unicodedata` (Python) and `punycode` / IDNA decoding modules. Always normalise and compare domains before analysis.

---

## Example Attack

**Displayed URL:**

```
https://paypaI.com
```

**Actual URL:**

```
https://paypal.com
```

The capital `I` is visually similar to lowercase `l`, making the spoofed domain difficult to spot at a glance.

---

## Goal

By completing this test, developers should understand how phishing emails evade detection, recognise encoding-based attacks, and improve defensive tooling and awareness across the team.

---

## Feedback

If you discover issues or edge cases, report them via the internal security channel. Include a sanitised sample email, the scanner output, and a description of the observed issue.
