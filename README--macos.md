# Email Encoding & Phishing Safety Test

This project helps identify hidden malicious content, Unicode obfuscation, and phishing indicators in real-world emails using our internal scanning tool.

## Purpose

Modern phishing attacks often use:

- Unicode homoglyphs (e.g., Cyrillic `а` vs Latin `a`)
- Hidden characters or zero-width spaces
- Encoded URLs or misleading links
- Spoofed sender information

This test helps ensure our systems — and our developers — can detect these threats effectively.

## Safety Guidelines

**Before submitting any email content, you MUST follow these rules:**

- **NEVER** submit emails containing passwords, credentials, API keys, or tokens
- **NEVER** submit emails with personal data (addresses, phone numbers, financial info)
- **NEVER** include attachments or attachment contents
- **NEVER** submit confidential or privileged communications
- **ALWAYS** redact names, internal project names, or sensitive identifiers before pasting
- **PREFER** low-risk emails such as newsletters, marketing emails, or automated notifications
- **WHEN IN DOUBT**, redact aggressively — the encoding analysis works on structure, not content

## Test Overview

```
Select a recent email → Redact sensitive content → Paste into scanner → Analyze encoding → Review results
```

## Instructions

### Step 1 — Retrieve a Recent Email

1. Open your email client (e.g., Gmail, Outlook)
2. Select a recent, non-sensitive email (newsletters and automated messages work best)
3. Copy the relevant email content:
   - Sender address
   - Subject line
   - Body text
   - Any visible links
4. **Before proceeding:** review what you've copied and redact anything sensitive

### Step 2 — Submit to Internal Scanner

1. Navigate to the company's internal scanning tool:

```
https://squidward.pro
```

> **Verification:** This is an internally maintained tool operated by the security team. If you have any doubts about the URL, confirm with your team lead or the `#security` channel before proceeding. Do not use this tool if the URL has been modified or redirects unexpectedly.

2. Paste the copied (and redacted) email into the input field
3. Click **Scan**

### Step 3 — Review Analysis Results

The scanner checks for the following:

**Unicode Anomalies** — Homoglyph attacks (Cyrillic vs Latin characters), zero-width or invisible characters, and mixed encoding patterns.

**Link Inspection** — Mismatched display vs actual URLs, suspicious domains, and URL encoding tricks.

**Header & Metadata Checks** — Sender spoofing indicators, domain inconsistencies, and reply-to mismatches.

**Phishing Heuristics** — Urgency language, credential harvesting patterns, and known phishing signatures.

## What to Look For

Be alert for:

- Characters that look normal but aren't
- Links that redirect unexpectedly
- Slight misspellings in domains (e.g., `micros0ft.com`)
- Unusual formatting or spacing

## Expected Output

The tool will return:

- **Risk score** — Low / Medium / High
- **Detected encoding anomalies**
- **Extracted and analyzed links**
- **Potential phishing indicators**
- **Recommendations**

## Privacy & Data Handling

- Submitted content is processed in a secure, isolated environment
- No email content is stored beyond the duration of the scan session
- Logs are anonymized — no submitted email content is written to logs
- If you have questions about data handling, contact the security team directly

> **Note:** Even with these protections, never rely solely on infrastructure safeguards. Always redact sensitive content before submission.

## Developer Notes

When working on or extending this tool, ensure proper handling of:

- UTF-8 / UTF-16 decoding
- Unicode normalization (NFC / NFKC)
- Domain normalization and comparison before analysis

Useful libraries:

- `unicodedata` (Python)
- `punycode` / IDNA decoding

## Example Attack

**Displayed:**
```
https://paypaI.com
```

**Actual:**
```
https://paypal.com
```

The capital `I` is visually identical to a lowercase `l` in many fonts — this is a classic homoglyph attack.

## Goal

By completing this test, developers should:

- Understand how phishing emails evade detection
- Recognize encoding-based attacks
- Improve defensive tooling and awareness

## Feedback & Reporting

If you discover issues or edge cases, report via the internal `#security` channel and include:

- A sanitized sample of the email (with all sensitive content removed)
- The scanner output
- A description of the observed issue
