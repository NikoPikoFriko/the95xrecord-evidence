# VERIFY.md — check the bytes yourself

Every file in `sha256/` is named after its own SHA-256. Verification needs nothing from us:
no API key, no account, no trust.

---

## 1. One file, one command (Linux / macOS / WSL / Git-Bash)

```bash
H=8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e     # any hash from CATALOGUE_80.csv
curl -sL "https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main/sha256/$H.pdf" -o "$H.pdf"
sha256sum "$H.pdf"
```

The output must be:

```
8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e  <your download>
```

If the printed hash equals the filename, the bytes are exactly ours. If it does not, **the download is not our
file** — stop and report it; that is a real finding, not a rounding error.

## 2. All 80, one command (after clone)

```bash
git clone https://github.com/NikoPikoFriko/the95xrecord-evidence
cd the95xrecord-evidence
sha256sum -c MANIFEST.sha256
```

Expected: 80 lines, each ending in `OK`. Any `FAILED` line is a genuine integrity failure.

## 3. All 80 without cloning (curl loop)

```bash
BASE=https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main
tail -n +2 CATALOGUE_80.csv | while IFS=, read -r case docket grade name bytes sha key r2 ok; do
  sha=$(echo "$sha" | tr -d '"')
  curl -sL "$BASE/$key" -o /tmp/x.bin
  got=$(sha256sum /tmp/x.bin | cut -d' ' -f1)
  [ "$got" = "$sha" ] && echo "PASS $key" || echo "FAIL $key ($got)"
done
```

## 4. Windows PowerShell equivalent

```powershell
$H = '8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e'
Invoke-WebRequest "https://raw.githubusercontent.com/NikoPikoFriko/the95xrecord-evidence/main/sha256/$H.pdf" -OutFile "$H.pdf"
(Get-FileHash -Algorithm SHA256 "$H.pdf").Hash.ToLower()   # must equal $H
```

## 5. Verify a specific docket, not a hash

`CATALOGUE_80.csv` maps document name and docket → hash. Example: find every row for trial docket 423:

```bash
awk -F, 'NR==1 || $2==423' CATALOGUE_80.csv
```

or, to find which docket a hash belongs to:

```bash
grep 8719ada165f96c1eff2040c0312bd6c7adf19871f3702fdf865b5ace5d04b94e CATALOGUE_80.csv
```

Then verify that row's `github_path` as in §1.

---

## What a pass and a fail mean

| result | meaning |
| --- | --- |
| hash == filename | the served bytes are the bytes we published. This is the only claim the repository makes. |
| hash != filename | either the download was corrupted/intercepted, or the repository was tampered with. Either way: a reportable event, and the reason this repository is useful. |
| 404 | the object is not published under that hash — check the spelling in `CATALOGUE_80.csv`. |

## Scope of the claim (stated plainly)

This repository proves **integrity and immutability of the bytes we published**. It does **not** by itself
prove a document is what a court certified: that is what the clerk's certification page inside ten of the
`CERTIFIED`-graded documents does, and what the underlying case record does. Keep the two claims separate —
conflating them is exactly the kind of overclaiming this project exists to avoid.

*No credentials are needed or accepted for verification. Nothing here phones home.*
