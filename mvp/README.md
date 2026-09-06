# The sample — 2024 & 2025

A single working page, built to be handed to somebody who has not been in the
project: academics, funders, a reporter. It is not a product and does not pretend
to be one. It exists to answer three questions in about ninety seconds —

1. **What is this?** Municipal elections that no state archive holds.
2. **How much of it is there?** 351 of 351 municipalities in 2025, 293 of the 296
   that held an election in 2024. 6,820 contests, 18,420 candidates.
3. **Why should I believe any of it?** Click a name.

```
research/interface/mvp/
  build_geo.py             MassGIS shapefile -> geo.js        (0.25 MB)
  extract_layout_text.py   each town-year PDF -> _text/       (columns preserved)
  office_schema.py         office_original -> a controlled vocabulary of 35 offices
  build_mvp.py             the corpus -> mvp-data.js, leads.csv
  index.html               the page
  apply_fixes.py           the only script here that writes to the corpus
  ocr_one.py               one town-year's scan, rendered first then OCR'd
  reparse_springfield.py   a re-total that refuses to write unless it closes
  _text/                   this directory's own pdfplumber extraction (cache)
```

Run, in order:

```
python research/interface/build_enrollment.py    # denominators (cached; no network unless the cache is empty)
python research/interface/mvp/build_geo.py
python research/interface/mvp/extract_layout_text.py
python research/interface/mvp/office_schema.py
python research/interface/mvp/build_mvp.py
python -m http.server 8781 --directory research/interface/mvp
```

Everything except `apply_fixes.py` is read-only over `publish/`, `data/` and
`C:/Users/Owner/QGIS`. `apply_fixes.py` repairs the corpus under the project's own
rules and records every change in `qa/reference/adjudications.csv`; it is idempotent,
and it is described in full below.

---

## Why the map is tilted

Massachusetts does not sit square to its own state plane. `build_geo.py` computes
the **minimum-area rectangle** enclosing the union of all 351 municipal polygons,
finds the bearing of its longest edge, and rotates the whole Commonwealth by the
negative of that bearing: **22.15°**.

| | aspect ratio |
|---|---|
| axis-aligned in EPSG:26986 | 1.63 |
| rotated onto the min-area rectangle | **1.90** |

That is the difference between a map that fits a page-width banner and one that
wastes two corners. The angle is computed, not hard-coded, and travels in the
bundle (`GEO.rotation`) so the figure can be reproduced and cited.

Coordinates stay in **EPSG:26986** rather than being reprojected to WGS84. The map
is a static SVG, so an equal-area state plane drawn with a linear transform is both
more correct and simpler than a Web Mercator computed in JavaScript. Rings are
simplified to 120 m and islets under 0.4 km² are dropped; shared borders are
simplified independently, so slivers exist below the drawn resolution.

---

## Making the converted source text readable

The earlier prototypes showed provenance as inventory columns, and the result was
the defect the owner spotted immediately: Provincetown's *"Audit trail"* said
**`verification: YES`**. That is a pipeline bookkeeping flag. Rendered as a
chain-of-custody row it reads as a claim about the votes, which is exactly what it
is not.

Three changes:

**1. The chain is a sequence of things that happened to the document.** How it was
found, where the town published it, whether we mirrored it, how it was turned into
text — each a plain sentence, each with its URL where one exists. Inventory columns
that gloss to nothing are dropped rather than displayed. What we *cannot* evidence
gets its own heading, **"What we cannot show you"**, so an absent mirror is stated
rather than implied.

**2. Every number is joined to the line it was read from.** For each candidate,
`build_mvp.py` locates the literal line in `publish/markdown/` and carries it.
Matching is on the letters alone, because the converters routinely lose the spaces
(`DavidCarrollAbramson-Elected`). A match is real evidence; **a miss is a finding**,
and is shown as one. 12,492 of 18,420 candidate rows (68%) currently match. Of the
rest, the great majority are not misses at all: **201 town-years converted to nothing
but `<!-- image -->`**, because the town published a scan with no text layer. Those
are marked `imageonly` — the numbers came out of a picture and no printed line exists
to quote. Only 54 town-years have a real text layer that the name is genuinely absent
from.

**3. The line is rendered, not dumped.** Markdown pipe-table structure is preserved
through the build (flattening it to one string is what made the text unreadable
before), so a line renders as cells. Spacing is reconstructed for display — digits
split from letters, lowercase from uppercase, `-Elected` set off — the candidate's
name is highlighted inside it, and **`exact text` toggles back to the unmodified
converted string**. The reconstruction is a display aid and is labelled as one.

Provincetown 2025 is the worked example: a two-column PDF whose columns interleaved
in conversion, so the line evidencing Chelsea Beth Crowe's 413 votes is
`John T. Golden — Elected | 405 Chelsea Beth Crowe — Elected | 413`. Which is
messy, and true, and better than `YES`.

---

## What the map is showing

**Turnout** — ballots cast over registered voters. The numerator is the town-year's
ballot count as established by the seat arithmetic below, not a sum of votes: a
voter who votes in nine contests is one ballot, and a vote-for-two contest takes two
marks from every ballot. Breaks are those of the printed map this is modelled on
(`<10 / 10–20 / 20–30 / 30–40 / >40`) so the two can be read side by side.

A town-year with no counts gets its own swatch — *"no counts published"* — and is
never shaded as the darkest band. Putting it at the bottom of a turnout ramp would
place a measurement where there is only a silence.

**Contested** — the share of a town's contests where more candidates cleared 5% of
the ballots than there were seats. The threshold matters: roughly a third of races
that look contested are a filed candidate against a write-in.

**Closest margin** — the narrowest gap in the town, as a share of ballots cast.

**Source** — where the result came from, and, for towns with nothing, the
distinction between *searched and not found* and *no election held*.

---

## Defects this page found in our own pipeline

All of them surfaced only because the data was drawn rather than tabulated. None
was found by the defect register.

### The denominator was being projected, and registration does not trend

`build_enrollment.py` used to extrapolate along the slope of the last two published
snapshots for any election falling outside the range. Registration **sawtooths**:
the biennial removal of inactive voters cuts the rolls between a November count and
the following February one. Projecting that slope forward nine months gave
Somerville's November 2025 election a denominator of **33,522 against a last-known
count of 51,622** — a 35% error, rendering as 62% municipal turnout, which looked
merely surprising rather than wrong.

Fixed by carrying the nearest published count forward unchanged (`basis: carried`,
610 town-years statewide). Carrying is wrong by whatever nine months of
registration adds; projecting was wrong by the purge, which is an order of
magnitude larger and has the wrong sign. Somerville now reads 40.3%, Lawrence
39.2%, Medford 33.4%.

### The town's own printed turnout was being read off a single precinct

`build_mvp.py` extracts the figures a document states about itself — registered
voters, ballots cast, turnout — as an independent check on ours. Taking the *first*
regex match picked up Precinct 1 in every return that reports precinct by precinct,
and manufactured disagreements that were not there. Now it takes the **largest**
match, and discards any printed ballot count materially smaller than what we have
already counted, on the grounds that it cannot be the town total. The check went
from 40 false disagreements to **12 real ones**.

---

### Every municipality now has a denominator, and East Bridgewater exists

`build_enrollment.py` keyed election dates on the `municipality` field alone. That
field is null in a number of published records, so thirteen towns — Leicester,
Groveland, Sutton and others — had **no registered-voter denominator at all**, and
`EastBridgewater2025` never resolved to a municipality, which is why the inventory
called 2025 complete at 351 while this page could only draw 350.

Both scripts now resolve a town on **letters alone**, falling back to the filename
stem, which is the town name with the spaces taken out. Denominators went from
1,756 town-years to 1,842 with none missing; the county-sum gate still passes on
all seven snapshots. Turnout now covers **644 of 644** town-years.

### The seat count on the record is sometimes arithmetically impossible

Somerville 2025 `COUNCILOR AT LARGE` carries `num_winners: 9` against eight
candidates, so the contest read as *uncontested with nine winners*. It is not: the
contest counted 83,288 votes against 20,822 ballots in the mayoral race, and
83,288 is exactly 4 × 20,822. Somerville elects four at-large councillors.

`ballots_and_seats()` implements qa/STANDARD.md **C2 and C4**. A contest takes one
mark per seat from every ballot, so one town-wide single-seat contest states the
ballot count outright. A recorded seat count is overridden only when it fails that
arithmetic and the alternative passes: `total / recorded_seats` misses the ballot
count by more than 5%, while `total / ballots` lands within 1% of a whole number
between 2 and 40 that is *smaller than the number of candidates*. Ward, precinct
and town-meeting contests are excluded entirely — a precinct contest draws only its
precinct, so dividing it by a town-wide ballot count is meaningless, and doing it
anyway was manufacturing a correction for every Town Meeting Member race in the
corpus.

**26 contests are corrected**, among them Somerville (9→4), Medford City Council
(13→7), Everett at-large (9→5), Pittsfield at-large (8→4), Lynn (5→4) and
Springfield (5→4). Waltham 2025, where 45,078 over 6 seats reproduces the ballot
count exactly, is left alone. Every corrected contest states both numbers on its
face and carries the `seatcount` flag. The seat count also fixes turnout, which had
been dividing by a wrong seat count: Medford 2025 was reading 50.5% and is 33.4%.

---

### The OCR had already run, and nothing was reading it

201 town-years converted to nothing but `<!-- image -->` and were being reported as
scans with no text. **171 of them already had their text on disk**, in
`data/raw_ocr/`, put there by `src/ocr_scanned_batch.py` — which renders each page
to a 250 dpi bitmap before handing it to Docling, because Docling will not
rasterise a scan on its own. Nothing downstream of that store was reading it.

`load_source()` now falls back to `data/raw_ocr/<stem>.txt` when the markdown
conversion is empty of text. Source text went from 443 town-years to **614**, and
names traced to a literal printed line from 12,492 to **15,729 of 18,420 (85%)**.
Thirty town-years remain genuinely un-OCR'd; the command for those is in
*Answering the open questions* below.

### `data/turnout.csv` holds the year in the ballots column, four times

Holden 2025 and Wilbraham 2023, 2024 and 2025 record `total_votes_cast` equal to
their own year. Wilbraham 2024 was therefore reporting **2,024 ballots cast**, and
2025 exactly 2,025. The arithmetic says 1,710 and 730, which reproduce the town's
own printed turnout of 15.18% and 6.01% to within a fifth of a point.

Those rows are now rejected on sight, and more generally `data/turnout.csv` no
longer feeds the numerator at all — it is a fallback for town-years with no counts,
and a cross-check. The contests are the numerator.

### Printed figures were being believed without bounds

The check that compares our count to the figures a document states about itself was
reading, variously: a per-precinct turnout row (`TURNOUT 3% 4% 3% 4% 2% ...`,
Wilmington 2024), a registration count out of a broken header (Chelsea 2025, 19,765
against 18,990 registered), and a figure that simply appeared later on the page than
the real one (Winchester 2025, where the return says `17.3% Voter Turnout` with the
number *before* the label, which the regex could not see).

Printed figures are now bounded before they are believed: a stated ballot count
above the registration count is discarded, a turnout outside 3–95% is discarded, and
percentages are read in both orders. Winchester now reproduces its own printed
figure exactly — 17.37% derived against 17.3% stated. The check dropped from a
high-severity contradiction to a **medium-severity irreconcilability**, because in
Winchester and Chelsea *we* were right and the parsed figure was wrong, and the flag
should not pretend to know which.

## What −1 means, and why it is not a defect

`qa/STANDARD.md` A3 reserves two negatives in the votes field:

- **−1** — elected, uncontested, no per-candidate count exists in the source.
- **−3** — a write-in winner whose count is not separable from the printed
  *All Others* total.

These are the standard working, not failing. The page renders them as `NO COUNT`
and `WRITE-IN` beside an *elected* mark, never as a number, and the provenance
drawer says so in words: *"No vote total is claimed, because none exists."* A
town-year in which every contest is a −1 gets a turnout of **n/a**, not 0.0%, and
is not flagged. Seven town-years are in that state.

Related: `Christopher A. Hopkins (Write-in)` is a person who won a seat, and was
being classified as a tally line by the same rule that catches a bare `Write-in`.
Names that survive stripping a write-in annotation and still read as a personal
name are now people.

---

## Grounding: the name, and then the number

Locating a candidate's name on a printed line proves the right row was read. It
does not prove the right number was taken off it, and in a two-column return those
come apart. So the check runs twice, and the page says which it got.

| | |
|---|---|
| names found on a line of the source | **17,495 of 18,421 — 95%** |
| vote totals found with the name | **16,295 of 17,361 — 94%** |

Both were well below that a day ago (88% and 70%), and closing the gap was almost
entirely a matter of reading stores that already existed.

### Four stores, not one

The build had been reading `publish/markdown/` and nothing else. There are four
readings of each document, and they fail in different places:

| store | what it is | how it fails |
|---|---|---|
| `data/pdftext/<sha>.txt` | the PDF's own text layer, keyed by the sha256 of the file | one value to a line, so name and number are never adjacent |
| `research/interface/mvp/_text/` | this directory's own `pdfplumber` extraction with `layout=True` | absent where the PDF is a scan |
| `publish/markdown/` | the converted document | often only the first page of a multi-page return |
| `data/raw_ocr/` | a reading of a scan, rendered first at 250 dpi | OCR noise |

They are readings of the same document, so a name or figure found in any of them is
evidence the document carries it. Pooling them, deduplicated, took names from 88% to
94% on its own.

### Geometry is what grounds a number

The single largest gain came from **re-extracting every town-year's PDF with
`pdfplumber(layout=True)`** (`extract_layout_text.py`, 443 of 566 PDFs; the rest are
scans). Every existing store flattens the page, and for an election return that is
the whole problem: Westfield 2025's write-in block converts to a bare list of names
with the counts somewhere else entirely, and 192 of its 280 figures were unprovable.
With the columns intact they are on the same line as their names. Figures went from
70% to 94%.

Two smaller rules do the rest:

- **Every occurrence is tried, not the first.** A name appears in several readings
  and several places within one. The page now takes the first occurrence whose own
  line — or whose column beneath it — carries the figure, and falls back to the
  first occurrence only when none does.
- **A column counts as a line.** Where a document is laid out one value to a line,
  the figure is searched for beneath the name, stopping at the next candidate so a
  number is never credited to the person after the one it belongs to.

In the election view a grounded figure carries a solid underline and an ungrounded
one a dotted underline; the drawer says which, and shows the second line where the
figure was found in a column rather than a row.

## A schema for `office_original`

`office_original` is one string carrying four different facts, and the 351
municipalities write it **3,210 different ways** in 2024-25 alone. The Select Board
family alone has 193 distinct spellings. `GROUP BY office_original` returns almost
as many groups as there are contests, so nothing can count how many towns elect a
Board of Health, or compare a Planning Board across towns.

The four facts, glued together:

```
SELECT BOARD (Vote for not more than TWO for THREE Years)
^^^^^^^^^^^^        ^^^                      ^^^^^
office              seats                    term

TOWN MEETING MEMBER - PRECINCT 6
^^^^^^^^^^^^^^^^^^^   ^^^^^^^^^^
office                scope
```

`build_mvp.py` already split out `term` and `seats`. `office_schema.py` adds the
other two — a canonical **code** and a **scope** — as a controlled vocabulary of 35
offices, matched on an exact normalised key so that a new spelling shows up in the
residue rather than being swallowed by a loose pattern.

| | |
|---|---|
| distinct `office_original` strings | 3,210 |
| contests | 6,821 |
| **matched to one of 35 codes** | **5,531 (81%)** |
| carrying a ward or precinct scope | 406 (6%) |

| code | contests | spellings |
|---|---:|---:|
| `town_meeting` | 712 | 145 |
| `select_board` | 571 | 193 |
| `planning_board` | 523 | 199 |
| `library` | 458 | 205 |
| `school_committee` | 457 | 205 |
| `assessors` | 420 | 146 |
| `health` | 393 | 124 |
| `moderator` | 317 | 91 |

Two decisions are worth stating because they are judgements, not derivations.
`Cemetery Commission` and `Cemetery Commissioner` are the same office; `School
Committee` and `Regional School Committee` are not, because they are different
bodies elected by different electorates. And where the answer is genuinely unclear
the string is left **unmatched and reported** rather than forced into the nearest
bucket — the residue is the honest part, and it is mostly what it should be: named
regional districts (`TRITON REGIONAL SCHOOL COMMITTEE`, `Hawlemont School
Committee`), and offices that exist in one town only (`TILTON FRUIT FARM
SUPERVISOR`, `Elector under Will of Oliver Smith`).

Outputs: `office_schema.csv` (the vocabulary and its coverage) and `office_map.csv`
(every raw string with its code and scope). The code and scope now travel on every
contest in the bundle, and the scope is printed beside the office in the election
view, which is the first time two contests called `Town Meeting Member` in the same
town have been distinguishable on the page.

### Recovering the district the office string dropped

Barnstable 2025 holds six contests all called `Member of the Town Council`, and its
return says **`Member of the Town Council Precinct 2`** in as many words. The
precinct was in the document and was dropped on the way into `office_original`.

`scope_from_source()` puts it back, by evidence rather than by order: for each
contest it finds the line the leading candidate appears on, walks back to the nearest
line naming both the office and a precinct, and takes that. A contest whose
candidates cannot be placed keeps no scope, and if the walk-back gives two contests
the same precinct neither is labelled -- a wrong precinct is worse than a missing
one. **138 district labels recovered**, and Barnstable comes out clean.

Greenfield does not, and the difference is instructive: its return is a matrix with
the precincts as *columns*, so the label is never attached to a contest at all. The
five `City Councilor` races stay ambiguous because the document really does not say
which is which.

### Which is how the next defect surfaced

**`ambiguousoffice`, 371 contests across 118 town-years.** A town-year holds the
same *raw* office string more than once at one seat each, so nothing in the record
tells the contests apart. Greenfield 2025 has **five contests all called `City
Councilor`** with no precinct on any of them; Barnstable has six `Member of the Town
Council`; Belmont 2024 has nine `Town Meeting Member`. The winners are right. Which
seat each of them won is not in the record.

This is deliberately separated from the 58 town-years where only the *display* name
collapses two contests — a three-year seat and a one-year unexpired one, say. There
the distinction still exists in `office_original` and losing it is a normalisation
choice made here, not a fault in the corpus.

## What the map shows about the whole Commonwealth

The panel beside the map carries the figure the printed map carries: ballots cast
against every registered voter in the municipalities that published a result.

| | ballots | registered | turnout |
|---|---:|---:|---:|
| 2024 | 439,325 | 2,499,189 | **17.6%** |
| 2025 | 938,514 | 5,025,826 | **18.7%** |

Both sides exclude towns with no published result, so the percentage is a turnout
and not a coverage figure in disguise, and the panel says how many towns are
excluded. For scale, the printed map this page is modelled on put 2023 at 18.0%.

The map also now draws **the Commonwealth's own outline**, taken as the union of all
351 municipal polygons. Towns that held no election are drawn without a stroke so
the map reads as *these are the towns that voted*; without the outline behind them,
an unlined town on the coast or the state border left a hole and the map looked
damaged rather than sparse.

## The QA pass

| flag | severity | town-years | |
|---|---|---:|---|
| `ambiguousoffice` | medium | 94 | two contests the record cannot tell apart |
| `nofigure` | low | 94 | the name is in the source, the number is not |
| `srcpartial` | medium | 66 | the converted text covers only part of the document |
| `shortfall` | medium | 19 | fewer marks than seats x ballots, where the document prints blanks elsewhere |
| `printed` | medium | 8 | cannot be reconciled with a figure the document states |
| `printed_scale` | low | 6 | the document states a figure of a different order entirely |
| `dupname` | low | 5 | a filed candidate and a write-in cast for the same person |
| `lowconf` | low | 5 | the extraction flagged itself below 0.75 |
| `preballot` | **high** | 3 | built from a document that cannot establish the outcome |
| `allzero` | **high** | 1 | zero votes with no sentinel to say why |
| `districttotal` | **high** | 1 | a town-wide contest carrying one district's total |
| `undercount` | **high** | 1 | most of a contest is missing |
| `docvotes` | medium | 1 | fewer ballots than `data/turnout.csv` recovered |
| `duprow` | medium | 1 | the same row twice |
| `noseats` | medium | 1 | seat count missing or not positive |
| `notgeneral` | low | 1 | a single special or board seat, not a town election |

**High severity is six town-years, down from twenty-nine.** `seatcount`,
`imageonly`, `nonint`, `stalestatus`, `turnout` and `noline` are gone from the
corpus. `ambiguousoffice` is new, and is a finding rather than a regression.

### Classes closed by distinguishing them from something else

Most of what looked like defects were two different things sharing a flag:

- **`shortfall` 47 -> 20.** Twenty-eight town-years never print a Blanks row
  anywhere in the document. Every multi-seat contest in them falls short by
  construction; that is a house style, and flagging it forty-seven times buried the
  nineteen where the same document does print blanks for other offices.
- **`undercount` 4 -> 1.** Ashburnham 2024's PLANNING BOARD holds every mark that
  was printed and no blanks; Milton 2024's SCHOOL COMMITTEE was a write-in contest
  whose winner took 251. Neither is missing. A genuinely part-parsed return is a
  multi-seat town-wide contest whose leader polled like a real candidate and whose
  total is still a third of what the seats require -- Springfield alone.
- **`printed` 19 -> 8, plus `printed_scale` 6.** A stated figure is only a
  disagreement when it measures the same thing. Sturbridge 2024's document yields a
  "turnout" of 76% against 333 ballots, which is a ballot question's YES share, not
  a dispute. Beyond a factor of two either way the figure is recorded as
  uninterpretable. Forty-four town-years now take their ballot count from a figure
  the document states about itself.
- **`stalestatus` 21 -> 0.** Nineteen cleared; the two that remained are
  sample-ballot town-years where the status is telling the truth.
- **`noseats` 3 -> 2.** Where `num_winners` is absent and the contest total is a
  clean multiple of the ballot count, the arithmetic supplies it (`seatderived`).

### The ballot count is the load-bearing number, and three things were wrong with it

Nearly every check here divides by the town-year's ballot count, so an error in it
propagates into turnout, seat counts and every completeness test at once.

- **A ward series only sums to the ballot count if the wards are disjoint.** Newton
  elects its ward councillors *citywide* -- every voter votes in all eight -- so
  adding them gave **185,136 ballots in a city with 59,179 registered voters**. The
  registration count is a ceiling no ballot count can pass, and it is now enforced.
- **A district series can put its tag last.** `District Councilor - A` never matched
  the adjacent-pair pattern, so Lawrence's six district contests were invisible and
  its ballot count came from `total / seats` on a contest whose seat count was
  itself what needed checking.
- **A stated total has to be applied before the seat arithmetic, not after.**
  Lawrence's at-large contests hold 34,137 marks: four times 8,534 if you believe
  the district sum, exactly three times 11,379 if you believe the grand total the
  return prints two columns over. Lawrence elects three at large.

## Applying the fixes

`apply_fixes.py` is the only script in this directory that writes. It repairs what
the audit found, under the project's own rules rather than new ones, and records
every change in `qa/reference/adjudications.csv` with its rule, its reason and its
evidence.

```
python research/interface/mvp/apply_fixes.py            # dry run, prints the plan
python research/interface/mvp/apply_fixes.py --apply    # writes
```

It is idempotent: a second run is a no-op, and the ledger will not double up. Both
`data/json` and `publish/json` are written, because their agreement is itself a
checked invariant — after the run the edited files are byte-identical across the
two stores, and all thirteen blocking invariants in `src/check_invariants.py` pass.

**104 ledger rows: 102 changes applied, plus two notes.**

| what | rule | n |
|---|---|---:|
| `num_winners` corrected where the ballot arithmetic contradicts it | C2 | 31 |
| `0`, `<UNKNOWN>` and one-vote-each re-encoded to `-1` | A5 | 21 |
| stale inventory statuses cleared | INV | 20 |
| `source_kind` recorded for the town-years that had none | INV | 14 |
| `data/turnout.csv` rows holding the year in the ballots column, retracted | E1 | 4 |
| registration counts moved out of the ballots column | E1 | 4 |
| ballot counts harvested out of prose notes | E1 | 3 |
| vote totals recovered from a second report of the same election | A5 | 3 |
| Leverett 2025 restated from the town's own minutes | G | 1 |
| Leverett 2025's citation repointed at the right document | A1 | 1 |
| a reason corrected, and a gap in the standard recorded | A5 | 2 |

### Why this exists alongside `src/adjudicate_seat_divergence.py`

That script asks which of two stores is right about `num_winners` when they
disagree, using the same equality — `sum(votes) == ballots × num_winners` — and the
same district-partitioned ballot reference. **It has finished its work: it now
reports zero disagreements.** The errors that remain are the ones where *both
stores agree on a wrong value*, which by construction it never looks at. Somerville
2025 `COUNCILOR AT LARGE` read 9 in `data/json` and 9 in `publish/json`, and the
town elects four.

After the run, `seatcount` is **gone from the corpus**: nought town-years, down
from twenty.

### What was deliberately not fixed

**Ipswich 2024 School Committee** — three candidates, two seats, no counts.
A5's first condition is uncontested shape, and re-encoding this would elect two of
the three at random. It stays exactly as it is, and stays flagged. It is now the
only `allzero` in the corpus.

**Mashpee 2024 Housing Authority** — Jill Allen has 1,276 votes and B. Lynne Barbee
has `<UNKNOWN>`. Barbee lost; `−1` would mark her elected, and the standard has no
code for "a loser whose count is unknown". Left, and flagged.

**Blandford 2024 Fence Viewer** — the row reads `No candidate`. An early version of
the re-encoder would have elected it, because its tally pattern was a near-copy of
`build_mvp`'s rather than the same one. The patterns are now identical, and the
absence stays an absence.

## The four that were fixed from their sources

**Mashpee 2024 — the inventory note was stale, and the counts were findable.**
Its row said `wrongyear: document dated 2025-05-10, refile` with `status: missing`,
while the record it guarded was dated 2024-05-11 and its text was a Cape Cod Times
report of the 11 May 2024 election. The note described a document that had already
been replaced. The Cape Cod Times story named three winners without printing their
totals, which is where the `<UNKNOWN>` values came from; the **Cape News** report of
the same election prints all of them. It was used because it checks out, not because
it is a newspaper: six of its races -- Planning Board's three, the four School
Committee candidates, the associate member and the Water Commissioner -- reproduce
the corpus's figures exactly, to the vote, and the Select Board race it supplies
closes on its own arithmetic (1,968 + 1,912 + 30 write-ins + 1,180 blanks = 5,090 =
2 seats x 2,545 ballots). Six races corrected; `nonint` is now gone from the corpus.

**Gosnold 2024 — elected on the floor of Town Meeting.** Every candidate carried
exactly one vote, giving a town-year of 1 ballot against 107 registered voters. The
record is the Annual Town Meeting minutes inside the town's Annual Town Report:

> Article 1: To choose a moderator to preside at said meeting.
>   **Leo Roy elected unanimously.**
> Article 3: To elect the following officers ...
>   a. One Selectboard member, Board of Health member for 3 years.
>   **Win Sanford elected unanimously.**

`elected unanimously` is the whole tally. Re-encoded to `-1` under A5, whose two
conditions both hold. Gosnold is, with Leverett, one of two municipalities in the
corpus that hold no municipal ballot at all. The signature -- every candidate on one
vote, nothing contested -- was checked across 2024-25 before it was used, and it
matches one town-year, so this is a case and not a rule.

**Southbridge 2024 — not a town election.** Its single contest is `Third Member of
the Southbridge Retirement Board`, 184 votes. Divided by the town's 14,593
registered voters that produced a turnout of 1.3% for an election that was never
held. A town-year that is one special or board seat now gets no turnout figure at
all.

**Ipswich 2024 — a gap in the standard, not in the data.** Three candidates, two
seats, no counts. The winners *are* known: the town-year's own cited source reports
Kate Eliot on 29% and Haley Rist on 24%, and Karen LaFarge losing. No absolute count
is published for any of them, and the town's own document-centre link for the 2024
annual town election serves the September 2024 state primary instead. A3 gives `-1`
for "elected, no count published" and C3 forbids mixing `-1` with numerics; neither
covers a **contested** race whose winners are known and whose counts are not. So the
record holds three zeros, which asserts a tally of nothing, and A5 rightly forbids
re-encoding because it would elect two of the three at random. Logged to the ledger
as a recommendation for a new sentinel rather than patched over.

## Leverett, New Braintree, and what the minutes actually say

The premise was that the town meeting minutes would carry a count of voters
present. For these towns they do not — but chasing them turned up worse.

**Leverett elects its officers from the floor of Town Meeting**, with the Town Clerk
casting a single ballot. There is no ballot count to recover, and the minutes of
May 3 2025 record only *"a quorum of voters being deemed present"*. Leverett is the
last municipality in the Commonwealth that does this.

What the minutes do settle is who took office, and on that they contradicted our
record, which had been built from a Daily Hampshire Gazette report:

| office | our record | the town's minutes |
|---|---|---|
| Board of Health | Marsha **Johnson** | Marcia **Jackson** |
| Finance Committee | **Bethany Seeger** | **Steve Weiss** (two-year term) |
| School Committee | Elizabeth **Thompson** | Liz **Johnson** (one-year term) |
| Constable | *absent* | Tom Masterton |

Three different people and a missing office. The record is now the minutes.

Worse, the town-year's `native_url` pointed at
`leverett.ma.us/n/12187/Leverett-Election-Results`, whose only attachment is
`FINAL PRES- 2024-11-5.xlsx` — **the November 2024 presidential results**. So the
record was derived from a newspaper and cited a page carrying a different election
altogether. The citation now points at the minutes, the old URL is recorded in
`known_bad_url`, and the mirrored text is the minutes rather than the newspaper —
otherwise the provenance drawer would go on quoting the very names the minutes
correct. All thirteen names now trace to a line of the town's own document.

**New Braintree's turnout was there all along, in prose.** Its inventory row records,
in words: *"TURNOUT: 61 ballots cast of 843 registered (7.23%), verbatim from the
town homepage post-election notice archived 2025-05-20."* The figure had been found,
written into the `verification` free-text field, and read by nothing — so the map
showed New Braintree as a town that published no counts. It now reads 61 ballots and
7.2%, and the figure lives in `data/turnout.csv`, which is the store that exists for
it. A sweep of every inventory note found six such trapped ballot counts; the other
one in scope, Ipswich 2024's 4,507, is now the turnout numerator there too, in place
of the 4,252 the busiest contest gave.

Its **results**, though, are another matter. New Braintree's election page carries a
single document, the November 2023 state special, and nothing for 2024 or 2025. Both
town-years in the corpus are built from the town's **sample ballot**.

That turned out to be a class, not a case.

### `preballot`: three records built from a document that cannot establish an outcome

`qa/STANDARD.md` A5 is explicit: *"A sample ballot, a ballot face, an election
warrant, or a pre-election candidate list only shows who was on the ballot, never
who took office."* Three town-years are built from exactly such a document, each
saying so in its own inventory note: **New Braintree 2024 and 2025, and Richmond
2024**, all from the town's sample ballot.

Nothing is corrected here — there is nothing to correct it *to*. The flag is the
finding: these records name winners on the authority of a document that, by the
project's own standard, cannot name them. `Holbrook2021` is the ledger's standing
proof this is not academic.

**This number was 22 an hour before it was 3, and the difference is a mistake I
made.** The first version keyed on `status: discarded_not_results`, on the
assumption that it meant "the document is not results". On most rows it does not:
Boxford 2024 carries it beside a note reading *"landed from the Chrome ingest
(official, #pages 59-60); 8 races"* — the status belongs to a document that was
rejected before the real return arrived, and never got cleared. Narrowed to rows
whose own note names a sample ballot, the class is three. The nineteen stale
statuses were cleared as part of the repair, and `stalestatus` is now empty: the
two that had remained were sample-ballot town-years where the status is telling the
truth, and `preballot` says it better.

A second heuristic was withdrawn entirely. `looks_like_a_ballot()` read the source
text for ballot language (*"MARK A CROSS (X)"*, *"Caucus Nominee"*) and, finding it
alongside very few numbers, concluded the document was a ballot. It fired on
Aquinnah, Holland and Wales 2025, all three of which are real returns with real
counts. What it was actually measuring was *how badly the OCR had degraded* —
Wales 2025's converted text reads `= pny lia MATCHETTIN` — which `nofigure` and
`srcpartial` already report without accusing the document of being something it is
not. Blandford 2024, which prompted the idea, turns out to be a real return whose
Fence Viewer genuinely had no candidate.

## Answering the open questions

**Getting a text layer out of the scans.** `src/ocr_scanned_batch.py` renders each
page to a 250 dpi bitmap before Docling, because Docling will not rasterise a scan
embedded as an image object. The thirty town-years that had never been through it
have now been: **thirty of thirty produced text and none came back blank**, and with
the 171 that already had OCR sitting unread, `imageonly` is gone from the corpus and
every one of the 644 town-years has source text.

**Source kind.** Unrecorded is **zero**, from 76. Most of it went by reading the
host rather than trusting the field: a host containing the municipality's name is
the town's own site, a CivicPlus `/DocumentCenter/View/` or `/sites/g/files/vyhlif`
path is a municipal CMS whatever the domain says (which is how Williamsburg's
`burgy.org` and Auburn's town-golf-course domain resolve), an archived URL is
unwrapped to the publisher inside it, and a document parked on Google storage that
the LEO sweep found is the town's. The page names the publisher -- *City of
Springfield*, *WATD 95.9 FM*, *Daily Hampshire Gazette* -- rather than the category.

The last fourteen were decided one at a time from their own inventory rows. Twelve
are the municipality; two are news. Hopkinton 2024-25 got a new value: `ehop.org` is
a Hopkinton civic non-profit hosting the clerk's own results PDF, neither the town
nor a newspaper, so the vocabulary gained a third real category, **`thirdparty`**.

**What is still open, and why.**

- **Springfield 2025** (`undercount`). Its document is a 33-precinct breakdown with
  no citywide total anywhere in it, so re-fetching cannot help -- the number is not
  in the document. `reparse_springfield.py` sums the blocks and **refuses to write**:
  the ballots it recovers from the OCR come to 17,032 against the 11,699 the ward
  contests independently give, it reads 30 of 33 blocks, rejects 109 candidate
  lines, and returns an at-large total lower than the partial parse it would
  replace. The gate compares the two ballot counts and will not write above 3%
  drift. The PDF is a scan and needs a cleaner rendering.
- **Lawrence 2025** (`districttotal`). Its mayoral contest carries 1,834, which is
  District A's total to the vote, against the 11,379 grand total the same return
  prints. Summing the six district TOTAL rows recovers the ballot count (applied,
  and it corrected the city's at-large seats from four to three) and Depena's
  citywide total of 6,106 -- but the sixth district's cell for his opponent is empty
  in every conversion held, so the contest cannot be completed without re-reading
  the document.
- **New Braintree 2024-25 and Richmond 2024** (`preballot`). Built from sample
  ballots. There is nothing to correct them *to*.
- **Ipswich 2024** (`allzero`). A gap in the standard rather than the data; see
  above.
- **`srcpartial`, 74 town-years.** Documents of which only a fragment was converted.
  Thirty-eight of them have been queued through the renderer-first OCR; the rest
  have no PDF to work from.

## Provenance: what is actually missing

Every one of the 644 town-years has a chain of custody. What varies is how much of
it we can evidence:

| | town-years |
|---|---:|
| full chain — found, published, mirrored, converted | 365 |
| three steps | 217 |
| fewer | 62 |
| conversion method not recorded | 201 |
| no mirrored copy kept | 132 |
| **inventory marks the document "not, in fact, results"** | **21** |
| original publication URL not recorded | 7 |

The last two are worth an owner's eye. Seven town-years have results and no record
of where they came from. Twenty-one carry `status: discarded_not_results` in the
inventory and yet produced parsed contests — either the status is stale or the
results came from somewhere the inventory does not name.

The statewide register covering 2021–2026, with sampled verification against source
documents, is `../audit.html`.
