# CLAUDE.md — mlb-bat-tracking-dashboard

Streamlit dashboard (English/Japanese) for MLB bat-tracking metrics: leaderboard, player comparison, WBC 2026 country strength, lineup builder, monthly trend. Live: https://yasumorishima-mlb-bat-tracking.streamlit.app/ . **Dormant**: no work is scheduled.

## Session protocol (cloud)
1. Read this file first. There is nothing scheduled: act only on what the owner asks in this session.
2. Before ending, add a dated line to "Progress log" (and update "Next tasks" if anything changed), then open a PR to `master` from the session branch and merge it yourself; report the merged commit.
3. No `gh` CLI in cloud (use the GitHub MCP tools); `workflow_dispatch` returns 403 from cloud.
4. Reply to the owner in Japanese (です/ます). Do not claim anything you did not run in this session; say where each number was measured.
5. **After a merge that changes a module the app imports** (`wbc_rosters.py`, `mlb_lineups_2025.py`) or `requirements.txt`, tell the owner: 「share.streamlit.io で yasumorishima-mlb-bat-tracking のアプリを Reboot してください」. Streamlit Community Cloud's automatic update re-runs only `streamlit_app.py` and can keep old modules loaded (this left the `mlb-win-probability` app on an `ImportError` screen on 2026-10-16). Cloud cannot press Reboot, so the live app is not verified until the owner has rebooted and looked at it.

## Progress log
- 2026-02-13 – 2026-03-04: app built; data loads on first visit; WBC 2026 rosters updated for the 1 March roster changes.
- 2026-03 – 2026-04-19: a Grafana/BigQuery section was added to README and removed again (`26861cd`, last commit).
- 2026-11-01: this file written for cloud. Nothing run. The app's health endpoint `/~/+/_stcore/health` answered 200 (checked from the owner's PC); that shows the server is up, not that the page renders without an error.

## What this is
`streamlit_app.py` (single file, ~1,600 lines) loads bat tracking with `savant_extras.bat_tracking` / `bat_tracking_monthly` / `bat_tracking_splits` (Baseball Savant), team names and pitching stats with `pybaseball.batting_stats` / `pitching_stats`, and static rosters from `wbc_rosters.py` and `mlb_lineups_2025.py`. Seasons offered: 2025, 2024. Python 3.11 on Streamlit Cloud (`runtime.txt`). The app is kept awake by the 6-hourly keepalive in the `wbc-scouting` repo (`streamlit_apps.txt` lists this URL).

## Non-negotiable discipline
1. Keep the live URL and the app path (`streamlit_app.py` on `master`) unchanged: the deployment is bound to them.
2. Show the data source in the app and README (Baseball Savant / Hawk-Eye).
3. Errors must not be hidden as empty results when you change a loader; prefer a visible message.

## Next tasks
None scheduled; only act when the owner asks. Candidates noticed on 2026-11-01 (not verified on the live page):
- `load_team_lookup` and `load_pitching_data` call FanGraphs through pybaseball and return `{}` / an empty table on any exception. FanGraphs has refused requests from cloud/datacenter hosts (owner's notes, measured from GitHub Actions); if the same happens on Streamlit Cloud, team filters and pitching scores are silently empty. MLB Stats API (`stats=sabermetrics`) is the replacement used in `mlb-data-pipeline`.
- 2026 is not selectable (`st.selectbox(..., [2025, 2024])`).
- README says "License: MIT" but there is no `LICENSE` file (GitHub detects none).
- `docs/images/grafana-preview.png` is no longer referenced after the Grafana section was removed.
- `requirements.txt` is unpinned (`savant-extras>=0.1.0`), so the next reboot installs the latest savant-extras; untested.
- No CI workflow (only GitHub's dependency graph).

## How to run (cloud, not yet tried there)
```bash
pip install -r requirements.txt
python -c "from streamlit.testing.v1 import AppTest; at = AppTest.from_file('streamlit_app.py', default_timeout=300).run(); print(at.exception)"
```
Needs `baseballsavant.mlb.com` (and `www.fangraphs.com` for pybaseball's stats). An empty `at.exception` means the script ran; it does not prove the live deployment works.

## Credits (keep in README)
Baseball Savant (Hawk-Eye, MLB Advanced Media); savant-extras; pybaseball; Streamlit; matplotlib + matplotlib-fontja.
