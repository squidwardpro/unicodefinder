 # Email Encoding & Phishing Test

Internal tool for detecting Unicode obfuscation, encoding tricks, and phishing indicators in real emails. Hosted at [squidward.pro](https://squidward.pro) — our company domain, maintained by the security team.

---

## How It Works

```
Copy three recent emails → Paste into squidward.pro → Review analysis
```

---

## Instructions

### Step 1 — Copy Your Three Most Recent Emails

1. Open your email client (Gmail, Outlook, etc.).
2. Copy your three most recent emails — sender, subject, body, and any visible links.
3. Redact credentials or personal data before pasting.

### Step 2 — Submit

1. Go to [https://squidward.pro](https://squidward.pro) (our internal scanning tool).
2. Paste each email into the input field, one at a time.
3. Click **Scan**.

### Step 3 — Review Results

The scanner analyses each email for:

**Unicode Anomalies** — Homoglyphs (Cyrillic vs Latin), zero-width characters, mixed encoding.

**Link Inspection** — Display vs actual URL mismatches, suspicious domains, URL encoding tricks.

**Header & Metadata** — Sender spoofing, domain inconsistencies, reply-to mismatches.

**Phishing Heuristics** — Urgency language, credential harvesting patterns, known signatures.

---

## Output

The tool returns a risk score (Low / Medium / High), detected encoding anomalies, extracted and analysed links, phishing indicators, and recommendations.

---

## Example Attack

```
Displayed:  https://paypaI.com
Actual:     https://paypal.com
```

Capital `I` vs lowercase `l` — easy to miss, commonly exploited.

---

## What to Look For

Characters substituted from other scripts, redirecting links, domain misspellings (e.g., `micros0ft.com`), and unusual formatting or spacing.

---

## Developer Notes

Ensure proper UTF-8 / UTF-16 decoding and Unicode normalisation (NFC/NFKC). Useful libraries: `unicodedata` (Python), `punycode` / IDNA decoding. Always normalise domains before comparison.

---

## Data Handling

Submitted content is processed in an isolated environment and not stored permanently. Logs are anonymised. Questions — contact the security team.

---

 here is the code used on the backend scanner. this is shown for transparency 
 ```
  import unicodedata                                                                                                                                                                                                                                
  from email import policy                                                                                                                                                                                                                          
  from email.parser import BytesParser                                                                                                                                                                                                              
                                                                                                                                                                                                                                                    
  SUSPICIOUS_RANGES = {                                                                                                                                                                                                                             
      'RTL/LTR Override': [0x200E, 0x200F, 0x202A, 0x202B, 0x202C, 0x202D, 0x202E, 0x2066, 0x2067, 0x2068, 0x2069],                                                                                                                                 
      'Zero-Width': [0x200B, 0x200C, 0x200D, 0xFEFF],                                                                                                                                                                                               
      'Homoglyph Latin-like': list(range(0x0400, 0x04FF+1)) + list(range(0x0370, 0x03FF+1)),                                                                                                                                                        
      'Confusable Punctuation': [0x2018, 0x2019, 0x201C, 0x201D, 0x2013, 0x2014, 0x2026, 0x00AB, 0x00BB],                                                                                                                                           
  }                                                                                                                                                                                                                                                 
                                                                                                                                                                                                                                                    
  def classify_char(cp):                                                                                                                                                                                                                            
      for category, codepoints in SUSPICIOUS_RANGES.items():                                                                                                                                                                                        
          if cp in codepoints:                                                                                                                                                                                                                      
              return category                                                                                                                                                                                                                       
      if cp > 0xFFFF:                                                                                                                                                                                                                               
          return 'Supplementary Plane (Emoji/Symbol)'                                                                                                                                                                                               
      if cp > 0x7F:                                                                                                                                                                                                                                 
          return 'Extended Unicode'                                                                                                                                                                                                                 
      return None                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                    
  def scan_unicode(text):                                                                                                                                                                                                                           
      findings = []                                                                                                                                                                                                                                 
      for i, ch in enumerate(text):                                                                                                                                                                                                                 
          cp = ord(ch)                                                                                                                                                                                                                              
          if cp > 0x7F:                                                                                                                                                                                                                             
              category = classify_char(cp)                                                                                                                                                                                                          
              try:                                                                                                                                                                                                                                  
                  name = unicodedata.name(ch, f'U+{cp:04X}')                                                                                                                                                                                        
              except ValueError:                                                                                                                                                                                                                    
                  name = f'U+{cp:04X}'                                                                                                                                                                                                              
              findings.append({                                                                                                                                                                                                                     
                  'position': i,                                                                                                                                                                                                                    
                  'char': ch,                                                                                                                                                                                                                       
                  'codepoint': f'U+{cp:04X}',                                                                                                                                                                                                       
                  'name': name,                                                                                                                                                                                                                     
                  'category': category or 'Extended Unicode',                                                                                                                                                                                       
                  'context': text[max(0,i-20):i+20],                                                                                                                                                                                                
              })                                                                                                                                                                                                                                    
      return findings                                                                                                                                                                                                                               
                                                                                                                                                                                                                                                    
  def parse_email_content(raw_bytes):                                                                                                                                                                                                               
      msg = BytesParser(policy=policy.default).parsebytes(raw_bytes)                                                                                                                                                                                
      headers = {}                                                                                                                                                                                                                                  
      for h in ['From', 'To', 'Subject', 'Date', 'Reply-To', 'Return-Path', 'Message-ID']:                                                                                                                                                          
          val = msg.get(h)                                                                                                                                                                                                                          
          if val:                                                                                                                                                                                                                                   
              headers[h] = str(val)                                                                                                                                                                                                                 
      body_text = ''                                                                                                                                                                                                                                
      if msg.is_multipart():                                                                                                                                                                                                                        
          for part in msg.walk():                                                                                                                                                                                                                   
              ct = part.get_content_type()                                                                                                                                                                                                          
              if ct == 'text/plain':                                                                                                                                                                                                                
                  body_text += part.get_content()                                                                                                                                                                                                   
              elif ct == 'text/html' and not body_text:                                                                                                                                                                                             
                  body_text += part.get_content()                                                                                                                                                                                                   
      else:                                                                                                                                                                                                                                         
          body_text = msg.get_content()                                                                                                                                                                                                             
      return headers, body_text                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                    
  def scan_email(filepath):                                                                                                                                                                                                                         
      with open(filepath, 'rb') as f:                                                                                                                                                                                                               
          raw_bytes = f.read()                                                                                                                                                                                                                      
      try:                                                                                                                                                                                                                                          
          headers, body_text = parse_email_content(raw_bytes)                                                                                                                                                                                       
      except Exception:                                                                                                                                                                                                                             
          headers = {}                                                                                                                                                                                                                              
          body_text = raw_bytes.decode('utf-8', errors='replace')                                                                                                                                                                                   
                                                                                                                                                                                                                                                    
      header_findings = []                                                                                                                                                                                                                          
      for hdr_name, hdr_val in headers.items():                                                                                                                                                                                                     
          for f in scan_unicode(hdr_val):                                                                                                                                                                                                           
              f['location'] = f'Header: {hdr_name}'                                                                                                                                                                                                 
              header_findings.append(f)                                                                                                                                                                                                             
                                                                                                                                                                                                                                                    
      body_findings = scan_unicode(body_text)                                                                                                                                                                                                       
      for f in body_findings:                                                                                                                                                                                                                       
          f['location'] = 'Body'                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                    
      return headers, body_text, header_findings + body_findings                                                                                                                                                                                    
                                                                                                                                                                                                                                                    
  if __name__ == '__main__':                                                                                                                                                                                                                        
      import sys  
      if len(sys.argv) < 2:                                                                                                                                                                                                                         
          print('Usage: python scanner.py <email.eml>')                                                                                                                                                                                             
          sys.exit(1)                                                                                                                                                                                                                               
      headers, body, findings = scan_email(sys.argv[1])                                                                                                                                                                                             
      print(f'Headers: {len(headers)}  |  Unicode chars found: {len(findings)}')                                                                                                                                                                    
      for f in findings:                                                                                                                                                                                                                            
          suspicious = ' << SUSPICIOUS' if f['category'] in ('RTL/LTR Override', 'Zero-Width', 'Homoglyph Latin-like') else ''                                                                                                                      
          print(f"  [{f['codepoint']}] {f['name']:<40} | {f['category']}{suspicious}")                                                                                                                                                              
                                                                                                                                             
 ```

## Feedback

Found an edge case? Report it in the internal security channel with a sanitised sample, scanner output, and description of the issue.
