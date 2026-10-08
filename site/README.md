# MARS test site

Static site the MARS agent downloads from and reads through Menlo isolation:
download DLP, redaction, Adaptive DLP and prompt-injection tests.
All data is **synthetic**: fictitious people, network test card numbers, and
AWS's documented example key. Never add real data here, since the site is public.

## Contents

| Path | Purpose |
|---|---|
| `index.html` | Download page. Links have stable ids: `dl-<id>` downloads, `view-<id>` opens a PDF inline |
| `files/dl01`–`dl02`, `dl07` | PII (SSN): xlsx, csv, and nested in a zip |
| `files/dl03`–`dl04` | PCI (test card numbers): docx, pdf |
| `files/dl05` | Secrets / credentials (Adaptive DLP dictionaries) |
| `files/dl06` | Redaction case: sensitive values inside normal prose, with KEEP canaries that must survive |
| `files/dl08`–`dl09` | Prompt injection hidden in docx/pdf (hidden, white, 1pt text, metadata) |
| `files/dl10`–`dl11` | Clean controls, the validity anchors |
| `inject.html` | Page with prompt injection hidden 9 ways, plus visible controls |
| `redact.html` | Form that POSTs typed text to an echo service (httpbin.org) so redaction is observable |
| `manifest.json` | Origin SHA-256, sensitive strings and canaries for every test, read by the agent |
| `_build/build_site.py` | Regenerates everything (deterministic). Not served by GitHub Pages |

Every test string carries a `MARS-CANARY-…` token. The agent checks which
tokens survive the trip through isolation. A hidden canary that disappears
means masking or CDR worked. A visible or KEEP canary that disappears means
the content was over-blocked.

## Publish on GitHub Pages (one time, ~10 minutes)

1. github.com → **New repository** → name `mars-test-files`, **Public** → Create.
2. **Add file → Upload files** → drag in **everything inside this `site/` folder**
   (including `files/` and `_build/`) → **Commit changes**.
3. **Settings → Pages** → Source: **Deploy from a branch** → `main` / `(root)` → **Save**.
4. After about a minute: `https://<your-github-username>.github.io/mars-test-files/`

Check it in a normal browser: the index lists 11 files and two test pages.

## Before running the agent against it, have the SE confirm the policy

- `<your-github-username>.github.io` → **Isolate** (not Allow) for the agent identity,
  with download DLP and CDR on. If it's Allow, nothing is inspected and every
  test "passes" for the wrong reason.
- `httpbin.org` → **Isolate**, with input/text DLP on (it's the destination for `redact.html`).
- Prompt-injection protection (**Mask** or **Block**) enabled for page content and files.
- Which dictionaries are active (PII, PCI, secrets/credentials) and whether each
  action is **Block** or **Redact**. The expected results depend on this.

## Regenerating

```bash
python3 site/_build/build_site.py
```

Same input gives byte-identical files and hashes. After a rebuild, upload the
changed files again.
