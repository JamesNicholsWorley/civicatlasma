# Civic Atlas MA

Annual municipal election results for Massachusetts cities and towns, with the
source document behind every record.

Massachusetts holds hundreds of municipal elections a year and no one collects
the results. Each town clerk publishes them however they like — a PDF, a scan, a
page on the town website, sometimes only a local newspaper — and a year later the
link is often gone. This is an attempt to gather them in one place and keep the
evidence.

## What's here

    json/       one record per town-year
    markdown/   text extracted from the source document
    pdfs/       the source document itself, <Stem>_d0.pdf
    xlsx/       spreadsheet exports
    images/     image sources, where that is what the town published

Files are named by **stem** — the municipality without spaces, then the year:
`Athol2023.json`, `Athol2023_d0.pdf`.

## A record

```json
{
  "elections": [{
    "municipality": "Paxton",
    "date": "2024-05-13",
    "office_original": "SELECT BOARD - 3 YEARS",
    "num_winners": 1,
    "candidates": [
      { "name_original": "Jane A. Smith", "votes": 612 },
      { "name_original": "Blanks",        "votes": 141 }
    ]
  }]
}
```

`_original` fields are transcribed from the document exactly as printed,
including clerk typos and inconsistent office names. Corrections are recorded
separately rather than overwriting what the document says.

`Blanks`, `Others` and `Write-ins` are tally rows, not people.

## Scope and caveats

Annual municipal elections. Special elections and preliminaries are collected but
not published here. State, county and federal races are out of scope even when
they appear on the same ballot.

**This is a work in progress.** Coverage is incomplete, and the quality-assurance
system that checks these records against their source documents is being rebuilt.
Some records have been verified line by line against the document; others have
not. Where a figure could not be confirmed, the aim is to say so rather than to
present the whole set as settled.

If you find an error, an issue with the town, year and what the document actually
says is the most useful thing you can send.

## Reuse

The records are compiled from public records published by Massachusetts
municipalities. The source documents are included so that any figure can be
checked against what the town actually printed.
