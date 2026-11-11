# 1\. Attempt — Diagnosis and planned decisions (THINK stage)

File audited: `teaching\_prices\_messy.csv` (97 rows × 6 columns: date, item, city, price, unit, data\_label). Invented teaching data.

## Diagnosis and planned decisions

|#|Problem|Code that revealed it|Planned decision|Reason|
|-|-|-|-|-|
|1|`price` is read as **text**; one value has a thousands comma: `"1,250"` (Staple A, City 2, 2026-01-15)|`df.dtypes`; `pd.to\_numeric(..., errors='coerce')`|**Correct** the type (strip the comma, convert to number). Then see problem 5|A text column cannot be averaged or compared. The comma is a formatting problem, but the number it gives (1,250) is still implausible|
|2|**Missing price**: Pulse C, City 2, 2026-01-01|`df.isna().sum()`|**Retain** the row, leave price missing, **flag** it. Do not fill|Any fill value would be invented. Dropping the row would hide that this observation exists and break the 12-per-series pattern|
|3|**Exact duplicate row**: Staple A, City 1, 2026-01-08 appears twice (rows 8 and 96)|`df.duplicated()`|**Exclude** the second copy|Identical in every column; the date-item-city key must be unique. No information is lost|
|4|**Inconsistent label** `oil b` (lower case) on 2026-02-05, City 1|`df\['item'].value\_counts()`|**Investigate**, then **correct** the label. It is Staple A, not Oil B|Oil B City 1 already has a row on that date (318.0); Staple A City 1 has none; 108.5 fits Staple A City 1 (range 99.5–116); and 114.5 for Staple A City 2 on that date is exactly 6 higher, the same gap as every other pair. Changing case alone would wrongly make it a second Oil B row|
|5|**Impossible / sentinel prices**: `9999` (Sweetener D, City 1, 2026-01-22) and `1,250` (problem 1)|`describe()` by item; ratio of each price to its own item-city median (60× and 11×)|**Exclude** the values (set to missing), keep the rows, **flag** them|Neighbouring prices are 105–116 (Staple A) and 149–185 (Sweetener D). 9999 is a placeholder code. The intended value of 1,250 can't be known|
|6|`date` is stored as text|`df.dtypes`|**Correct**: convert to datetime|Needed for sorting and time analysis; the 12 weekly dates are all valid|
|7|`unit` and `data\_label` are the same in every row|`value\_counts()`|**Retain**|They document that these are illustrative prices in PKR per kg, not real observations|

## Left alone on purpose

* Prices that go down from one week to the next (e.g. Staple A 101.0 → 99.5). They are within each series' normal range and are genuine variation.
* The 0.5 decimals (99.5, 202.5, ...). They are real precision, not noise.
* The 12 weekly dates and the two city labels. They are clean.

## Expected structure after cleaning

4 items × 2 cities × 12 dates = **96 rows**, each date-item-city once, **3 prices missing** (problems 2 and 5), 93 priced observations.



