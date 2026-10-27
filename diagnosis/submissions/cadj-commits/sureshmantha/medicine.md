# Medicine sheet: cadj-commits (sureshmantha)

**Status: PROPOSED — gold-set harm tests pending; not certified for final submission.**

Pack: `cadj-commits` · Source: `bigcode/commitpackft` · Physical records: **500** · Distinct text/document IDs: **63**. This file applies the code taxonomy from `diagnosis/DISEASES.md`. Candidate labels come from a complete programmatic scan of previously inspected data whose Parquet SHA-256 is `ce7aba1b82f49c7ea8b96f76f4b021b5b7619240e93c63f3cab1ed4e616fd7be`. The instructor requires full personal reading of each record; the accompanying CSV therefore retains `read=partial` until the author performs that review.

Several labels overlap on the same record. In particular, D1 mismatches are associated with records that are also H1 duplicates. For a repeated ID, pack row position distinguishes the first occurrence from later duplicates. A `doc_id` is **not unique per CSV row**; TSAI should clarify its expected reconciliation before final submission.

## Medicine 1 — H1: Exact duplicate

1. **Disease and count:** H1, **437 later duplicate occurrences** after 63 first occurrences. Three position-specific examples: row 64 (`8bcfa4035deb24d4`), row 65 (`6e3f1042350208e8`), row 73 (`6e3f1042350208e8`). Evidence: the document text and 16-hex `doc_id` are identical to a preceding record (row 64 repeats row 2; rows 65 and 73 repeat row 1). **There are only two distinct document IDs within these 437 duplicate occurrences**, so three different IDs cannot honestly be supplied.
2. **Medicine:** Generate an exact digest from the *raw* UTF-8 text and retain the first occurrence in an approved, deterministic traversal. Use the original raw bytes/text, not case-folded, whitespace-normalized or privacy-redacted text. Treat the pack's original `doc_id` as supporting evidence only, and verify content before deduplication.

```python
import hashlib

def exact_hash(text: str) -> str:
    return hashlib.sha256(text.encode("utf-8")).hexdigest()

def dedup_keep_first(records):
    seen = set()
    for record in records:
        digest = exact_hash(record["text"])
        if digest in seen:
            continue  # action: DROP this repeat after approval
        seen.add(digest)
        yield record
```

3. **Action:** `drop` later occurrences. Retain the first occurrence *only if* the lab agrees these are unintended duplicates; otherwise annotate duplicates for sampling-weight review. Store the original lineage separately.
4. **Harm test (mandatory, not performed):** The repository ZIP does not contain the official Python/code gold set. **Gold N = unavailable; changed = not measured; dropped = not measured; result = PENDING**. Run the exact function on the approved gold set; fail if `(changed + dropped) / N > 0.01`, counting each affected gold document only once. Do not claim 0%.
5. **Side effects:** Removing intentionally up-weighted repeated examples alters the training mixture; deduplication across mixed-language partitions could alter category counts; ordering determines the retained record. Recheck code provenance.

## Medicine 2 — D1: Incorrect language label

1. **Disease and count:** D1, **428 row occurrences**. Examples: row 73 (`6e3f1042350208e8`), row 145 (`6e3f1042350208e8`), row 217 (`6e3f1042350208e8`); observed labels JavaScript, TypeScript, Java, while both the `.py` embedded path and `meta.lang=Python` identify Python. All mismatched instances happen to refer to the **same repeated document ID**. Do not invent three distinct IDs.
2. **Medicine:** Verify that a recognized file extension and independent metadata agree; change the record's language label *only when both indicate Python*, and only for retained records. Do not edit the Python source code itself.

```python
from pathlib import PurePosixPath

def propose_python_label(record):
    path = PurePosixPath(record["embedded_file_path"])
    if path.suffix.lower() == ".py" and record.get("meta_lang") == "Python":
        if record["lang_label"] != "Python":
            return "Python"  # action: FIX only label metadata
    return None  # keep original; manual review if uncertain
```

3. **Action:** `fix` the label of any retained misclassified record. When the record is also H1, apply approved exact-deduplication policy first and avoid needless mutation of documents that will be dropped.
4. **Harm test (mandatory, not performed):** Official code-language gold set not supplied. **Gold N = unavailable; changed = not measured; dropped = not measured; result = PENDING.** Measure records whose labels change, divided by gold N; require ≤1% affected.
5. **Side effects:** Extensions may not reliably identify language; a multi-file or mixed-language commit may have one `.py` path but contain another language; `meta_lang` may be wrong too. Require both fields to agree and preserve a review audit.

## Medicine 3 — F1: Personal email addresses in code/commit metadata

1. **Disease and count:** **18 high-confidence personal-email candidate records** are proposed for F1; the original regex identified 21 email-containing records, but three were withheld from F1 pending contextual adjudication. Example records: row 9 (`e0c1e30626b9b1ff`), row 10 (`6454d40260da86f0`), row 12 (`49b151d5092e2af4`). Each CSV evidence field records only `email at offset N`—**never the email value**. Check author/maintainer identity carefully: public corporate addresses and synthetic test fixtures must not automatically be treated as personal data.
2. **Medicine:** Identify private-person email strings in source/commit text using a parser or regex candidate finder, perform *contextual confirmation*, and replace confirmed addresses with a privacy-safe valid placeholder where preserving code makes sense. Because addresses may occur in `setup.py` strings, use string-safe replacement rather than deleting syntax.

```python
import re
EMAIL = re.compile(r"(?<![\w.+-])[\w.+-]+@[\w.-]+\.[A-Za-z]{2,}(?![\w.-])")

def email_candidates(text):
    # Return positions only for review; never print or export the raw value.
    return [m.start() for m in EMAIL.finditer(text)]

def redact_confirmed_email(text, confirmed_spans):
    # confirmed_spans: list of (start, end) approved by a human reviewer
    for start, end in sorted(confirmed_spans, reverse=True):
        text = text[:start] + "redacted@example.invalid" + text[end:]
    return text
```

3. **Action:** `fix` after confirmation; if code semantics cannot survive the edit, recommend `drop` for separate review. **Do not redact all emails blindly**, because author/maintainer contacts and synthetic fixtures may be legitimate code examples.
4. **Harm test (mandatory, not performed):** Official gold set missing. **Gold N = unavailable; changed = not measured; dropped = not measured; result = PENDING**. Evaluate only the final approved detection+confirmation policy against the official gold sample; fail above 1% affected.
5. **Side effects:** False positives include corporate contacts and test fixture values; masking can change functional metadata and reduce code accuracy. Preserve originals in restricted-access storage only, never public PRs.

## A2: Control character — one occurrence (supplemental; not above five)

One record, row 24 (`f9b766f8d70fd170`), contains **14 U+000D CARRIAGE RETURN** characters, starting at character offset **170**. The `hex` entry uses `170 U+000D CARRIAGE RETURN (14x)`. Verify CRLF context in the original viewer. Conditional remediation: convert CRLF to LF only if approved and proven not to change meaning, literal values or code behavior. The official gold set is absent; do not assert harm safety.

## Gold-set acceptance register — unresolved

| Medicine | Gold count | Changed | Dropped | Harm rate | Result |
|---|---:|---:|---:|---:|---|
| H1 | not provided | not measured | not measured | not measured | **PENDING** |
| D1 | not provided | not measured | not measured | not measured | **PENDING** |
| F1 | not provided | not measured | not measured | not measured | **PENDING** |

A gold-set medicine **fails** when more than **1%** of gold documents are modified or dropped. No full-pack cleaning or public release of source data has been executed. Codes B2/B4 are not automatically assigned to legitimate HTML or template strings inside program source, and E3 is not assigned to synthetic test strings without evidence of a real secret.
