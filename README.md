# MLB Bat Tracking Dashboard

2024年からMLBが導入した **Bat Tracking（バットトラッキング）** データをインタラクティブに可視化する Streamlit ダッシュボード。
Hawk-Eye 光学システムで計測される「バットスピード・スイング角度・アタックアングル」などの指標を探索できます。

**Live Demo:** [https://yasumorishima-mlb-bat-tracking.streamlit.app/](https://yasumorishima-mlb-bat-tracking.streamlit.app/)

English / 日本語 対応 | モバイル対応

![Dashboard Screenshot](assets/streamlit-dashboard-top.png)

## 主な機能（5タブ構成）

| タブ | 目的 | 主な操作 |
|---|---|---|
| **Leaderboard** | バットスピード等の指標でリーグ全体を一覧 | シーズン・利き手・WBC国・MLBチームでフィルタ、TOP N 棒グラフ（リーグ平均ライン付き） |
| **Player Comparison** | 最大6選手を同時比較 | レーダーチャート + 指標別棒グラフを横並びで表示 |
| **WBC Country Strength** | WBC 2026 参加国の打撃・投球総合力を比較 | 自動計算（データ平均）または手動スコア入力、散布図・レーダーチャート |
| **Team Lineup Builder** | 9ポジションの打線を編成して全30球団と比較 | 選手を9枠に配置 → チーム総合スコアで順位付け |
| **Monthly Trend** | 月別のバットトラッキング推移を追跡 | 複数選手を重ね表示してシーズン通じた変化を確認 |

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
