# Cummington, Massachusetts -- Annual Town Election, Monday, May 11, 2026

Source: the Town of Cummington's own Town Clerk record, published on the town
website at https://cummington-ma.gov/TownMeet.php?ATM , where it is the document
titled "2026-05-11 Annual Town Election Results" -- document id 373 in the site's
own Town Meeting document index at https://cummington-ma.gov/TownMeet.php?TM

## A note on how this page behaves, because it matters for re-checking

This row was previously recorded as "not yet published", on the reasoning that
Cummington prints its returns in the annual town report and that the FY2026 report
had not appeared. That was wrong, and the reason it was wrong is worth writing
down.

Cummington publishes the return as a database-backed page, not as a PDF. A direct
fetch of TownMeet.php?ATM returns an 18,212-byte response that stops dead at the
heading "Town Meeting Minutes" and contains none of the results. The same
truncation occurs on every query-string variant tried, on a multipart POST of the
site's own document-picker form with all six of its fields, and on its printer
view. The page also loads jQuery over plain HTTP from ajax.googleapis.com, which
now answers 503, so the document does not render in an automated browser session
either.

The consequence for the sweep record is specific: no PDF sweep could have found
this, because there is no PDF; and no fetch of the page could have found it
either. The document is served normally to an ordinary browser. It is listed by
name in the site's own document index, and that index IS fetchable -- it lists
"2026-05-11 Annual Town Election Results" alongside the 2022, 2023, 2024 and 2025
equivalents. That index is the reliable place to check this town in future.

## The record

The clerk's text opens:

"The polls were open; the ballot box declared empty and working properly. Voting
began at 1 p.m. and ended at 8 p.m., after 60 voters voted, resulting in the
following elected officials"

and closes:

"Such are the results of this Annual Town Election. Attest: Joshua Emerson,
Assistant Town Clerk, May 15, 2026"

The clerk adds a note on write-ins:

"Being a town election and not a federal election, all registered names that were
written-in have been listed, with counts below 10."

## Turnout

60 ballots cast. The record states the number who voted and does not state the
number of registered voters, so no turnout percentage is computed here.

The figure of 60 does not rest on that sentence alone; the table proves it. See
the arithmetic check below.

## Results

| Office | Term | Candidate | Votes | Note |
|---|---|---|---|---|
| Selectman | three years | June Lynds | 54 | elected |
| Selectman | three years | Blanks | 6 | |
| Selectman | one year | Kyle Citro | 53 | elected |
| Selectman | one year | Ryan Strong | 1 | write-in |
| Selectman | one year | Blanks | 6 | |
| Assessor | three years | Elliot Ring | 53 | elected |
| Assessor | three years | Blanks | 7 | |
| Moderator | one year | Joshua Wachtel | 52 | elected |
| Moderator | one year | Mark Demaranville | 1 | write-in |
| Moderator | one year | Blanks | 7 | |
| Town Clerk | three years | Brenda Emerson-Camp | 59 | elected |
| Town Clerk | three years | Blanks | 1 | |
| Finance Committee | three years | Murray Soloman | 53 | elected |
| Finance Committee | three years | Eric Pohlman | 49 | elected |
| Finance Committee | three years | Blanks | 18 | |
| Almoner of Charitable Funds | three years | Susan Forgea | 52 | elected |
| Almoner of Charitable Funds | three years | Blanks | 8 | |
| Board of Health | three years | Judith Bogart | 56 | elected |
| Board of Health | three years | Blanks | 4 | |
| Vocational School Committee | three years | Ryan Strong | 55 | elected |
| Vocational School Committee | three years | Blanks | 5 | |
| Trustee, Bryant Library | five years | Wynne Busby | 54 | elected |
| Trustee, Bryant Library | five years | Blanks | 6 | |
| Trustee, Bryant Library | one year | Stephanie Wondriska-Clark | 54 | elected |
| Trustee, Bryant Library | one year | Blanks | 6 | |
| Water Commissioner | three years | Dann Emerson | 54 | elected |
| Water Commissioner | three years | Blanks | 6 | |
| Recreation Committee | three years | Stacey Mackowiak | 53 | elected |
| Recreation Committee | three years | Eliza Dragon | 49 | elected |
| Recreation Committee | three years | Emily Michalenko | 54 | elected |
| Recreation Committee | three years | Brian Gillman | 7 | write-in |
| Recreation Committee | three years | Blanks | 17 | |
| Planning Board | five years | Dennis Carr | 50 | elected |
| Planning Board | five years | Blanks | 10 | |
| Municipal Light Plant Board | three years | Brenda Arbib | 55 | elected |
| Municipal Light Plant Board | three years | Andrew Liebenow | 57 | elected |
| Municipal Light Plant Board | three years | Blanks | 8 | |
| Municipal Light Plant Board | two years | Theodore Lynds | 54 | elected |
| Municipal Light Plant Board | two years | Blanks | 6 | |
| Commissioner of Trust Funds | three years | Kenneth Howes | 59 | elected |
| Commissioner of Trust Funds | three years | Blanks | 1 | |

## Arithmetic check: seventeen offices, seventeen exact sums

Cummington counts blanks as their own row, so every office must sum to exactly the
ballots cast, and a multi-seat office to an exact multiple of it. All seventeen do,
against 60:

    Selectman, 3 yr              54 + 6                      =  60  = 60 x 1
    Selectman, 1 yr              53 + 1 + 6                  =  60  = 60 x 1
    Assessor                     53 + 7                      =  60  = 60 x 1
    Moderator                    52 + 1 + 7                  =  60  = 60 x 1
    Town Clerk                   59 + 1                      =  60  = 60 x 1
    Finance Committee            53 + 49 + 18                = 120  = 60 x 2
    Almoner of Charitable Funds  52 + 8                      =  60  = 60 x 1
    Board of Health              56 + 4                      =  60  = 60 x 1
    Vocational School Committee  55 + 5                      =  60  = 60 x 1
    Trustee, Bryant Library 5yr  54 + 6                      =  60  = 60 x 1
    Trustee, Bryant Library 1yr  54 + 6                      =  60  = 60 x 1
    Water Commissioner           54 + 6                      =  60  = 60 x 1
    Recreation Committee         53 + 49 + 54 + 7 + 17       = 180  = 60 x 3
    Planning Board               50 + 10                     =  60  = 60 x 1
    Municipal Light Plant, 3 yr  55 + 57 + 8                 = 120  = 60 x 2
    Municipal Light Plant, 2 yr  54 + 6                      =  60  = 60 x 1
    Commissioner of Trust Funds  59 + 1                      =  60  = 60 x 1

Seventeen independent sums, none off by one. The seat counts in the table above
are read off those multiples: the Finance Committee and the three-year Municipal
Light Plant Board seats each seat two, the Recreation Committee seats three, and
every other office seats one.

## Not a contested election

No office had more candidates than seats, and every named candidate was elected.
The only names that did not take a seat are the three write-ins the clerk listed
under his own stated policy of printing every registered write-in name: Ryan
Strong (1) for the one-year Selectman seat, Mark Demaranville (1) for Moderator,
and Brian Gillman (7) for the Recreation Committee.

Two people appear twice on the ballot. Ryan Strong drew a single write-in vote for
Selectman and was separately elected to the Vocational School Committee with 55.
Theodore Lynds took the two-year Municipal Light Plant Board seat while June Lynds
took the three-year Selectman seat; the record states no relationship between
them and none is inferred here.
