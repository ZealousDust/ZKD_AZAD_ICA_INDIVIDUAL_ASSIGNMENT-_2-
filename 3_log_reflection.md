# 3\. Log and Reflection

Name: Zeeshan Khan · ERP: 32133

The ten "cleaning lines" below are the standard one-liners an automated cleaner typically suggests, written by me and run on a copy. Each was tried on its own in `2\_work.ipynb` (section 2) against a baseline where price was converted from text to numbers.

## Cleaning Code Court

|#|Candidate line|Evidence (from the trial table)|Verdict|
|-|-|-|-|
|L1|`df.drop\_duplicates()`|Rows 97 → 96; key duplicates 1 → 0; valid observations lost: 0. The dropped row is identical to row 8 in every column|**Keep**|
|L2|`df\['item'] = df\['item'].str.strip().str.title()`|Changes 1 label (`oil b` → `Oil B`) and raises key duplicates from 1 to 2: two "Oil B, City 1, 2026-02-05" rows with prices 108.5 and 318.0. The row is really Staple A|**Modify**: correct that one label by evidence, not by changing case. Keep `.strip()` as a safety step (it changes nothing here)|
|L3|`df.dropna()`|Removes the Pulse C, City 2, 2026-01-01 row, so that series has 11 rows instead of 12. Leaves 1,250 and 9999 untouched|**Reject**: keep the row with a missing price and a flag|
|L4|`df\['price'].fillna(df\['price'].mean())`|Would put **315.5** (mean pooled over all items) in a Pulse C slot where every price is 200–213|**Reject**: invents data and distorts the mean|
|L5|`df\[df\['price'] < 1000]`|Rows 97 → 94. Removes 1,250 and 9999 but also silently removes the missing-price row (`NaN < 1000` is False). Arbitrary cut-off; cannot see the mislabelled row|**Modify**: keep the idea (screen for impossible prices) but flag per item-city series and set to missing instead of deleting rows|
|L6|`df\[df\['price'] != 9999]`|Removes only the 9999 row; maximum price afterwards is still **1,250**|**Modify**: set 9999 to missing, keep the row, and also handle 1,250|
|L7|`df\['price'] = df\['price'].round(0)`|Changes **35 valid prices** (e.g. 99.5 → 100.0, 202.5 → 202.0)|**Reject**: changes valid numbers without need; the 0.5 precision is real|
|L8|`df.drop(columns=\['unit', 'data\_label'])`|No numbers change, but the "illustrative, not actual PBS" label and the unit disappear|**Reject**: documentation must stay with the data|
|L9|pooled IQR filter on `price`|Fences about −100 to 547; removes the two bad values (0 valid rows) but the fences are pooled across items, so a wrong Staple A price of 300 would pass; cannot see the mislabelled row|**Reject** as the main method; use a per-item-city comparison and flag, don't delete|
|L10|`df\['price'].clip(upper=quantile(.99))`|Turns 9999 into **1,687.45**, a number that never existed; 1,250 stays; 0 valid values changed|**Reject**: manufactures a price|

Also kept in the final code (checked with `dtypes` and asserts): `pd.to\_numeric(price.str.replace(',', ''))` (turns "1,250" into a number; the value is then flagged as implausible), `pd.to\_datetime(date)` and `sort\_values(\['date','item','city'])`.

## Before / after (from `2\_work.ipynb`, section 4)

|Measure|Before|After|
|-|-|-|
|Rows|97|96|
|Distinct items|5 (`oil b` is a stray)|4|
|Distinct dates|12|12|
|Exact duplicate rows|1|0|
|Missing price|1|3 (flagged)|
|Price min / median|99.5 / 200.0|99.5 / 200.0|
|Price max|9,999|350|
|Price mean|315.46|203.61|

**Valid observations removed:** none. All 93 genuine priced observations are still there, and each has the same value as in the raw file (0 differences).

**Does every item appear the expected number of times?** Yes: each item-city appears **12** times, each date has **8** rows, each date-item-city key appears **once**, and there are exactly 4 items.

**Extra check, made after the decisions:** against the Week 1 `teaching\_prices\_clean.csv`, all 93 priced values match, and the three values I left missing are 206.0, 109.0 and 157.5. I did not apply these, since the decision rules were made without that file.

## Log rows

|#|Change made|Rows affected|Check run and evidence|Decision|
|-|-|-|-|-|
|1|Dropped exact duplicate|1 (row 96)|`duplicated()` 1 → 0; identical to row 8|Accepted|
|2|Relabelled `oil b` → Staple A, flagged|1 (2026-02-05, City 1)|Staple A City 1 had no row that date; Oil B City 1 already had 318.0; 108.5 within Staple A range and +6 gap to City 2|Accepted|
|3|Price text → number (comma removed)|1 text value (`"1,250"`)|`to\_numeric` result; no other value changed (0 differences)|Accepted|
|4|`1,250` set to missing, flagged|1|11× its series median; neighbours 105–116|Accepted|
|5|`9999` set to missing, flagged|1|60× its series median; sentinel code|Accepted|
|6|Missing price left missing, flagged|1|Not filled, not dropped|Accepted|
|7|Date text → datetime, rows sorted|all|dtype check; row count unchanged|Accepted|
|8|`round`, `fillna(mean)`, `dropna`, `clip`, column drops, pooled IQR|—|See court table|Rejected|

## Reflection

**Where my decisions differed from the standard cleaning lines, and which was better supported.** The standard lines tend to treat the file as a pile of cells: drop or fill the missing value, filter out big numbers, round, tidy the labels. My decisions treated it as a ledger with a structure (4 items × 2 cities × 12 dates). That was better supported because I could test every step against the structure (12 per series, 8 per date, each key once), and the standard lines failed that test: `dropna` and `< 1000` shrink the Pulse C and Sweetener D series, `fillna(mean)` and `clip` invent numbers (315.5 and 1,687.45), and `round` changes 35 valid prices.

**What a pipeline without these checks would have missed.** The `oil b` row. Changing its case with `.str.title()` makes the table look tidy while creating two "Oil B, City 1" rows on 2026-02-05 and leaving Staple A City 1 with only 11 observations. A price filter doesn't catch it either, because 108.5 is a normal price for Staple A. It only showed up by counting rows per item and city, and by checking what was actually missing on that date. I also would not have noticed that `< 1000` removes the missing-price row as a side effect.

**What I would still do differently with more information.** I flagged the three bad or missing prices instead of repairing them. If the source system or the intern could be asked, the right next step is to get the true values rather than guess.

