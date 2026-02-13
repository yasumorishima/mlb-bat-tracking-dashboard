# MLB Bat Tracking Dashboard — 開発メモ

最終更新: 2026-02-11（セッション8）

---

## 📁 ファイル構成

```
savant-extras/
├── streamlit_app.py          # メインアプリ（5タブ）
├── wbc_rosters.py            # WBC 2026 全20カ国ロスター（306人）
├── mlb_lineups_2025.py       # 2025 MLB Opening Day ラインナップ（全30チーム）★更新
├── requirements.txt          # 依存パッケージ
├── STREAMLIT_DEV_NOTE.md     # このファイル
├── savant_extras/            # ライブラリ本体（PyPI公開済み）
│   └── bat_tracking.py       # bat_tracking(), bat_tracking_monthly(), bat_tracking_splits()
└── tests/                    # 23件テスト（全pass）
```

---

## 🚀 起動方法

```bash
python -m streamlit run C:\Users\fw_ya\Desktop\Claude_code\savant-extras\streamlit_app.py
```

---

## 📊 現在のタブ構成（5タブ）

### Tab 1: 🏆 Leaderboard
- サイドバーでシーズン・Player type・利き腕・WBC国・MLBチームフィルタ
- メトリクス選択（Bat Speed / Attack Angle / Ideal AA% / Swing Tilt）
- 上位N人の横棒グラフ（リーグ平均点線付き）
- フィルタバッジ表示（有効フィルタを画面上部に）

### Tab 2: 👤 Player Comparison
- 最大6選手を multiselect で選択（ドロップダウン形式）
- レーダーチャート（4指標、リーグ全体でnormalize）
- 4指標の棒グラフ横並び（灰色点線=リーグ平均）

### Tab 3: 🌍 WBC Country Strength
- **Auto mode**: 全20カ国の在籍選手データを自動集計してスコア化
- **Manual mode**: 国を選んで最大9人の打者を手動ピック → スコアがリアルタイム更新
- Overall Strength Ranking 棒グラフ（バッティング+ピッチングの複合スコア）
- Batting vs Pitching 散布図（各国を2軸でプロット）
- Top 6カ国のレーダーチャート（Batting）
- Full Score Table（折りたたみ）

**スコア算出方法:**
- Batting Score: avg_bat_speed + ideal_attack_angle_rate + attack_angle を正規化平均（0-100）
- Pitching Score: ERA（逆）+ FIP（逆）+ K/9 + BB/9（逆）を正規化平均（0-100）
- Overall Score: Batting + Pitching の平均

**WBCの注意点:**
- Bat trackingデータは2024シーズン以降のみ（Hawk-Eye）
- MLB選手が少ない国（日本9人、韓国7人、オーストラリア5人、ブラジル5人、チェコ0人）は暫定スコア
- ピッチングスコアはpybaseball batting_stats/pitching_statsからLast,Firstフォーマットでマッチング（精度80〜85%）
- 今後の改善案: WBC専用データベース作成（非MLB選手含む）

### Tab 4: ⚾ Team Lineup Builder ← NEW
**2モード:**

#### 🔧 Build single team lineup
- チーム選択 → 9ポジションスロット（C / 1B / 2B / SS / 3B / LF / CF / RF / DH）
- 各スロットはそのチームの選手をドロップダウンで選択
- **ポジション厳密マッチ不要** — どのスロットにも誰でも置ける（ファン向け自由度）
- デフォルト配置: `MLB_LINEUPS_2025` のハードコードを優先（全30チーム対応）、未マッチはwRC+順にフォールバック
- 4指標（Bat Speed / Attack Angle / Ideal AA% / Swing Tilt）の横棒グラフ
- ラインナップ平均 vs リーグ平均の差分をメトリクスカードで表示

#### 📊 Compare all teams (auto top-9)
- 各チームのwRC+上位9選手を自動選択
- メトリクス選択で全30チームを横棒グラフでランキング
- Composite Batting Score（0-100）でも全チームランキング

**デフォルトデータ取得:**
- `batting_stats(year, qual=10)` でチーム所属と wRC+ を取得
- `bat_tracking()` とプレイヤー名で結合（"Last, First" ↔ "First Last" 変換）
- マッチング率: 約85%（特殊文字含む選手名で一部未マッチ）

### Tab 5: 📅 Monthly Trend
- 「Load Monthly Data」ボタン（約30秒かかる）
- リーグ平均を灰色ベースラインで表示
- 最大6選手をドロップダウンで重ねてプロット
- WBCフィルタが有効な場合、その国の選手がデフォルトで自動選択

---

## 🔧 データソース・取得方法

| データ | 取得方法 | キャッシュ |
|---|---|---|
| Bat tracking | `savant_extras.bat_tracking()` → Baseball Savant CSV | `@st.cache_data` |
| チーム情報 | `pybaseball.batting_stats()` + `pitching_stats()` で名前結合 | `@st.cache_data` |
| ピッチング stats | `pybaseball.pitching_stats(year, qual=20)` | `@st.cache_data` |
| WBC ロスター | `wbc_rosters.py` にハードコード（2026-02-05公式発表） | — |
| MLB Opening Dayラインナップ | `mlb_lineups_2025.py` にハードコード（全30チーム）。出典: MLBレギュラー.txt | — |

---

## 🐛 既知の問題・制限

1. **名前マッチング精度**: bat_trackingは "Last, First" 形式、pybaseballは "First Last" 形式。
   変換して結合しているが特殊文字（アクセント等）含む選手で一部未マッチ（約15%）
2. **ポジション情報なし**: pybaseball の `Pos` 列は守備得点（数値）でポジション名ではない。
   `batting_stats_bref` はスクレイピングエラーで取得不可。現在はwRC+順でデフォルト配置
3. **2025データ**: 2025シーズンは途中経過のみ（2026-02-11時点）。年選択で切り替え可
4. **WBC非MLB選手**: bat trackingはMLBデータなのでNPBなど海外リーグの選手はスコアに未反映

---

## ✅ 完了済み（セッション3〜6）

1. ✅ **mlb_lineups_2025.py を全30チームに更新** — 出典: `C:\Users\fw_ya\Desktop\MLBレギュラー.txt`（Opening Day 2025）
   - 前回の疑惑データを修正: Mullins重複解消（BALのみ）、NYY→Bellinger/Cabrera、HOU→AltuveがLF、LAD→H.Kim at 2B
   - 旧MISSING_TEAMS 12チームも全て追加（ARI/CIN/COL/CWS/LAA/MIA/MIN/OAK/PIT/SFG/STL/WSN）
2. ✅ **Tab4に組み込み完了** — `MLB_LINEUPS_2025` を import、各ポジションのデフォルトをハードコード優先に変更。bat trackingデータにない場合のみwRC+順フォールバック
3. ✅ **日英2言語対応** — サイドバー最上部に `🇺🇸 English / 🇯🇵 日本語` トグル追加。翻訳辞書 `T["en"]`/`T["ja"]` で全UIテキスト管理（約50項目）
4. ✅ **難しい用語の補足説明追加** — 日英両対応。各タブに `📖 Metric Guide / 指標解説` expander を追加
   - Tab1・Tab2・Tab4: Bat Speed / Attack Angle / Ideal AA% / Swing Tilt（＋Tab4はwRC+）
   - Tab3: 上記打撃指標 + ERA / FIP / K/9 / BB/9（Batting/Pitchingセクション分け）
5. ✅ **NameError修正** — データ未読み込み時 `df_raw` が未定義だったバグ修正
6. ✅ **グラフ全面改善（セッション6）** — 数値ラベル・グリッド・軸範囲を実データに合わせる
7. ✅ **リーグ平均線をオレンジ色・太め** — `AVG_COLOR = "#e67e22"` で全グラフ統一
8. ✅ **日本語フォント** — `japanize-matplotlib` 導入（requirements.txt 追加済み）
9. ✅ **WBC国名文字化け修正** — matplotlibグラフ内の旗絵文字を除去（`country_label` 列・タイトルも修正）
10. ✅ **初回アクセス画面改善** — 5タブ機能をカード形式で表示（Monthly Trendを緑で目立たせ）
11. ✅ **ロードボタン上部移動** — シーズン・選手タイプ直後に配置
12. ✅ **リセットボタン追加** — `🔄 Reset / リセット`（全 session_state クリア）
13. ✅ **アプリガイド追加** — expander形式（データ読み込み後は折りたたみ）
14. ✅ **ダークモード対応** — `.streamlit/config.toml` でカスタムテーマ設定
15. ✅ **Sliderバグ修正** — フィルタ後選手数5未満でクラッシュする問題を修正

---

## 🔜 次回やること（優先度順）

### 高優先度（「続き」で着手）

1. **【要修正】japanize_matplotlib エラー（Streamlit Cloud）**
   - エラー: `from distutils.version import LooseVersion` → Python 3.13 で `distutils` 削除済み
   - `runtime.txt`（python-3.11）を追加したが Streamlit Cloud が Python 3.13 のまま → 効いていない
   - 解決策候補:
     - ① `japanize-matplotlib` を `matplotlib-fontja` に差し替え（Python 3.12+ 対応の後継パッケージ）
       - `requirements.txt`: `japanize-matplotlib` → `matplotlib-fontja`
       - `streamlit_app.py`: `import japanize_matplotlib` → `import matplotlib_fontja`
     - ② `runtime.txt` を削除して `.python-version` ファイルで指定（Streamlit Cloud の仕様変更対応）
   - **次回まず①を試す**

2. **ブログ記事執筆**（日本語 + 英語）
   - 内容: savant-extras の使い方 + Streamlit でダッシュボードを作った話
   - トーン: 「非エンジニアがやってみた」系。断定を避け「参考程度に」「誤りがある可能性あり」を明記
   - 注意書き必須: 名前マッチング精度85%・MLB所属選手のみ・暫定スコアあり・2024年以降のHawk-Eyeデータのみ
   - 媒体: Zenn / Qiita / DEV.to（英語版）

2. **アプリURL確定後**: savant-extras README と Kaggle Dataset ページにリンク追加

3. **カードクリックで該当タブにジャンプ**（難易度高め、後回し可）

### 完了済み（セッション8）
- ✅ **Streamlit Community Cloud デプロイ**: GitHub push → share.streamlit.io でデプロイ完了
- ✅ **runtime.txt 追加**: Python 3.11 固定（japanize_matplotlib が Python 3.13 の distutils 削除で動かない問題を解決）
- ✅ **投手モード対応**: Tab1/Tab2 で ERA/FIP/K9/BB9 に切り替え。Tab5 は batter 専用メッセージ
- ✅ **WBC/Team タブを df_batter 固定**: player_type 設定に依存しない安定したスコア計算
- ✅ **スコア一覧をWBCタブ先頭に移動**（デフォルト展開）: 打者数・投手数・打者名を一番上で確認可能
- ✅ **WBC総合ランキンググラフ削除**: 投球スコアなし国が混在して信頼性低いため
- ✅ **投手リーダーボード**: BB/9 をデフォルト指標に設定、20IP基準の説明追加
- ✅ **WBCキャプション改善**: 打者データ固定の説明 + MLB選手少ない国は暫定値の注意書き
- ✅ **WBC スコア一覧に列追加**: 投手数（n_pitchers）・打者名一覧（batter_names）を追加。1人しかいない国でも誰か即確認できる

### 完了済み（セッション7）
- ✅ タブ名・グラフ軸ラベル・タイトル全て日本語対応
- ✅ サイドバー `🌙` ダーク/ライトトグル（Languageと横並び）
- ✅ ダークモードCSS強化（テキスト・ボタン・multiselect全対応）
- ✅ 初回カードUI改善（ボーダー付きカード）
- ✅ グラフ直下に色凡例・指標補足caption（全タブ）
- ✅ WBC: 打球・投球詳細テーブル追加
- ✅ WBC: 打撃・投球レーダー横並び + 国フィルタ
- ✅ Tab4: 全チーム比較をデフォルト表示
- ✅ Tab4: 投球スコア・総合スコアグラフ・散布図・テーブル追加
- ✅ Tab4: チームフィルタ付き打撃・投球レーダー
- ✅ チーム略称→正式名マッピング（MLB_TEAM_NAMES全30チーム）
- ✅ Tab2: multiselect デフォルト空＋placeholder追加

### 中優先度
5. **Streamlit Community Cloud デプロイ**
   - 動作確認完了後、savant-extras リポジトリを GitHub に push → share.streamlit.io でデプロイ
   - 手順: https://share.streamlit.io → GitHub連携 → `streamlit_app.py` を指定
   - `requirements.txt` には pybaseball・japanize-matplotlib 追加済みなので追加作業不要
6. **WBC専用データベース** — 非MLB選手のデータも含めた国別比較

### 低優先度
7. **国際大会向け拡張** — WBC以外（プレミア12など）にも対応できる構造

---

## 📝 wbc_rosters.py について

- 出典: Baseball America（2026-02-05公式発表）
- 全20カ国 × MLB所属選手のみ（306人）
- 名前フォーマット: `"Last, First"` — bat_trackingデータの `name` 列と合わせるため
- チェコのみMLB所属選手0名でエントリーなし
- 日本選手の英語名: `"Ohtani, Shohei"` 形式で収録（日本語名は除外）

---

## 💡 将来的なアイデア（ユーザー発案）

- WBC専用のデータベース（非MLB選手含む国別強さ比較）
- Streamlit Community Cloudで公開 → savant-extrasのデモアプリとして宣伝
- KaggleのDatasetページからStreamlitへリンク → upvote獲得の導線
- 他の国際大会（プレミア12など）への拡張
- MLB分析シリーズ（ダルビッシュ・今永・千賀・大谷など）の可視化も同一アプリに統合予定

---

## 🔑 環境メモ

```bash
# ローカル起動
python -m streamlit run C:\Users\fw_ya\Desktop\Claude_code\savant-extras\streamlit_app.py

# pybaseball インストール済み確認
pip show pybaseball  # version 2.x

# savant-extras（PyPI公開済み）
pip show savant-extras  # version 0.1.0
```
