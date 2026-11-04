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
| `1RC191003776:8:ROS` | `768271d05442425f` | 6 | 217,240 | `-2876578835718946788` | primary |
| `1RC191003776:16:NEF` | `b8b0bd881b91cbe4` | 3 | 717,502 | `-4771073404671874446` | pending |
| `1RC191003776:17:OT` | `1216f6117eac5e24` | 12 | 3,837,927 | `4766590497954969979` | primary |
| `1RC191003776:28:NTCE` | `0e54f37f8eef46ce` | 1 | 391,392 | `8518093925117452384` | primary |
| `1RC191003776:31:NEF` | `cbe17d37c4461782` | 2 | 4,425 | `-4295213209181548085` | primary |
| `1RC191003776:32:PSTMT` | `90d0b880e3e2790f` | 4 | 139,708 | `-25444717309201708` | primary |
| `1RC191003776:29:NEF` | `2696cd9e22cd80ac` | 1 | 560,929 | `4017798341672112210` | primary |
| `1RC191003776:15:OT` | `eb8d4531aca65b59` | 3 | 52,122 | `7750358205010716444` | primary |
| `1RC191003776:20:NEF` | `83ddf34dba91407b` | 2 | 4,316 | `-3843977836804516787` | primary |
| `1RC191003776:244:ORDG` | `f28f11fb32264249` | 2 | 424,937 | `-6865042206930483200` | primary |
| `1RC191003776:334:SCON` | `e8a60c3438e197ac` | 2 | 411,897 | `4601022016122520957` | primary |
| `1RC191003776:336:SCON` | `e0517fdcd399613f` | 2 | 605,714 | `-6124888902046763277` | primary |
| `1RC191003776:434:NEF` | `2b7a230198c3b42b` | 2 | 7,012 | `-5861320999948940764` | primary |
| `1RC191003776:435:DOC` | `ceff5d911a2c29fb` | 11 | 2,922,517 | `-3708604582101802005` | primary |
| `1RC191003776:436:NEF` | `4acb689fe87d5e68` | 2 | 6,711 | `-4428057794105905643` | primary |
| `1RC191003776:437:DOC` | `b1c916c1829c893f` | 27 | 8,273,273 | `-4583325873678444182` | primary |
| `1RC191003776:438:NEF` | `bf976116b27b690c` | 2 | 6,721 | `3029127306147235728` | primary |
| `1RC191003776:432:MOT` | `3b27f3eadf55a8d5` | 10 | 133,410 | `-6808358543498790237` | primary |
| `1RC191003776:433:MOT` | `487eaf0cb7cf48af` | 37 | 6,776,950 | `-1141983175788426948` | primary |
| `1RC191003776:473:NEF` | `cb703f16e8605801` | 2 | 6,956 | `7148816620299916817` | primary |
| `1RC191003776:466:MOT` | `8d273740bfd1570f` | 2 | 6,351 | `3920425210463038782` | primary |
| `1RC191003776:468:MOT` | `0bdd58e87ccb69d1` | 7 | 632,503 | `4592066290144680272` | primary |
| `1RC191003776:475:NEF` | `9be1765cb599ba17` | 2 | 6,937 | `-3652923381126334150` | primary |
| `1RC191003776:474:MOT` | `e1da55f73f299ee7` | 5 | 230,967 | `7979009138876147612` | primary |
| `1RC191003776:476:MOT` | `109ddd2a24ccdc53` | 2 | 90,307 | `5512801541774547381` | primary |
| `1RC191003776:477:NEF` | `a5c291feebbef9a0` | 2 | 6,673 | `368281696223305538` | primary |
| `1RC191003776:461:MOT` | `6f7fc7455ddddaa0` | 2 | 190,628 | `-310188176449549074` | primary |
| `1RC191003776:483:NEF` | `28811970ab089be4` | 2 | 6,885 | `2726093706829294060` | primary |
| `1RC191003776:479:NEF` | `45e3ee7186b3c239` | 2 | 6,698 | `-1918554894364169679` | primary |
| `1RC191003776:482:MOT` | `5918e57936422bed` | 7 | 226,914 | `5927767913407640609` | primary |
| `1RC191003776:484:MOT` | `80a5edc13c962ceb` | 4 | 762,823 | `-4912384106686638124` | primary |
| `1RC191003776:451:MOT` | `a6c06756ea15df09` | 18 | 625,537 | `-2894608197161397327` | primary |
| `1RC191003776:441:MOT` | `b995effe382f146a` | 10 | 133,631 | `147726643068267451` | primary |
| `1RC191003776:492:DOC` | `6e956ef1189c1092` | 4 | 1,068,894 | `-3575177123423105830` | primary |
| `1RC191003776:19:MDJ` | `46429ea256f90049` | 5 | 156,010 | `6028049872815053193` | pending |
| `1RC191003776:496:NA` | `8719ada165f96c1e` | 5 | 149,314 | `-4790310662820639479` | primary |

6 objects in 3 rendition groups. dkt 9 (1RC191003776) is the case that made
this table necessary: `6f489905…` is 6 pages and `20ca4fda…` is 2 pages of the same DOC — different
bytes, different rendition, each with its own ADS receipt.
