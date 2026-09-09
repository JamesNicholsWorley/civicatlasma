# Pre-2021 town-years, from Annual Town Reports

These are separate from `json/` on purpose. They are weaker evidence and the
separation is the honest way to say so.

A record in `json/` came from a document the clerk published as the election
return. A record here was **cut out of a two-hundred-page annual town report by
a locator** that is sometimes wrong, and transcribed by the cheap model. Every
record carries `_provenance: "atr_section"` so nothing downstream has to infer
which corpus it came from.

Only records the publication gate rated `publish` are here: the arithmetic
closes, every figure was read, and every candidate row has a name. The gate is
calibrated against `json/` rather than invented — run over those 1,900 records
it passes 80%, so these clear the same height as the material beside them.

`_gate_reasons` records anything still unresolved about a published record. The
commonest is that no two contests agree on a ballot count, which blocks
cross-checking rather than correctness, and which 294 records in `json/` share.

Held back, and not here: sections whose arithmetic is impossible, and sections
the transcriber reports are not returns at all -- town meeting minutes,
warrants, salary schedules. Those are locator failures with a known fix and
they are counted separately, because re-cutting a report is cheap and
re-reading one is not.
