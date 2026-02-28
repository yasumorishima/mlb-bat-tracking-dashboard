# MLB Bat Tracking Dashboard

A Streamlit dashboard for exploring MLB bat tracking data (Hawk-Eye, 2024 season onward). English/Japanese bilingual. Mobile-friendly.

**Live Demo:** [https://yasumorishima-mlb-bat-tracking.streamlit.app/](https://yasumorishima-mlb-bat-tracking.streamlit.app/)

![Dashboard Screenshot](assets/streamlit-dashboard-top.png)

## Tabs

| Tab | Description |
|---|---|
| Leaderboard | Filter by season, player type, handedness, WBC country, MLB team. Top-N bar chart with league average line. |
| Player Comparison | Compare up to 6 players with radar chart and side-by-side bar charts. |
| WBC Country Strength | Auto/Manual mode for WBC 2026 country batting & pitching scores. Scatter plot, radar charts, full score table. |
| Team Lineup Builder | Build single team lineup (9 positions) or compare all 30 teams. Composite batting/pitching scores. |
| Monthly Trend | Month-by-month bat tracking trends with multi-player overlay. |

## Tech Stack

- [Streamlit](https://streamlit.io/)
- [savant-extras](https://github.com/yasumorishima/savant-extras) — bat tracking data retrieval with date range support
- [pybaseball](https://github.com/jldbc/pybaseball) — batting/pitching stats
- [matplotlib](https://matplotlib.org/) + [matplotlib-fontja](https://pypi.org/project/matplotlib-fontja/) — charts with Japanese font support
- Data source: [Baseball Savant](https://baseballsavant.mlb.com/) (Hawk-Eye)

## Local Setup

```bash
pip install -r requirements.txt
streamlit run streamlit_app.py
```

## License

MIT
