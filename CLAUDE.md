# civicatlasma

The public face of Civic Atlas MA: annual municipal election results for
Massachusetts towns and cities. This repository publishes the data
and hosts the source documents. It is not where work happens.

Work happens in `muni-harvest`. Sensitive material lives in `civicatlas-private`.

## What is here

    json/       one record per town-year, <Stem>.json 
    pdfs/       the source document for each town-year, <Stem>_d0.pdf, the most common document source
    xlsx/       spreadsheet exports
    images/     direct image exports (JPEG, PNG, etc.)
    markdown/   text OCR (or extraction) of the original file format
    raw_ocr/    OCR of documents with no text layer -- the only reading of a scan

A **stem** is a municipality with spaces removed followed by the four-digit
year: `Athol2023`. It names every file belonging to that town-year.

Documents carry a `_d<N>` suffix. A new number is minted only for genuinely
different bytes, so `_d3` means the third distinct document tried for that
town-year.

## This directory is generated

Everything here is built from the working corpus and published by a script.
**Never edit a file in this repository by hand.** A hand edit is silently
overwritten by the next publish, and worse, it makes the public copy disagree
with the corpus it claims to represent — with nothing to say so.

Corrections belong upstream, as rows in the adjudication ledger.

## Pages

The site is served from `main` at the repository root:
`https://jamesnicholsworley.github.io/civicatlasma/`

`pdfs/` is not only for readers. **The Claude API fetches source documents from
these URLs during parsing**, because some municipal domains block it from
fetching directly. Moving, renaming or removing a PDF breaks parsing for that
town-year, and the failure is silent. If a document must be renamed, the
`hosted_url` column in the working corpus changes in the same operation.

## What must never be published here

- **Full text of news articles.** This corpus cites journalism — headline, URL,
  date, a short snippet — it does not reproduce it. Several sources are paid
  subscriptions. Full article text belongs in `civicatlas-private`.
- **Newspaper issue PDFs.** Complete issues of commercial local papers.
- **Anything carrying residents' personal details.** One town's survey appendix
  carries names, street addresses, phone numbers and email addresses.

A publish step that would place a news-sourced markdown file in this repository
is a bug in the publish step, not a judgement call to make in the moment.

## A caution about accuracy

The corpus is a work in progress and its QA system is being rebuilt. Records
here have not all been verified to the standard the project intends. Prefer
saying what is known and what is not over presenting the whole set as settled.

## Where to look when you are not given a specific task

The site is not only for readers. These three pages are how an agent finds work
without being told what it is:

- **`coverage.html`** — what has been found and what is missing, town by town,
  with the sources already tried for each gap. Start here for harvest work; the
  attempts already recorded are what stops the same dead end being walked twice.
- **`mvp/`** — every town-year, with each figure traceable to the line of the
  document it came from. Start here for QA: the towns carrying anomalies are
  visible, and the largest ones are worth more than the smallest.
- **`audit.html`** — records that failed a check, and why.

Prefer the most populous town-years with anomalies. A wrong figure in Quincy is
read by more people than a wrong figure in Gosnold, and the work is the same.

`mvp/README.md` documents the method behind the map.

Two things to hold on to when reading these pages. A record shown here without a
flag has not necessarily been verified — it may only be unchecked, and the two
look identical. And the pages are built from a snapshot; if a number here
disagrees with `json/`, the JSON is the corpus and the page is stale.
