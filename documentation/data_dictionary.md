# Data dictionary
 
Every file, every column, and how each derived value is calculated. All data covers the **2025/26 Premier League season** (15 Aug 2025 to 24 May 2026) and comes from [Understat](https://understat.com).
 
## Contents
 
1. [Files at a glance](#files-at-a-glance)
2. [Key concepts](#key-concepts)
3. [`epl_2025_matches.csv` (raw)](#epl_2025_matchescsv-raw)
4. [`matches.csv`](#matchescsv)
5. [`season_table.csv`](#season_tablecsv)
6. [`home_away.csv`](#home_awaycsv)
7. [Relationships in Power BI](#relationships-in-power-bi)
8. [Data quality checks](#data-quality-checks)
9. [Known quirks](#known-quirks)
---
 
## Files at a glance
 
| File | Grain (one row per) | Rows | Columns | Created by |
|---|---|---|---|---|
| `epl_2025_matches.csv` | team per match | 760 | 23 | `collection.ipynb` |
| `matches.csv` | team per match | 760 | 27 | `data_cleaning.ipynb` |
| `season_table.csv` | team | 20 | 27 | `data_cleaning.ipynb` |
| `home_away.csv` | team and venue | 40 | 10 | `data_cleaning.ipynb` |
 
Each real match appears **twice** in the match-level files, once from each team's perspective. 20 teams x 38 matches = 760 rows.
 
---
 
## Key concepts
 
| Term | Meaning |
|---|---|
| **xG** (expected goals) | The quality of a team's shots, summed. Each shot is given a probability of being a goal based on factors like location and shot type. A team's xG is how many goals it "should" have scored. |
| **xGA** (expected goals against) | The quality of chances a team allowed. How many goals it "should" have conceded. |
| **xPTS** (expected points) | The points a team "should" have earned from a match, given the chances created and conceded by both sides. Always between 0 and 3 per match. |
| **npxG / npxGA** | xG and xGA with penalties removed, so a single penalty does not distort a team's open-play quality. |
| **Deep completions** | Completed passes within roughly 20 metres of the opponent's goal. A proxy for how much a team operates in dangerous territory. |
| **PPDA** (passes per defensive action) | Opponent passes allowed per defensive action by this team, measured outside its own defensive third. **Lower = more aggressive pressing.** |
| **Luck** | Actual points minus expected points. Negative means the team got fewer points than its performances deserved. |
 
---
 
## `epl_2025_matches.csv` (raw)
 
The unmodified table produced by flattening Understat's JSON. 760 rows, 23 columns. Numbers may be stored as text in the source, which is why the cleaning step forces numeric types.
 
| Column | Type | Description |
|---|---|---|
| `h_a` | text | `h` = the team played at home, `a` = away |
| `date` | datetime | Kickoff date and time, e.g. `2025-08-15 19:00:00` |
| `result` | text | `w` = win, `d` = draw, `l` = loss |
| `wins` | 0/1 | 1 if this team won this match (a flag, not a running total) |
| `draws` | 0/1 | 1 if the match was a draw |
| `loses` | 0/1 | 1 if this team lost |
| `pts` | integer | Points earned in this match: 3, 1 or 0 |
| `scored` | integer | Goals scored by this team |
| `missed` | integer | Goals conceded by this team |
| `xG` | decimal | Expected goals created |
| `xGA` | decimal | Expected goals against (the opponent's xG) |
| `npxG` | decimal | Expected goals excluding penalties |
| `npxGA` | decimal | Expected goals against excluding penalties |
| `npxGD` | decimal | Non-penalty expected goal difference: `npxG - npxGA` |
| `xpts` | decimal | Expected points for this match (0 to 3) |
| `deep` | integer | Deep completions by this team |
| `deep_allowed` | integer | Deep completions by the opponent against this team |
| `ppda.att` | decimal | PPDA numerator: opponent passes outside the pressing team's defensive third |
| `ppda.def` | decimal | PPDA denominator: this team's defensive actions (tackles, interceptions and similar) |
| `ppda_allowed.att` | decimal | Same components, describing the pressing the opponent applied against this team |
| `ppda_allowed.def` | decimal | Same as above, denominator |
| `team_id` | integer | Understat's team identifier (added during collection) |
| `team` | text | Team name (added during collection) |
 
> The two `ppda_allowed` columns describe the opposing team's pressing against this team. Their exact definition is not documented by Understat, so the description above is an interpretation of the column names.
 
---
 
## `matches.csv`
 
The cleaned match-level table that feeds the team pages in Power BI. 760 rows, 27 columns, numbers rounded to 3 decimals.
 
### Changes from the raw file
 
- Dotted column names renamed: `ppda.att` to `ppda_att`, `ppda.def` to `ppda_def`, `ppda_allowed.att` to `ppda_allowed_att`, `ppda_allowed.def` to `ppda_allowed_def`.
- `date` kept as a date only (no time).
- `h_a` and `result` replaced with readable labels.
- `wins`, `draws` and `loses` dropped.
- Running totals, per-match luck and rolling averages added.
### Columns
 
| Column | Type | Description |
|---|---|---|
| `team` | text | Team name. Links to `season_table[team]` |
| `team_id` | integer | Understat team ID |
| `match_no` | integer | The team's match number in chronological order, 1 to 38 |
| `date` | date | Match date |
| `venue` | text | `Home` or `Away` |
| `result_label` | text | `Win`, `Draw` or `Loss` |
| `pts` | integer | Points earned in this match |
| `xpts` | decimal | Expected points for this match |
| `match_luck` | decimal | `pts - xpts` for this match |
| `cum_pts` | decimal | Running total of points up to and including this match |
| `cum_xpts` | decimal | Running total of expected points |
| `cum_luck` | decimal | `cum_pts - cum_xpts`. Shows when in the season luck built up |
| `scored` | integer | Goals scored |
| `missed` | integer | Goals conceded |
| `xG` | decimal | Expected goals created |
| `xGA` | decimal | Expected goals against |
| `npxG` | decimal | Non-penalty xG |
| `npxGA` | decimal | Non-penalty xGA |
| `npxGD` | decimal | `npxG - npxGA` |
| `deep` | integer | Deep completions by this team |
| `deep_allowed` | integer | Deep completions allowed |
| `ppda_att` | decimal | PPDA numerator |
| `ppda_def` | decimal | PPDA denominator |
| `ppda_allowed_att` | decimal | Opponent pressing numerator |
| `ppda_allowed_def` | decimal | Opponent pressing denominator |
| `xg_roll5` | decimal | 5-match rolling average of `xG` |
| `xga_roll5` | decimal | 5-match rolling average of `xGA` |
 
**Missing values:** `xg_roll5` and `xga_roll5` are blank for each team's first 4 matches (80 blank cells in total), because a 5-match average needs 5 matches. Nothing else is missing.
 
**Important:** `cum_pts`, `cum_xpts` and `cum_luck` are running totals. Use **Maximum** (not Sum) when charting them for a single team, and do not use them as per-match values.
 
---
 
## `season_table.csv`
 
One row per team with season totals and the luck metrics. 20 rows, 27 columns, numbers rounded to 2 decimals. Rows are sorted by real league position.
 
### Totals
 
| Column | Type | Description |
|---|---|---|
| `team` | text | Team name. The unique key of this table |
| `played` | integer | Matches played (38 for every team) |
| `pts` | integer | Points won |
| `xpts` | decimal | Expected points (sum of match xPTS) |
| `gf` | integer | Goals for |
| `ga` | integer | Goals against |
| `gd` | integer | Goal difference: `gf - ga` |
| `xg` | decimal | Expected goals created |
| `xga` | decimal | Expected goals against |
| `xgd` | decimal | Expected goal difference: `xg - xga` |
| `npxg` | decimal | Non-penalty xG |
| `deep` | integer | Deep completions, season total |
| `deep_allowed` | integer | Deep completions allowed, season total |
| `ppda` | decimal | Season PPDA: total `ppda_att` divided by total `ppda_def`. Lower = more pressing |
 
### Luck metrics
 
| Column | Type | Formula | How to read it |
|---|---|---|---|
| `pts_luck` | decimal | `pts - xpts` | Negative = unlucky, positive = lucky |
| `finishing` | decimal | `gf - xg` | Positive = scoring more than the chances deserved |
| `defending` | decimal | `xga - ga` | Positive = conceding fewer than the chances against deserved |
| `luck_label` | text | `pts_luck <= -3` Unlucky, `>= 3` Lucky, else As expected | Used to colour charts. The +/-3 threshold is a judgement call |
 
### Rankings
 
| Column | Type | Description |
|---|---|---|
| `table_rank` | integer | Real league position, sorted by points, then goal difference, then goals scored |
| `xpts_rank` | integer | Position if teams were ranked by expected points (ties broken by order of appearance) |
| `rank_gap` | integer | `table_rank - xpts_rank`. **Positive = deserved a higher position** (unlucky). **Negative = sits higher than deserved** (lucky) |
 
### Per-game versions
 
| Column | Formula |
|---|---|
| `pts_pg` | `pts / played` |
| `xpts_pg` | `xpts / played` |
| `gf_pg` | `gf / played` |
| `ga_pg` | `ga / played` |
| `xg_pg` | `xg / played` |
| `xga_pg` | `xga / played` |
 
Per-game columns let you compare complete and incomplete seasons fairly if more data is added later.
 
---
 
## `home_away.csv`
 
One row per team and venue. 40 rows (20 teams x 2 venues), 10 columns, numbers rounded to 2 decimals.
 
| Column | Type | Description |
|---|---|---|
| `team` | text | Team name. Links to `season_table[team]` |
| `venue` | text | `Home` or `Away` |
| `played` | integer | Matches played at this venue (19 each) |
| `pts` | integer | Points won at this venue |
| `xpts` | decimal | Expected points at this venue |
| `gf` | integer | Goals for |
| `ga` | integer | Goals against |
| `xg` | decimal | Expected goals created |
| `xga` | decimal | Expected goals against |
| `luck` | decimal | `pts - xpts` at this venue |
 
---
 
## Relationships in Power BI
 
`season_table` is the "one" side. It filters the other two tables, so a team slicer based on it filters everything.
 
| From (one) | To (many) | Column | Direction |
|---|---|---|---|
| `season_table[team]` | `matches[team]` | `team` | Single |
| `season_table[team]` | `home_away[team]` | `team` | Single |
 
Because of this, a slicer on `season_table[team]` also shrinks any visual built from `season_table`. The league-wide visuals therefore sit on a separate page from the team slicer, or have their interactions edited off.
 
---
 
## Data quality checks
 
Run in `data_cleaning.ipynb`, all passed:
 
| Check | Result |
|---|---|
| Number of teams | 20 |
| Matches per team | 38 for all 20 |
| Duplicate team and date rows | 0 |
| Missing values in the raw data | 0 |
| `pts` disagreeing with `result` | 0 |
| Match `xpts` range | 0.008 to 2.985 |
 
---
 
## Known quirks
 
- **League-wide, xG exceeds goals scored.** Total xG is 1,162.4 against 1,045 goals, and total xPTS is 1,055.9 against 1,036 points won. As a result, almost every team has negative `finishing` and slightly negative `pts_luck`. Compare teams with each other, and read single values with that in mind.
- **Each match appears twice.** Summing a match-level column across all teams counts every match from both sides. This is intended, but keep it in mind for league totals such as total goals.
- **`team` names are Understat's.** For example, `Wolverhampton Wanderers` and `Manchester United` are written in full, which matters if you join to another source.
- **PPDA is calculated from totals.** Season `ppda` is total attempts divided by total actions, not the average of 38 match PPDA values. The two differ slightly.
- **Source stability.** Understat has no official API. The endpoint used here may change, and the data belongs to Understat.
