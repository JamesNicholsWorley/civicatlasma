# Marshfield 2008, 2011, 2013 were never harvested

These three town-years look collected and are not. The crawl that should have
fetched Marshfield's annual town report fetched Boylston's instead -- the two
towns share a revize CMS host -- so the slot was filled, the section was cut, and
nothing reported a gap. The records have been re-keyed to Boylston, which is what
they always were, and these three years of Marshfield are now openly missing.

Candidate URLs already in `config/atr_pre2021_urls.csv`, none of them tried:

    2008  http://marshfield-ma.gov/Collateral/Documents/English-US/accounting/TA-Town-Report-2008.pdf
    2011  http://marshfield-ma.gov/Collateral/Documents/English-US/administration/MARSHFIELD2011TOWNREPORT-WEBCOPY.pdf
    2011  https://marshfield-ma.gov/government/select_board/2011_annual_report_-_pdf.pdf
    2013  https://marshfield-ma.gov/government/select_board/2013_annual_report_-_pdf.pdf

The wider check that found this: of 4,451 sectioned documents, 1,216 were fetched
from a URL that does not name their own town, and 53 of those name a different
revize tenant -- several of them out of state (bellflowerca, akronoh,
berwickmaine, bennington in Vermont). Only these three reached a published
record; the publication gate held the other fifty. The gate works. What it cannot
catch is the case where the wrong document names no town at all, which is 83 of
the 961 published records.
