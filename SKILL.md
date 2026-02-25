---
name: agentic-letters
description: "Send physical letters anywhere in Germany with a single command. Use when: user wants to send a letter, cancellation (Kündigung), DSGVO request, or any physical mail. Requires an API key from agentic-letters.com."
homepage: https://agentic-letters.com
metadata:
  {
    "openclaw":
      {
        "emoji": "✉️",
        "requires": { "bins": ["python3"] },
      },
  }
---

# SKILL.md - AgenticLetters

Send physical letters anywhere in Germany with a single command.

## When to use

- User wants to send a physical letter, cancellation, or legal notice
- User needs to mail a document (PDF) to a German address
- User says "send a letter", "Kündigung schicken", "Brief versenden"
- User wants to mail a DSGVO request, cancellation, complaint, or greeting card

## Setup

Store your API key:

```bash
mkdir -p ~/.openclaw/secrets
echo 'AGENTIC_LETTERS_API_KEY=al_your_api_key' > ~/.openclaw/secrets/agentic_letters.env
```

Get your API key at https://agentic-letters.com/buy

## Tool

`agentic_letters.py` — a zero-dependency Python CLI (stdlib only).

## Send a letter

1. Generate an A4 PDF (max 3 pages, max 10 MB, black & white print)
2. Run the tool:

```bash
python3 agentic_letters.py send \
  --pdf letter.pdf \
  --name "Max Mustermann" \
  --street "Musterstraße 1" \
  --zip 10115 \
  --city Berlin \
  --label "Kündigung Fitnessstudio"
```

Optional flags:
- `--type <type>` — letter type (default: `standard`). New types will be added over time; the API rejects unknown types with a list of valid ones.
- `--country <code>` — country code (default: `DE`). Currently only Germany is supported.

Output (JSON to stdout):
```json
{
  "id": "550e8400-e29b-41d4-a716",
  "status": "queued",
  "type": "standard",
  "label": "Kündigung Fitnessstudio",
  "created_at": "2026-02-24T19:00:00Z",
  "credits_remaining": 4
}
```

## Check letter status

```bash
python3 agentic_letters.py status <letter-id>
```

Status values: `queued` → `printed` → `sent` → `returned`

## Check remaining credits

```bash
python3 agentic_letters.py credits
```

## List all letters

```bash
python3 agentic_letters.py list
```

## Generating PDFs

If the user doesn't have a PDF ready, generate one:

- `pandoc` for markdown → PDF: `echo "Dear Sir..." | pandoc -o letter.pdf`
- `wkhtmltopdf` for HTML → PDF: `wkhtmltopdf letter.html letter.pdf`
- Python with `fpdf2` or `reportlab` for programmatic generation

Always ensure A4 format (210 × 297 mm) with at least 15 mm margins.

## Error handling

Errors go to stderr with a clear origin tag. The exit code is non-zero on failure.

**Origins:**
- `[local]` — problem before the request (missing file, no API key)
- `[server]` — the API rejected the request (includes error code, HTTP status, detail, and field)
- `[network]` — could not reach the API (DNS, timeout, connection refused)

Example server error:
```
[server] Invalid German postal code
  code: recipient_zip_invalid
  http_status: 400
  detail: Expected a 5-digit German PLZ (e.g. "10115"), got "123".
  field: recipient.zip
```

On success, JSON is printed to stdout. On failure, nothing goes to stdout.

## Important constraints

- **Germany only** — recipient must have a German address
- **Max 3 pages** — longer PDFs are rejected by the server
- **Max 10 MB** — compress images if needed
- **Black & white** — images are printed in grayscale
- **1 credit = 1 letter** — check credits before sending
- **A4 format** — ensure correct page size
- **Do not validate the PDF locally** — the server handles all PDF validation

## Typical workflows

**Kündigung (cancellation):**
Ask for: service name, customer number, recipient address. Generate a formal cancellation letter as PDF, send it.

**DSGVO Auskunftsersuchen (data access request):**
Ask for: company name, address, user's full name. Generate a DSGVO Art. 15 request letter, send it.

**Widerspruch (objection/appeal):**
Ask for: authority/company, reference number, reason. Generate a formal objection letter, send it.
