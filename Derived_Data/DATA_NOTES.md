% Data Notes
% Curtis C. Bohlen, Casco Bay Estuary Partnership
% July, 2020

<img
    src="https://www.cascobayestuary.org/wp-content/uploads/2014/04/logo_sm.jpg"
    style="position:absolute;top:10px;right:50px;" />


# Data Extraction
Extracting data from PDF files is always problematic.

# Primary Approach
Here, we were able to copy tables from four of five PDFs, paste them into Word,
where the table structure was recognized, then copy them from Word into Excel.
In several cases, cells were badly mis-aligned, so we needed to shift data
horizontally to restore data alignment.

## 2017 Data Handling
That approach did not work for the data from 2017, which copied from the PDF in 
white-space delimited format.  We pasted these into Word, used global
search and replace to replace spaces with tabs, and then pasted into Excel.
We again cleaned up cell alignment by hand.  

This approach resulted in partial disaggregation of the "SHIP DETAILS" field.
We took those data and converted them, largely by hand, to LOA, B and D fields. 

Data on the Cat Ferry was included in the 2017 data.  2017 was the only year
during this period in which the Cat Ferry operated out of Portland. Data on the
Ferry is less complete than data on Cruise ships.  For example, it does not
include data on the number of passemgers or crew.

We separated
the ferry data from data on other ships, to simplify data reorganization and
cleaning. Ferry data is retained in the Excel sheet, but the ferry data is not
interleaved chronologically with the cruise ship data.

Users should think carefully abotu whether to nclude the Ferry data in their
analysis or not.  If your focus ins on cruise ships, you may want to omit this
vessel, but if your emphasis is on the use of the port, then it makes more
sense to retain it.

Data from 2017 did not include information on the number of staff on board.

## Data Manipulation in Excel
We  derived length, beam, and depth data from the "SHIP  DETAILS" field. We
did that using a combination of excel formulas and hand typing of data. 
We checked consistency by using pivot tables to ensure that data entry for 
each ship was consistent across entries and  sheets.

We determined whether each ship stayed at port over night by checking if the
arrival day of the week was the same as the departure day of the week.  Since
it is possible for a ship to depart after midnight, this is not strictly a
measure of whether the ships were in port overnight, but it's close.

We renamed a few data columns to avoid name conflicts in R.

# Notes
* Users should not rely on data being in chronological order.
* Data on number of passemgers appears to be a mixture of data on CAPACITY and
  actual numbers, or perhaps the ships sometimes cruise with different numbers
  of passengers and crew.  Total passenger numbers should probably be treated
  as approximations to the actual number of passengers.
  * Data on Ship Details appear to ihave some inconsistnecies too. Some ships
  appear to have reported LOA, othes WLL or other length nmeasures.  The D field
  appears especially inconsistent, referring either to depth or draught.

# Error Checking and Data Evaluation
We aggregated the annual data int oa single sheet of the spreadsheet to simplify
data QA/QC and import into R.

We used Pivot Tables in Excel and sorting to check whether Ship Details
(Name, LOA, B. D) differed among different entries for each ship.  We found
many, generally minor, inconsistencies.  Several ships were listed in slightly
different ways in different years.  For  example, the ship listed as 
"Adventure of the Sea" in 2019 was listed as "Adventure of the seas" in 2018.
In 2018, it was listed with a beam of 128 feet, while it was listed in 2019 as
having a beam of 127 feet.

We corrected such inconsistencies ONLY in the "Cruise" tab of the Excel
Spreadsheet, which feeds into the pivot table sumamrizing properties or each vessel.

We checked spelling and presentation of ship names.  We corrected names
to match the most common form, or a name found on-ine if we could find one.

Similarly, Cruise Lines were recorded slightly differently from year to year, 
generally due to differneces in abbreviations used.  We tried to correct these
inconsistencies, but without accompanying data againt which to check, it is
impossible to be certain whether our "corrections" introduced new errors.



