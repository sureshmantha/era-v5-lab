# Medicine sheet: cadj-commits (sureshmantha)

**Pack:** `cadj-commits` · **Source:** `bigcode/commitpackft` · **Status:** evidence-backed draft; human adjudication and the official gold-set harm test remain required.

## Integrity of diagnosis — corrected 27 October 2026

The source Parquet contains **500 rows with 500 unique document IDs and 500 unique full document texts**. Every `doc_id` matches the first 16 hexadecimal characters of `SHA-256(text)`. The seven `lang_label` values match the corresponding original metadata language and file extension for every record.

**Do not apply H1 or D1.** A previously issued analysis had an incorrect dictionary-index decode, which fabricated 437 duplicates and 428 language mismatches. This correction supersedes it. Neither H1 nor D1 appears in the newly generated `labels.csv`.

The scan found **45 email-bearing documents**; **40** have identifiable author/signer/reviewer context and are recorded as **F1 candidates**, not blanket confirmed private data. Five email-bearing documents relate to a company mailbox or a test fixture and are held for human judgment. The evidence CSV intentionally contains **only type and character position**, never email addresses or credentials.

## Medicine 1 — F1: potential personal email addresses in code/commit records

1. **Disease and count:** **40 candidate records** with named author, signer, reviewer or copyright contact information. Three examples: row 9 (`e0c1e30626b9b1ff`): email at offset 518; row 10 (`6454d40260da86f0`): email at offset 480; row 12 (`49b151d5092e2af4`): email at offset 742. The offsets are zero-based Unicode-character positions in the original `text` field. Public source availability does not by itself establish permission to train on personal contact information.
2. **Medicine (rule):** Inspect each code record, identify email-address spans, and **confirm that the mailbox belongs to a private person** rather than a corporate contact or a clearly fictional fixture. Only redact confirmed private-person addresses. Preserve source-code punctuation and quotation syntax. Do not change the original pack in this assignment; provide the cleaning rule for approval.

   ```python
   import re
   EMAIL = re.compile(r"(?<![\w.+-])[\w.+-]+@[\w.-]+\.[A-Za-z]{2,}(?![\w.-])")

   def candidate_offsets(text: str) -> list[tuple[int, int]]:
       """Positions only. Do not emit the raw matched values to public logs."""
       return [(m.start(), m.end()) for m in EMAIL.finditer(text)]

   def redact_approved_spans(text: str, approved_spans: list[tuple[int, int]]) -> str:
       """Only use spans explicitly approved by privacy review."""
       for start, end in sorted(approved_spans, reverse=True):
           if not (0 <= start < end <= len(text)):
               raise ValueError("Invalid span")
           text = text[:start] + "redacted@example.invalid" + text[end:]
       return text
   ```

3. **Action:** `fix` confirmed personal addresses using a syntactically safe placeholder; escalate/drop the record if the change breaks code semantics. Do **not** automatically remove an organization’s designated package-maintainer address, anonymous placeholders, or synthetic seeded test values.
4. **Gold-set harm test:** **NOT EXECUTED — official code-language gold set not supplied.** Gold `N`: unknown; changed: unmeasured; dropped: unmeasured; affected fraction: unmeasured; **result: PENDING**. For the final approved *automatic* rule, compare original versus transformed text for every official gold record; count every changed or dropped record once and reject when `affected/N > 0.01`. A nonautomatic human approval step is not a substitute for testing a proposed scaled-up cleaning rule.
5. **Side effects:** Package metadata and copyright attribution may be meaningful; replacing email literals can modify intended test fixtures; publicly named reviewers can have official rather than personal addresses; regexes may flag syntactically incomplete email fragments. False positives must be tested with privacy-preserving evidence.

## Additional lower-frequency conditions (medicine optional because frequency ≤5)

### A1 — U+FEFF BYTE ORDER MARK (1 document)

Row 93 (`eefa9818729074c3`), JavaScript, has **two U+FEFF** code points at offsets **116** and **761**. Action: investigate if BOM is only an encoding signature at the start of an embedded old/new file; if so, strip only redundant BOM according to the code-corpus policy. Verify UTF-8 decoding, preserve actual source semantics and run the gold harm test before deploying any automated transformation.

### A2 — U+000D CARRIAGE RETURN (5 documents)

The verified records are:

| Pack row | `doc_id` | U+000D count | First zero-based offset |
|---:|---|---:|---:|
| 24 | `f9b766f8d70fd170` | 14 | 170 |
| 197 | `aa905c0bfba1f969` | 99 | 139 |
| 223 | `9de24aed2480aa18` | 56 | 267 |
| 272 | `4a77ea59aa73b52a` | 66 | 193 |
| 282 | `4c25f9f3425b8be1` | 76 | 150 |

These are predominantly CRLF line-ending contexts, **not** random corruption. A normalizer may convert CRLF to LF **only after** confirming the data policy and verifying it does not alter meaningful string literals or expected code bytes. The `labels.csv` `hex` column lists U+000D counts and first offsets. This group does not exceed the taxonomy's >5 occurrence threshold.

## Other diseases checked, not established from this pack scan

- **H1 exact duplicate:** 0; IDs and texts are all unique.
- **D1 wrong language:** 0; document labels, `meta.lang` and file paths agree.
- **E3 secrets:** no independently verified active credential. The original draft's row 53 `johnpassword` appears in a fictional Django test account and is **not** evidence of a leaked real secret.
- **F1 name-only cases:** rows 43 and 74 contain credited contributor names without verified private contact information; earlier F1 labels were not substantiated.
- **B2/B4/E4/C2:** old/new commit snapshots repeat legitimate code and markup. Do not strip tags, braces, licences or repeated lines without demonstrating actual disease in context.

## Limits and audit notes

A programmatic pass inspected the complete `text` value of all 500 source records and confirmed the structural and character findings above. **That is not equivalent to the instructor's full personal reading of all 500 documents.** For the first 151 rows, pre-existing `read=full` entries were inherited from uploaded `labels-2.csv` unless conflicting evidence required reopening them; their human-review status cannot be independently certified. The remaining rows are labeled `partial` pending human reading. The official gold data are not in the uploaded repository ZIP; **no medicine passes the harm test until it is actually run**.

The instructor's `view_pack.py` tool needs `pyarrow`, unavailable in the current execution environment. We independently decoded the exact Parquet dictionary/RLE layout and verified each `doc_id` against the source SHA-256. Before final submission, run the official viewer and verify the remaining row decisions locally.
