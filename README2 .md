# 911 Calls — Exploratory Data Analysis

Analysis of `911.csv`, a log of 911 (emergency) call records from Montgomery County, PA, covering **December 10, 2015 – August 24, 2016**. The notebook `911_Calls.ipynb` builds this dataset step by step; this README summarizes the findings, with a focus on interpreting the heatmaps and clustermap it produces.

## Dataset at a glance

| | |
|---|---|
| Rows | 99,492 calls |
| Columns | `lat`, `lng`, `desc`, `zip`, `title`, `timeStamp`, `twp`, `addr`, `e` |
| Date range | 2015-12-10 to 2016-08-24 (~8.5 months, **no Sep–Nov 2015 or 2016 data**) |
| Missing values | `zip`: 12,855 missing · `twp`: 43 missing · `addr`: 519 missing |
| Unique call titles | 110 |

Two engineered columns drive most of the analysis:
- **`Reason`** — extracted from `title` (text before the colon): `EMS`, `Fire`, or `Traffic`.
- **`Hour` / `Month` / `Day of Week` / `Date`** — parsed from `timeStamp`.

> **Note on coverage:** the dataset has no records for September, October, or November, and only partial months for December (from the 10th) and August (through the 24th). Any month-level comparison below should be read with that gap in mind — it isn't a full annual cycle.

## Key findings

**Call volume by reason** — EMS dominates:

| Reason | Calls | Share |
|---|---|---|
| EMS | 48,877 | 49.1% |
| Traffic | 35,695 | 35.9% |
| Fire | 14,920 | 15.0% |

![Calls by Reason](images/reason_countplot.png)

Medical emergencies are essentially 1-in-2 calls, traffic incidents roughly 1-in-3, and fire the smallest (but still substantial) share.

**Most common single call types:**
1. Traffic: Vehicle Accident — 23,066
2. Traffic: Disabled Vehicle — 7,702
3. Fire: Fire Alarm — 5,496
4. EMS: Respiratory Emergency — 5,112
5. EMS: Cardiac Emergency — 5,012

**Top townships by call volume:** Lower Merion (8,443), Abington (5,977), Norristown (5,890), Upper Merion (5,227), Cheltenham (4,575). Lower Merion leads in every category (EMS, Fire, *and* Traffic), consistent with it being the county's largest township by population.

**Top zip codes:** 19401 (Norristown, 6,979 calls), 19464 (Pottstown, 6,643), 19403, 19446, 19406.

**Day-of-week totals** are fairly even, with a mild dip on weekends:

Tue (15,150) > Wed (14,879) > Fri (14,833) > Mon (14,680) > Thu (14,478) > Sat (13,336) > Sun (12,136)

![Calls by Day of Week and Reason](images/dayofweek_reason.png)

![Top Townships](images/top_townships.png)

## Heatmap & clustermap interpretation

### 1. Day of Week × Hour of Day

To see *when* calls happen, the notebook reshapes the data into a table with `Day of Week` as rows and `Hour` (0–23) as columns, each cell counting calls, then visualizes it two ways.

**Heatmap:**

![Heatmap: Day vs Hour](images/heatmap_dayhour.png)

Reading the heatmap:
- There's a clear **dark band overnight (roughly midnight–6 AM)** every day — this is the quietest period, bottoming out at **156 calls on Wednesday at 4 AM**, the single lowest cell in the whole matrix.
- Volume **ramps up sharply from 7 AM**, staying elevated through the entire midday and afternoon.
- The **brightest region is weekday afternoons, 12 PM–6 PM**, peaking at **1,039 calls on Friday at 4 PM** — the busiest single hour/day combination in the dataset. Monday, Tuesday, and Wednesday all also peak around 4–5 PM with similarly high counts.
- **Weekends (Saturday/Sunday) look different**: overnight hours (0–6 AM) are noticeably *brighter* than on weekdays — people are awake later and call more (e.g., Saturday 1 AM has ~300 calls vs. ~220–270 on weekdays) — but the weekend afternoon peak is *lower* and flatter than the weekday peak, and it fades out earlier in the evening. This is a classic weekday-commute-and-workday pattern (accidents, work-related medical events) layered on top of a baseline of emergencies that happens any time.

**Clustermap:**

![Clustermap: Day vs Hour](images/clustermap_dayhour.png)

The clustermap runs hierarchical clustering on both axes of that same table and reorders rows/columns so similar patterns sit next to each other, with dendrograms showing how they group.

- **Columns (hours) split into three clear clusters**, matching intuition:
  - A **low-activity cluster**: hours 0–6 and 23 (late night/early morning) — these all cluster together first, confirming they behave alike (uniformly low).
  - A **medium cluster**: hours 7–8 and 19–22 (early morning ramp-up and evening wind-down).
  - A **high-activity cluster**: hours 9–18 (late morning through early evening) — the core "daytime" block, with 16 and 17 (4–5 PM) forming their own tight pair inside it, confirming that hour as the true peak.
- **Rows (days) split into two groups**: {Mon, Tue, Wed, Thu, Fri} cluster together as "weekdays," while **Saturday and Sunday cluster together separately** as "weekend." This is the clustermap's main takeaway — the algorithm rediscovers the weekday/weekend split purely from call patterns, without being told which days are weekends. Within weekdays, Tuesday/Monday/Wednesday group most closely (very similar hourly shape), while Friday and Thursday are slightly more distinct — consistent with Friday's slightly earlier/later spread noted above.

### 2. Day of Week × Month

The same technique is repeated with `Month` as the columns instead of `Hour`.

**Heatmap:**

![Heatmap: Day vs Month](images/heatmap_daymonth.png)

- The most visually striking cell is **Saturday in January (2,291 calls)** — by far the brightest square on the grid, well above every other day/month combination.
- The **darkest region is December** across all days (bottoming out at 907 calls on Sunday), but December only has ~21 days of data (from the 10th) in this dataset, so its totals are naturally lower — this isn't necessarily a real seasonal dip, just a partial month.
- **August is also comparatively dark**, again partly because it's cut off after the 24th.
- Excluding those two partial months, activity is fairly stable across the year, with a mild bump around **June** (Wed/Thu are notably bright in June) and **January** weekends.

**Clustermap:**

![Clustermap: Day vs Month](images/clustermap_daymonth.png)

- **Columns (months) cluster August and December together first** — confirming, quantitatively, that these two behave alike and differently from the rest, which lines up with them being the two partial/incomplete months rather than a genuine "low season." The remaining months (1, 2, 3, 4, 5, 6, 7) form a second, looser cluster of "normal, fully-observed" months.
- **Rows (days) cluster Saturday and Sunday together** again, separate from the five weekdays — the same weekend-vs-weekday structure seen in the hour-based clustermap, reappearing independently in the month-based view. This is a good consistency check: two different groupings of the same underlying data both recover the same day-of-week split.
- Within weekdays, Wednesday and Thursday cluster most tightly (both spike in June), while Friday sits a bit apart (its own high month is July, not June).

**Takeaway from both clustermaps:** the dendrograms consistently isolate a "weekend" cluster (Sat + Sun) from a "weekday" cluster (Mon–Fri) — this is the strongest and most reliable pattern in the whole dataset. On the column side, they separate genuine behavioral clusters (hour-of-day case) from an artifact of incomplete data (month case) — a useful reminder that clustering will faithfully group whatever pattern is in the data, whether it's meaningful (daytime vs. night) or an artifact (partial months), so results should always be checked against what's actually being measured.

## Files in this analysis

```
README.md                          this file
images/
  reason_countplot.png             call volume by Reason (EMS/Traffic/Fire)
  dayofweek_reason.png             calls by day of week, split by Reason
  top_townships.png                top 10 townships by call volume
  heatmap_dayhour.png              heatmap: Day of Week × Hour
  clustermap_dayhour.png           clustermap: Day of Week × Hour
  heatmap_daymonth.png             heatmap: Day of Week × Month
  clustermap_daymonth.png          clustermap: Day of Week × Month
```

## How to reproduce

```python
import pandas as pd, seaborn as sns, matplotlib.pyplot as plt

df = pd.read_csv('911.csv')
df['Reason'] = df['title'].apply(lambda t: t.split(':')[0])
df['timeStamp'] = pd.to_datetime(df['timeStamp'])
df['Hour'] = df['timeStamp'].dt.hour
df['Month'] = df['timeStamp'].dt.month
df['Day of Week'] = df['timeStamp'].dt.dayofweek.map(
    {0:'Mon',1:'Tue',2:'Wed',3:'Thu',4:'Fri',5:'Sat',6:'Sun'})

dayHour = df.groupby(['Day of Week','Hour']).count()['Reason'].unstack()
sns.heatmap(dayHour, cmap='viridis')
sns.clustermap(dayHour, cmap='viridis')

dayMonth = df.groupby(['Day of Week','Month']).count()['Reason'].unstack()
sns.heatmap(dayMonth, cmap='viridis')
sns.clustermap(dayMonth, cmap='viridis')
```
