# Redaction check specification and test corpus

How to decide whether a PDF's blackouts actually removed the information
underneath them — and the documents to test an implementation against.

- **[`redaction-check-spec.json`](redaction-check-spec.json)** — the rules.
  Eleven checks (A1–A11), the thresholds that decide what counts as a
  blackout, and the shape of a result.
- **[`generate.py`](generate.py)** — builds the test documents. Standard
  library only, deterministic output, no dependencies.
- **[`manifest.json`](manifest.json)** — the expected verdict for every
  document, including which checks **must not** fire on it.
- **[`files/`](files)** — the generated documents, committed so they can be
  read without running anything.

## Why this is published

A redaction checker is only worth something if you can see how it judges.
abyo software sells a PDF editor, and also publishes a free checker that
recommends it. "Does it go easy on its own output?" is the right question
to ask about such a tool, and the only useful answer is the criteria plus
the tests — including the tests that exist specifically to catch
over-reporting.

Anyone is free to implement these rules, disagree with them, or use the
corpus to find where an implementation is wrong. If you find a document
that these rules judge incorrectly, that is worth an issue.

## What is not here

The checker's own source code. The criteria and the corpus are what make
a verdict auditable; the user interface does not. Treat this repository as
the specification, not as the tool.

## What the rules are for

They answer one question: **for the places a document blacked out, is the
hidden information gone?**

They deliberately do not answer whether everything that *should* have been
blacked out was — that is the author's judgement and cannot be read out of
a file — and no OCR is performed, so text inside an image is not examined.

## Two things the thresholds exist to prevent

Most of the specification is about **not** crying wolf, because a checker
that flags correct work is worse than none:

- **Paint order decides concealment.** Text only counts as hidden when it
  was painted *before* the thing covering it. A white heading on a black
  band is covered by every area-based test and is perfectly visible; the
  order test excludes it structurally rather than by tuning numbers.
- **An opaque image is a blackout only when its own pixels are dark.** A
  scanned page is one big opaque image, usually with an OCR text layer
  under it. Without that condition, every scanned-and-OCR'd document
  reports as leaking.

The same reasoning governs A6: redacting an image properly rewrites its
pixels *and* draws a bar over them, so "a bar covers an image" on its own
is the signature of a correct redaction, not a defect.

## Running the corpus

```sh
python3 generate.py          # writes ./files
```

Each file carries exactly one failure mode. `manifest.json` says what each
should produce:

| Field | Meaning |
|---|---|
| `expect.verdict` | `fail` (readable information survives), `review` (a person must look), `pass` (nothing found) |
| `expect.checks` | Check ids that must be raised |
| `expect.mustNotFlag` | Check ids that would be a **false positive**; raising one is a failure |
| `expect.acceptVerdicts` | For cases whose outcome legitimately depends on whether an implementation can decode a particular image codec |

Numbering: `1`–`15` are the shared cases, `16+` are editor-only, `90+` are
locally generated stand-ins for cases whose real article has to be produced
by a specific application.

`12-acrobat-correct.pdf` and `13-abyo-editor-correct.pdf` cannot be
generated: the whole point of them is that a document redacted properly
**by those applications** is not flagged, so they have to come from
Acrobat and from the editor. Where one is absent the manifest marks it
`pending`, and a generated stand-in covers the false-positive test
meanwhile.

## Implementations

- The free browser checker at <https://abyo.net/pdf/redaction-check/> —
  runs entirely in the browser; the file is not uploaded.
- The pre-save check in abyo PDF Editor.

Both adjudicate by `specVersion` 1. The browser checker's test suite fails
if the numbers in `redaction-check-spec.json` stop matching the numbers in
its code, so the published criteria cannot quietly drift from the
implemented ones.

## Licence

MIT — see [LICENSE](LICENSE). Use the rules and the corpus for anything,
including in a competing product.
