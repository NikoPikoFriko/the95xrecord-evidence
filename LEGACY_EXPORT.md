# Legacy export — bytes outside the capture ledger

**What this is.** On 2026-11-03 the commit `9f12298` ("Publish 80 evidence documents under content
addresses") published 80 documents to this repository as content-addressed bytes. That export predates the
capture procedure: the files were imported in bulk from the evidence export, not captured from JIMS through
the station's browser flow. **29 of those 80 files are still here with no receipt and no ledger entry** (three of the original 37 later acquired capture receipts and are published normally).

**Why they have no receipt.** A receipt is a record of a *capture*: a browser download carrying a
`Zone.Identifier` ADS with the JIMS `HostUrl`, a page-1 case gate, and a capture-ledger row. For these 29
objects none of that ever existed:

* no receipt has ever been committed for them (`git log --all -- receipts/<sha>.json` is empty);
* they are absent from `captures/ledger.jsonl` and `captures/ledger_resolved.jsonl`;
* no staged copy carries a `Zone.Identifier` ADS (the two staged copies that exist have empty stream lists);
* they came from the bulk export, and their bytes still match `CATALOGUE_80.csv` and the local
  `evidence-access/documents/` export.

**What it means for an audit.** Their bytes are public and mirror-verified (local, GitHub, GitHub-served,
Drive, Box all MATCH), but **they are not evidence of capture**. Nothing here claims they were retrieved
from the court through the procedure, and no receipt was fabricated for them. A reader who needs a capture
record for one of these dockets should treat it as **not captured**, only published as a legacy export.

**How the site treats them.** `captures/publication_view.jsonl` does not list them as published; they show as
`capturer_only`/unknown. That is accurate: the publication ledger — the only source of `published: true` —
has no row for them.

**How the verifier treats them.** `tools/jims/verify_installation.py` check #16 accepts an orphan byte file
**only** when its sha256 appears exactly in [`LEGACY_EXPORT.json`](LEGACY_EXPORT.json) with a non-empty
`reason` and the file's `source_commit` is set. Everything else is a failure. The exception is narrow by
construction:

* an orphan **not** listed → `bytes-without-receipt` FAIL (the T80 incident class);
* a listed object that **later gains a receipt** → consistency error: it cannot be both a legacy exception
  and a capture — remove it from this list rather than keeping the exception;
* the count is reported on every run as `known legacy exceptions: 29`.

Files: [`LEGACY_EXPORT.json`](LEGACY_EXPORT.json) (machine-readable, one entry per object) ·
`LEGACY_EXPORT.md` (this page).
