# The Unlucky Team Detector: Premier League 2025/26
 
**Who is performing better than the league table suggests?**
 
Football is low-scoring, so results are noisy: a team can create the better chances and still lose. This project compares each Premier League team's **actual points** with its **expected points (xPTS)**, which are based on the quality of the chances it created and conceded, to find out who was lucky, who was unlucky, and where that luck came from.
 
Match-level data for the full 2025/26 season was collected from Understat with Python, cleaned and validated in pandas, and turned into an interactive Power BI dashboard.
 
-->
 
---
 
## Table of contents
 
1. [Key findings](#key-findings)
2. [Questions this project answers](#questions-this-project-answers)
3. [Data](#data)
4. [Method](#method)
5. [Dashboard](#dashboard)
6. [Repository structure](#repository-structure)
7. [How to run](#how-to-run)
8. [Limitations](#limitations)
9. [Next steps](#next-steps)
10. [Credits](#credits)
---
 
## Key findings
 
All numbers below come from the 760 team-matches in this repository (20 teams x 38 matches, 15 Aug 2025 to 24 May 2026). `Luck = actual points - expected points`.
 
**Most unlucky teams**
 
| Team | Actual pts | xPTS | Luck |
|---|---|---|---|
| Wolverhampton Wanderers | 20 | 35.44 | **-15.44** |
| Leeds | 47 | 56.50 | -9.50 |
| Crystal Palace | 45 | 53.88 | -8.88 |
| Tottenham | 41 | 49.25 | -8.25 |
| Chelsea | 52 | 58.85 | -6.85 |
 
**Luckiest teams**
 
| Team | Actual pts | xPTS | Luck |
|---|---|---|---|
| Aston Villa | 65 | 51.07 | **+13.93** |
| Sunderland | 54 | 42.03 | +11.97 |
| Fulham | 52 | 45.08 | +6.92 |
| Manchester United | 71 | 64.45 | +6.55 |
| Arsenal | 85 | 79.87 | +5.13 |
 
**What stands out**
 
- **The top three finished where their performances said they should.** Arsenal, Manchester City and Manchester United rank 1st, 2nd and 3rd both in the real table and in the xPTS table. Arsenal won the league with 85 points against 79.87 expected, and had the best defence by chance quality (33.13 xGA).
- **Aston Villa and Sunderland were the biggest beneficiaries.** Villa finished 4th but rank 12th by xPTS (a gap of 8 places). Sunderland finished 7th but rank 18th by xPTS (a gap of 11 places).
- **Wolves' season was unlucky on top of poor.** They finished bottom with 20 points, but their performances were worth about 15 more. They still rank only 19th by xPTS, so even with average luck they would have finished in the bottom two.
- **Leeds deserved a higher position:** 14th in the table, 8th by xPTS.
- **Crystal Palace wasted the most chances.** They scored 41 goals from 62.46 xG, a shortfall of 21.5 goals, the largest in the league. Chelsea (-14.2) and Brentford (-11.3) follow.
- **Several teams conceded far fewer than their chances against suggested:** Nottingham Forest (+13.7 goals better than xGA), Sunderland (+12.3) and Manchester City (+11.5). This usually points to strong goalkeeping, good fortune, or both.
- **Pressing styles vary widely.** Brighton press hardest (PPDA 8.88), followed by Arsenal (9.58) and Bournemouth (9.68). West Ham (13.84) press least.
- **Luck is not spread evenly across venues.** For example, Tottenham's season luck was concentrated at home (-11.1 points) while they were slightly lucky away (+2.9), and Aston Villa were lucky both at home (+8.4) and away (+5.5).
> **Reading the numbers correctly:** across the league, Understat's xG (1,162.4) is higher than goals actually scored (1,045), and expected points (1,055.9) are slightly higher than points won (1,036). So nearly every team shows negative "finishing" and slightly negative luck. Compare teams with each other rather than reading any single value as an absolute. See [Limitations](#limitations).
 
---
 
## Questions this project answers
 
1. **Who is lucky and who is unlucky?** Actual points vs expected points for all 20 teams.
2. **Does the table hide who played best?** The league table compared with the xPTS table, and the rank gap between them.
3. **Where does the luck come from?** Finishing (goals vs xG), defending (goals conceded vs xGA), and home vs away.
4. **What did one team's season look like?** A match-by-match view of form, goals vs xG, and the matches where results did not match performances.
---
 
## Data
 
| Item | Detail |
|---|---|
| Source | [Understat](https://understat.com) league data endpoint |
| League and season | English Premier League, 2025/26 (Understat season `2025`) |
| Period covered | 15 Aug 2025 to 24 May 2026 |
| Size | 760 rows: 20 teams x 38 matches (each real match appears twice, once from each team's side) |
| Collected with | Python `requests`, using the endpoint `https://understat.com/getLeagueData/EPL/2025` with the `X-Requested-With: XMLHttpRequest` header |
 
Understat has no official public API, so this endpoint could change in future. All data belongs to Understat. 
 
A full column-by-column description of every file is in **[documentation/data_dictionary.md](documentation/data_dictionary.md)**.
 
### Files in `Data/processed/`
 
| File | One row per | Rows | Purpose |
|---|---|---|---|
| `epl_2025_matches.csv` | team per match (raw download) | 760 | Unmodified data from Understat |
| `matches.csv` | team per match (cleaned, with extra columns) | 760 | Running totals, venue, rolling averages. Feeds the team pages |
| `season_table.csv` | team | 20 | Season totals and luck metrics. Feeds the league pages |
| `home_away.csv` | team and venue | 40 | Home vs away luck |
 
---
 
## Method
 
### 1. Collect (`notebooks/collection.ipynb`)
- Request the Understat league endpoint and parse the JSON response.
- Loop over every team's `history` list and flatten it with `pd.json_normalize` (this also splits the nested PPDA values into separate columns).
- Save the raw table to `epl_2025_matches.csv`.
### 2. Clean and validate (`notebooks/data_cleaning.ipynb`)
- Convert dates and force numeric types.
- Rename dotted column names (`ppda.att` to `ppda_att`) because Power BI does not handle dots well.
- **Validation checks, all passed:**
| Check | Result |
|---|---|
| Teams in the data | 20 |
| Matches per team | 38 for every team |
| Duplicate team-date rows | 0 |
| Missing values in the raw data | 0 |
| Points that disagree with the match result | 0 |
| Match xPTS range | 0.008 to 2.985 (always between 0 and 3) |
 
### 3. Engineer features (`notebooks/data_cleaning.ipynb`)
 
| Feature | Formula | How to read it |
|---|---|---|
| `pts_luck` | `pts - xpts` | Negative = unlucky, positive = lucky |
| `luck_label` | `pts_luck <= -3` Unlucky, `>= 3` Lucky, otherwise As expected | The +/-3 point threshold is a judgement call |
| `finishing` | `goals scored - xG` | Positive = scoring more than the chances deserved |
| `defending` | `xGA - goals conceded` | Positive = conceding fewer than the chances against deserved |
| `table_rank` | Sorted by points, then goal difference, then goals scored | Real league position |
| `xpts_rank` | Rank by expected points | Position the performances deserved |
| `rank_gap` | `table_rank - xpts_rank` | Positive = deserved a higher position |
| `match_luck`, `cum_luck` | Per-match and running `pts - xpts` | Shows when in the season luck happened |
| `xg_roll5`, `xga_roll5` | 5-match rolling average of xG and xGA | Attacking and defensive momentum |
| `ppda` | `ppda_att / ppda_def` over the season | Lower = more aggressive pressing |
 
### 4. Export for Power BI
Three tidy CSVs (`matches.csv`, `season_table.csv`, `home_away.csv`) with consistent team names for relationships.
 
### 5. Visualize (`PowerBi/pl dashboard.pbix`)
`season_table` is the "one" side of one-to-many relationships (on `team`) to `matches` and `home_away`. Custom DAX measures (winner, most unlucky team, luck colours and so on) are documented in **[documentation/Dax_measures.md](documentation/dax_measures.md)**.
 
`notebooks/analysis.ipynb` contains the first Plotly prototypes of the charts that were later rebuilt in Power BI.
 
---
 
## Dashboard
 
The Power BI report (`PowerBi/pl dashboard.pbix`) has two pages.
 
### Page 1: League overview
| Visual | What it shows |
|---|---|
| Cards | League winner, total goals, most unlucky team |
| Scatter chart | Actual vs expected points for every team (below the diagonal = unlucky) |
| Bar chart | Luck ranking, red for unlucky and green for lucky |
| Table | League table next to the xPTS table, with the rank gap highlighted |
| Slicer | Filter by luck label (Unlucky, As expected, Lucky) |
 
### Page 2: Team deep dive
Pick any team with the slicer.
 
| Visual | What it shows |
|---|---|
| Cards | Points, expected points and luck for the selected team |
| Form matrix | Result of every match, coloured by points won (3, 1 or 0) |
| Gauge | Points earned against the xPTS target |
| Combo chart | Goals scored (columns) against xG (line), match by match |
| Match scatter | xG vs xGA for every match, coloured by result. A loss with high xG and low xGA marks an "unlucky" match |
| Donut | Share of wins, draws and losses |
 
---
 
## Repository structure
 
```
PL tracker/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── collection.ipynb       # download and flatten the Understat data
│   ├── data_cleaning.ipynb    # clean, validate, engineer features, export CSVs
│   └── analysis.ipynb         # chart prototypes in Plotly
├── Data/
│   └── processed/
│       ├── epl_2025_matches.csv
│       ├── matches.csv
│       ├── season_table.csv
│       └── home_away.csv
├── PowerBi/
│   └── pl dashboard.pbix
├── documentation/
│   ├── data_dictionary.md
│   └── dax_measures.md
└── images/              
```
 
---
 
## How to run
 
**Requirements:** Python 3.10+ and Power BI Desktop on Windows.
 
```bash
# 1. Clone the repository
git clone https://github.com/LinaMahouachi/Premiere-League-Overview-Data-analysis.git
cd Premiere-League-Overview-Data-analysis
 
# 2. Create an environment and install the dependencies
python -m venv .venv
.venv\Scripts\activate         
pip install -r requirements.txt
 
# 3. Open the notebooks
jupyter notebook
```
 
Run the notebooks **in this order**:
 
1. `collection.ipynb` downloads the data and saves `epl_2025_matches.csv`.
2. `data_cleaning.ipynb` cleans it and writes `matches.csv`, `season_table.csv` and `home_away.csv`.
3. `analysis.ipynb` builds the chart prototypes. It reuses the `matches` table from the cleaning step, so run the cleaning notebook's cells first in the same session.
Then open `PowerBi/pl dashboard.pbix`. If Power BI cannot find the files, go to **Home, Transform data, Data source settings, Change source** and point each CSV to `Data/processed/`.
 

## Limitations
 
- **xG is a model.** Understat's methodology is not fully public, and the model may not suit every team or situation.
- **The model runs high this season.** League-wide xG (1,162.4) exceeds goals scored (1,045), so almost every team shows negative finishing. Read finishing and luck figures relative to other teams.
- **"Luck" is not only luck.** Elite finishers and goalkeepers can sustain over- or under-performance, so part of each gap is real skill.
- **Game state is ignored.** Teams that lead tend to sit back and concede chances, which xPTS does not account for.
- **One season is a small sample.** A single season's luck is mostly noise. Testing whether luck carries over from year to year would need more seasons.
- **Tactical metrics are proxies.** PPDA and deep completions describe pressing and territory only roughly. The data has no passing networks, formations or shot locations.
- **The thresholds are mine.** The +/-3 point cut-off for "lucky" and "unlucky" is a judgement call.
---
 
## Next steps
 
- Add more seasons and test whether luck regresses towards zero the following year.
- Predict second-half points from first-half xG and compare with predictions from points alone.
- Simulate each team's season from match xG to put a confidence range around every luck figure.
- Add a tactical page covering pressing, territory and style.
---
 
## Credits
 
- Data: [Understat](https://understat.com)
- Built with Python (`requests`, `pandas`, `numpy`, `plotly`), Jupyter and Power BI
- The data remains the property of Understat.
 
