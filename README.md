# 911-Calls-Analysis
### Objective
Analysis of 911, a log of 911 (emergency) call records from Montgomery County, PA, covering December 10, 2015 – August 24, 2016. I Performed Exploratory data analysis using Python to check the reason why people call 911 the most. The reasons are: EMS(Emergency Medical Service), Fire and Traffic.

## Dataset at a glance

| | |
|---|---|
| Rows | 99,492 calls |
| Columns | `lat`, `lng`, `desc`, `zip`, `title`, `timeStamp`, `twp`, `addr`, `e` |
| Date range | 2015-12-10 to 2016-08-24 (~8.5 months, **no Sep–Nov 2015 or 2016 data**) |
| Missing values | `zip`: 12,855 missing · `twp`: 43 missing · `addr`: 519 missing |
| Unique call titles | 110 |

## Key findings

**Call volume by reason** — EMS dominates:

| Reason | Calls | Share |
|---|---|---|
| EMS | 48,877 | 49.1% |
| Traffic | 35,695 | 35.9% |
| Fire | 14,920 | 15.0% |

**Most common single call types:**
1. Traffic: Vehicle Accident — 23,066
2. Traffic: Disabled Vehicle — 7,702
3. Fire: Fire Alarm — 5,496
4. EMS: Respiratory Emergency — 5,112
5. EMS: Cardiac Emergency — 5,012

## Process

- Imported the CSV file into the Jupyter Notebook.
- The dataset included latitude, longitude, address, zip code, title, timestamp, and township (`twp`).
- Found that there are 110 unique title codes.
- Used Seaborn to create a countplot and discovered that the top **Reason** people call 911 is **EMS**.
- Used `.apply()` to create three new columns: `Hour`, `Month`, and `Day of Week`.
- Re-plotted with Seaborn as separate countplots by Month and Day of Week, and still found EMS is the top reason across both.
- Grouped by `Month` and plotted the counts, discovering that January–February had the most calls.
- Grouped by `Date` and found the same overall pattern, but for the EMS reason specifically, March had the most calls.
## Heatmap & clustermap interpretation
BETWEEN HOURS and DAY OF WEEK

**HEATMAP:**
Reading the heatmap:
- There's a clear **dark band overnight (roughly midnight–6 AM)** every day — this is the quietest period, bottoming out at **156 calls on Wednesday at 4 AM**, the single lowest cell in the whole matrix.
- Volume **ramps up sharply from 7 AM**, staying elevated through the entire midday and afternoon.
- The **brightest region is weekday afternoons, 12 PM–6 PM**, peaking at **1,039 calls on Friday at 4 PM** — the busiest single hour/day combination in the dataset. Monday, Tuesday, and Wednesday all also peak around 4–5 PM with similarly high counts.
- **Weekends (Saturday/Sunday) look different**: overnight hours (0–6 AM) are noticeably *brighter* than on weekdays — people are awake later and call more (e.g., Saturday 1 AM has ~300 calls vs. ~220–270 on weekdays) — but the weekend afternoon peak is *lower* and flatter than the weekday peak, and it fades out earlier in the evening. This is a classic weekday-commute-and-workday pattern (accidents, work-related medical events) layered on top of a baseline of emergencies that happens any time.

**Clustermap:**

- **Columns (hours) split into three clear clusters**, matching intuition:
  - A **low-activity cluster**: hours 0–6 and 23 (late night/early morning) — these all cluster together first, confirming they behave alike (uniformly low).
  - A **medium cluster**: hours 7–8 and 19–22 (early morning ramp-up and evening wind-down).
  - A **high-activity cluster**: hours 9–18 (late morning through early evening) — the core "daytime" block, with 16 and 17 (4–5 PM) forming their own tight pair inside it, confirming that hour as the true peak.
- **Rows (days) split into two groups**: {Mon, Tue, Wed, Thu, Fri} cluster together as "weekdays," while **Saturday and Sunday cluster together separately** as "weekend." This is the clustermap's main takeaway — the algorithm rediscovers the weekday/weekend split purely from call patterns, without being told which days are weekends. Within weekdays, Tuesday/Monday/Wednesday group most closely (very similar hourly shape), while Friday and Thursday are slightly more distinct — consistent with Friday's slightly earlier/later spread noted above.

The same process is repeated with MONTH and DAYS OF WEEK

**Heatmap:**
Reading the heatmap:
- The most visually striking cell is **Saturday in January (2,291 calls)** — by far the brightest square on the grid, well above every other day/month combination.
- The **darkest region is December** across all days (bottoming out at 907 calls on Sunday), but December only has ~21 days of data (from the 10th) in this dataset, so its totals are naturally lower — this isn't necessarily a real seasonal dip, just a partial month.
- **August is also comparatively dark**, again partly because it's cut off after the 24th.
- Excluding those two partial months, activity is fairly stable across the year, with a mild bump around **June** (Wed/Thu are notably bright in June) and **January** weekends.

**Clustermap:**

- **Columns (months) cluster August and December together first** — confirming, quantitatively, that these two behave alike and differently from the rest, which lines up with them being the two partial/incomplete months rather than a genuine "low season." The remaining months (1, 2, 3, 4, 5, 6, 7) form a second, looser cluster of "normal, fully-observed" months.
- **Rows (days) cluster Saturday and Sunday together** again, separate from the five weekdays — the same weekend-vs-weekday structure seen in the hour-based clustermap, reappearing independently in the month-based view. This is a good consistency check: two different groupings of the same underlying data both recover the same day-of-week split.
- Within weekdays, Wednesday and Thursday cluster most tightly (both spike in June), while Friday sits a bit apart (its own high month is July, not June).

Takeaway from both clustermaps: the dendrograms consistently isolate a "weekend" cluster (Sat + Sun) from a "weekday" cluster (Mon–Fri) — this is the strongest and most reliable pattern in the whole dataset. On the column side, they separate genuine behavioral clusters (hour-of-day case) from an artifact of incomplete data (month case) — a useful reminder that clustering will faithfully group whatever pattern is in the data, whether it's meaningful (daytime vs. night) or an artifact (partial months), so results should always be checked against what's actually being measured.
