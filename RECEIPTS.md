# RECEIPTS.md — where each byte came from, and how to check it

Every object in [`sha256/`](sha256/) that carries a court receipt has a matching file
[`receipts/<sha256>.json`](receipts/). The receipt is the capture record: the **JIMS servlet URL** the bytes
were retrieved from, the **raw NTFS `Zone.Identifier` string verbatim**, the timestamp, the docket/code/filed
as the court page rendered them, and the sha256/bytes of the file that landed.

`receipts/index.json` is the machine index: `sha256 → {docket, code, filed, tier, servlet_url, captured_at}`.
`RECEIPTS_MANIFEST.sha256` lets you verify the receipts themselves.

---

## The 5-step recipe

1. Sign in to JIMS at <https://jimspss1.courts.state.hi.us/JIMSExternal/manage-cases.iface> (a session is required — see the limit below).
2. **My Case View** → open the case named in `index.json` → **Dockets** (trial documents are under **Documents**).
3. Find the docket row the receipt names and retrieve the document — use the same button kind the receipt records in `court.servlet` (`regular` vs `certified`).
4. `sha256sum <downloaded-file>` — on Windows: `(Get-FileHash -Algorithm SHA256 <file>).Hash.ToLower()`.
5. Compare that hash with the **filename** in `sha256/` and with `sha256` in the receipt. Equal = the same bytes.

If the hash matches, provenance and integrity are both closed: the court served those bytes, and the bytes are
the ones published here. If it does not match, that is a finding worth reporting — either the court row changed
(a new rendition) or the download was corrupted.

```bash
# one object, end to end
H=$(python -c "import json;print(json.load(open('receipts/index.json'))['receipts'].popitem()[0])")
curl -sL "https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main/receipts/$H.json" -o r.json
curl -sL "https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main/sha256/$H.pdf" -o o.pdf
sha256sum o.pdf          # must equal $H, and equal .sha256 in r.json
jq -r '.court.servlet_url, .court.captured_at, .court.ads_verbatim' r.json
```

---

## The honest limit — read this before quoting a receipt

**The servlet URLs are session-bound.** They are not anonymously fetchable: without a signed-in JIMS session the
certified servlet answers `HTTP 500` and the regular servlet returns an **88-byte HTML shell** instead of a
document (measured, not assumed). The `id=` parameter is tied to the live session and may expire.

So a receipt proves exactly this, and no more:

> **at the recorded timestamp, in the recorded JIMS session, the court's DocumentRetrieval servlet served these
> bytes for this docket row.**

It does **not** prove, and must never be read as proving:

- that the URL still works today (it may not — that is expected, not tampering);
- that the document is authentic beyond what the court's own record says;
- that a `certified` servlet capture is a certified copy — the *grade* in `CATALOGUE_80.csv` records what the
  bytes are; only the ten records carrying the Clerk's certification sentence on page 1 are certified copies.

**What is unconditionally verifiable is the byte identity**: the filename *is* the SHA-256, so anyone can hash the
file they have and compare. The receipt adds *where it came from*; content addressing proves *what it is*.
Keep the two claims separate — conflating them is the failure mode this repository exists to avoid.

## Append-only

A receipt is never edited. If a later capture contradicts an earlier one, the new receipt is written and carries
`supersedes` naming the previous receipt's path, batch and timestamp. Corrections are additions, not rewrites.

## What has no receipt (and why)

Objects published before court-capture telemetry existed have no receipt. They are still content-addressed and
independently hash-verifiable; they are simply not *pinned to a retrieval URL*. `CATALOGUE_80.csv` column
`receipt` is empty for those, and `receipts/index.json` does not list them — absence is visible, not implied.

*No credentials or session tokens are stored in this repository. `jims_session` in a receipt is a JIMS UI
session label as rendered at capture time, not a usable credential.*

## Rendition pairs

One document row can yield more than one set of bytes. Each set is a separate object with its own
receipt and its own SHA-256; they are NOT duplicates and NOT corrections of each other. The pair is
declared by the publication view, and the receipts copy those fields verbatim.

| group | sha256 (first 16) | pages | bytes | ADS id | rendition |
|---|---|---|---|---|---|
| `1RC191003776:496:` | `5ed5e1968bcfcf28` | - | 380,102 | `-4148043684089643505` | primary |
| `1RC191003776:496:` | `afda0dbbbd371f6e` | - | 133,136 | `-6724965835921518722` | additional |
| `1RC191003776:9:DOC` | `20ca4fda20dace9e` | 2 | 797,009 | `2252464258259662806` | primary |
| `1RC191003776:9:DOC` | `6f4899057046964f` | 6 | 1,409,566 | `-1348131103579714594` | additional |
| `1RC191003776:ICA:` | `267462accd55e8f6` | - | 27,362 | `-6025935909910058506` | primary |
| `1RC191003776:ICA:` | `27369ac930c946fe` | - | 6,034 | `1151383558689383108` | additional |
| `1RC191003776:18:NEF` | `89bf637abbdfa961` | 2 | 4,236 | `-702025660265980255` | primary |
| `1RC191003776:19:MDJ` | `9c1dcaa8c79079bb` | 42 | 1,620,945 | `-6277603892225234157` | primary |
| `1RC191003776:494:PROP` | `dade2925abaf8b35` | 20 | 2,087,765 | `839617491208344317` | pending |
| `1RC191003776:495:NEF` | `d0cf44d711ed14c3` | 2 | 25,626 | `-2132126284213201075` | pending |
| `1RC191003776:114:ORD` | `5085ae3dc2b83d13` | 3 | 676,400 | `4744514484897820887` | primary |
| `1RC191003776:132:ORDG` | `e061f052aa09f8a8` | 3 | 684,988 | `2937727878503168956` | primary |
| `1RC191003776:134:ORD` | `1371a51ad317024a` | 3 | 681,237 | `5605136942311343171` | primary |

6 objects in 3 rendition groups. dkt 9 (1RC191003776) is the case that made
this table necessary: `6f489905…` is 6 pages and `20ca4fda…` is 2 pages of the same DOC — different
bytes, different rendition, each with its own ADS receipt.
