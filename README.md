# the95xrecord-evidence

**Public, content-addressed copies of the evidence documents behind `https://the95xrecord.vercel.app`.**

Every file in [`sha256/`](sha256/) is named after **its own SHA-256 hash**. That is the whole design:

> If you doubt a document, you do not have to trust a website, a person, or a server.
> You hash the file you downloaded. If the hash equals the filename, you have the exact bytes we published.
> To make us wrong, you would have to find a SHA-256 collision.

This repository contains **no secrets, no session data, no private correspondence**. It contains public court
records and derived indexes of them, nothing else.

---

## What is here

| path | what it is |
| --- | --- |
| `sha256/<hash>.<ext>` | **80 evidence documents**, each byte-for-byte identical to the copy held on the station and mirrored to Box and Google Drive. The filename *is* the SHA-256 of the contents. |
| `CATALOGUE_80.csv` | the mapping: case · docket · grade → document name → bytes → **sha256** → GitHub path → intended R2 key |
| `MANIFEST.csv` | the same set in `MANIFEST` form (case, docket, grade, file, bytes, sha256) |
| `MANIFEST.sha256` | `sha256sum`-format checksums with paths relative to this repository root — one command verifies everything |
| `VERIFY.md` | how to verify one file, or all of them, in one command each |

**Totals:** 80 objects · 50,453,826 bytes · 80 unique hashes (no two documents share a hash).
Extension mix: 75 `.pdf`, 2 `.txt`, 2 `.md`, 1 `.csv`.

Files are addressed by hash **never by document name**. Names are metadata; the hash is the identity.
A document renamed anywhere still resolves to the same object here.

## Verify one file, in one command

```bash
# download one object and check its bytes against its own name
H=8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e
curl -sL "https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main/sha256/$H.pdf" -o "$H.pdf"
sha256sum "$H.pdf"          # must print  8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e
```

The printed hash must equal the filename. That is the entire proof.

## Verify all 80, in one command

```bash
git clone https://github.com/NikoPikoFriko/the95xrecord-evidence
cd the95xrecord-evidence
sha256sum -c MANIFEST.sha256      # 80 lines, each ending in OK
```

See [`VERIFY.md`](VERIFY.md) for the long form (per-file loop, Windows/PowerShell equivalent, and how to
check a single docket from `CATALOGUE_80.csv`).

## What this repo is not

- **Not the site.** The site is `https://the95xrecord.vercel.app`; this repository stores bytes only.
- **Not authoritative.** The authoritative copies are the court's and the station's case room. These are
  copies whose integrity you can check, not a claim of authenticity beyond the documents themselves.
- **Not a certified-copy claim.** "Certified" in `CATALOGUE_80.csv` means the *court clerk certified that
  particular copy* — the grade column records what the bytes are, and the ten marked `CERTIFIED` are the ones
  that carry the clerk's certification. The other documents are ordinary copies. We never label, and will
  never label, an uncertified copy as certified.
- **Not a mirror service.** There is no server here to fail. It is a git repository of bytes.

## Where these bytes also live

| mirror | state |
| --- | --- |
| station (case room / export) | ✅ source of truth for this publication |
| Box | ✅ via the station's Box helper (see `tools/verify_mirrors.py`) |
| Google Drive (`niko@mojehawaje.pl`) | ✅ via rclone |
| GitHub (this repo) | ✅ public, content-addressed |
| Cloudflare R2 | ⏳ same keys (`/sha256/<hash>.<ext>`) — awaiting a Cloudflare API token; the catalogue is already R2-ready, so the copy is mechanical |

## Provenance

The set is exactly the 80 documents published on the site's `/access` route, verified in task-49:
80 rows from the site's `MANIFEST.csv`, every SHA-256 recomputed from the bytes (not copied from any list),
**80/80 matching their declared size and hash**, total 50,453,826 bytes.

Publication verified after the push by **re-downloading every object from this repository and re-hashing the
served bytes** — see `PUBLIC_BYTES_REPORT.md` on the station for the per-file table.

*No credentials, tokens or private data are present in this repository.*
