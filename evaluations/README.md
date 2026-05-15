# 🎯 ハッカソン・アイデアソンの目的

LLM をエージェントの「判断主体」として組み込んだ **マルチエージェント・シミュレーション** の可能性を探究する。単発の LLM 応答ではなく、複数エージェントの相互作用から **集団的・創発的な行動** がどこまで現れうるかを、設計と実装の両面から検証することを狙いとする。

評価軸はその狙いを反映しており、以下の観点で成果物を見る。

- **創発を引き出す世界設計（A）** — 精密な世界ルール × エージェントへの「生データのみ」の入力により、行動戦略を完全に LLM の判断に委ねた構造になっているか
- **意味あるシナリオ（B）** — 社会科学・組織・市場ダイナミクスなど、現実世界の問題と接続したシナリオを扱っているか
- **研究プラットフォームとしての拡張性（C）** — 仮説検証や条件比較を重ねられる、モジュラーかつ設定駆動の設計になっているか
- **実装品質（D）** — LLM 統合・メモリ設計・コード品質・シミュレーション正確性が、再現可能な実験に耐える水準にあるか

## 📏 評価軸（各 0–10 点 / 合計 40 点）

評価基準の詳細は [`../.claude/skills/evaluate-submission/SKILL.md`](../.claude/skills/evaluate-submission/SKILL.md) を参照。各軸は 2 回スコアリングして平均（差が 3 以上ある場合のみ 3 回目を実施）。

### 🌱 A. 創発設計 (Emergent Design)
「世界ルールの設計精度」と「エージェント行動の自由度」の調和を評価。LLM がエージェントに与えるのは **生データとルールのみ**（例: `occupancy: 0.83, distance: 4.2`）で、行動指示（例: 「逃げろ」「警告せよ」）は含まないのが理想。両軸の harmonic balance で採点し、片方だけ高くても 5 点が上限。
- **9–10**: 精密な世界ルール × 生データのみのエージェント入力 → 最大の創発ポテンシャル
- **5–6**: ルールは明確だが行動誘導あり、または自由だが世界ルール不足
- **1–2**: 行動がスクリプト化され、LLM はテキスト整形役に過ぎない

### 🌍 B. 世界設定 (World Design)
シナリオの **独創性** と **現実問題との結びつき**（ビジネス・社会科学的意義）を評価。
- **9–10**: 高い独創性、または実世界の組織・市場・行動ダイナミクスを明確にモデル化
- **5–6**: 妥当だが既視感のあるシナリオ（標準的なクラウドシミュレーション等）
- **1–2**: 実質的なシナリオが無く、エージェントが空虚な空間を動くだけ

### 🚀 C. 発展性 (Extensibility & Future Potential)
**コード拡張性（5 点）** と **将来展望（5 点）** の 2 軸。
- コード拡張性: モジュール分離（エージェント/世界/LLM クライアント）、外部設定化（YAML/JSON）、新エージェント・新ルール追加の容易さ
- 将来展望: README に書かれた拡張計画の具体性・技術的妥当性・研究的深度
- **9–10**: モジュラー設計 × 中核の洞察を深める具体的な将来構想の両立
- **5–6**: 拡張性はあるが将来計画が「エージェントを増やす」程度に曖昧
- **1–2**: 拡張困難、将来方針なし

### 🛠️ D. 技術実装 (Technical Implementation)
4 つの下位次元の合算。
- **LLM 利用 (3 点)**: プロンプト設計（構造化出力・コンテキスト管理）、温度等のパラメータ選定、エラー処理
- **メモリ設計 (3 点)**: エージェントメモリの保存・更新・プロンプトへの注入。単なるログダンプではなく意味のある記述か
- **コード品質 (2 点)**: 可読性、構造、命名、関心の分離
- **シミュレーション正確性 (2 点)**: 同期ステップ実行、競合状態の不在、再現性（シード・設定）
- **9–10**: 洗練された LLM 統合 × 設計されたメモリ × 整ったコード × 正確なステップ機構
- **1–2**: LLM がほぼ統合されておらず、コードも追いづらい

## 🏆 ランキング（総合点 降順）

`20260511-1st-Hackathone-valuations/` 配下の `*_eval.md` を集計し、総合点（4 軸合計・満点 40 点）の降順で並べたもの。各セルの数値は評価スキルが複数回スコアリングし平均化した最終スコア。リファレンス実装（`reference_eval.md`）はランキング対象から除外している。

研究員番号・研究員名は [`../submissions/README.md`](../submissions/README.md) を参照。チーム参加の場合は複数名を併記。本名は公開していないため、研究員名（ハンドル）のみ表示する。提出リスト外（参考用と思われるローカルディレクトリ）は「—」で示す。

| 順位 | プロジェクト（評価結果 / GitHubリポジトリ） | 研究員番号 | 研究員名 | 総合点 | A. 創発設計 | B. 世界設定 | C. 発展性 | D. 技術実装 |
|------|--------------|-----------:|----------|--------|------------:|------------:|----------:|------------:|
| 1 | [lunar_agents](20260511-1st-Hackathone-valuations/lunar_agents_eval.md) / [🔗](https://github.com/sbilxxxx/lunar_agents) | 564 | Ben | **37.0** | 8.5 | 10.0 | 9.5 | 9.0 |
| 1 | [Project_Gaara](20260511-1st-Hackathone-valuations/Project_Gaara_eval.md) / [🔗](https://github.com/atomgreyfreeks/Project_Gaara) | 2171 | af10 | **37.0** | 10.0 | 9.0 | 9.0 | 9.0 |
| 3 | [near-future-ai-society-100-steps](20260511-1st-Hackathone-valuations/near-future-ai-society-100-steps_eval.md) / [🔗](https://github.com/ops324/near-future-ai-society-100-steps) | 1301 | 滝本哲也（もん） | **36.0** | 9.0 | 9.0 | 9.0 | 9.0 |
| 4 | [shibuya-sim](20260511-1st-Hackathone-valuations/shibuya-sim_eval.md) / [🔗](https://github.com/spacedream6090-svg/shibuya-sim) | 2021 / 146 | syota / Aji | **35.0** | 9.0 | 9.0 | 8.0 | 9.0 |
| 5 | [my-social-agents](20260511-1st-Hackathone-valuations/my-social-agents_eval.md) / [🔗](https://github.com/cygkichi/my-social-agents) | 2231 | ひろぽ | **34.5** | 7.5 | 10.0 | 8.0 | 9.0 |
| 6 | [good_echo_iss_sim_cursor_50s_no_accident](20260511-1st-Hackathone-valuations/good_echo_iss_sim_cursor_50s_no_accident_eval.md) / [🔗](https://github.com/Hop-Step-Jump/good_echo_iss_sim_cursor_50s_no_accident) | — | — (提出リスト外) | **34.0** | 8.0 | 9.0 | 9.0 | 8.0 |
| 6 | [hackathon_](20260511-1st-Hackathone-valuations/hackathon__eval.md) / [🔗](https://github.com/reireichel01/hackathon_) | 46 | RERE | **34.0** | 9.0 | 9.0 | 8.0 | 8.0 |
| 8 | [good_echo_iss_sim_cursor_100s_w_accident](20260511-1st-Hackathone-valuations/good_echo_iss_sim_cursor_100s_w_accident_eval.md) / [🔗](https://github.com/Hop-Step-Jump/good_echo_iss_sim_cursor_100s_w_accident) | — | — (提出リスト外) | **33.5** | 8.0 | 9.0 | 9.0 | 7.5 |
| 8 | [happy-to-chat-bench-simulation](20260511-1st-Hackathone-valuations/happy-to-chat-bench-simulation_eval.md) / [🔗](https://github.com/ops324/happy-to-chat-bench-simulation) | 1301 | 滝本哲也（もん） ※別レポ | **33.5** | 9.0 | 8.5 | 8.0 | 8.0 |
| 8 | [hackathon-singulab](20260511-1st-Hackathone-valuations/hackathon-singulab_eval.md) / [🔗](https://github.com/karesansui-u/hackathon-singulab) | 1882 | Karesansui | **33.5** | 8.0 | 9.0 | 8.5 | 8.0 |
| 11 | [2d-multi-places-simulation-on-fire-public](20260511-1st-Hackathone-valuations/2d-multi-places-simulation-on-fire-public_eval.md) / [🔗](https://github.com/suii00/2d-multi-places-simulation-on-fire-public) | 1954 | su | **33.0** | 9.0 | 7.0 | 8.0 | 9.0 |
| 11 | [hackathon-singulab-inu](20260511-1st-Hackathone-valuations/hackathon-singulab-inu_eval.md) / [🔗](https://github.com/karesansui-u/hackathon-singulab-inu) | 1882 | Karesansui ※別レポ | **33.0** | 7.0 | 9.0 | 9.0 | 8.0 |
| 13 | [AlberiaMisinformationProtest](20260511-1st-Hackathone-valuations/AlberiaMisinformationProtest_eval.md) / [🔗](https://github.com/mpcb1729/AlberiaMisinformationProtest) | 1300 | S.H | **32.5** | 7.5 | 9.0 | 7.0 | 9.0 |
| 14 | [automata-hackathon-2026-ebiyama](20260511-1st-Hackathone-valuations/automata-hackathon-2026-ebiyama_eval.md) / [🔗](https://github.com/yamachas0/automata-hackathon-2026-ebiyama) | 533 / 287 | やまちゃそ / えびねこ | **32.0** | 6.0 | 9.0 | 8.0 | 9.0 |
| 15 | [singulab](20260511-1st-Hackathone-valuations/singulab_eval.md) / [🔗](https://github.com/Kai-85-git/singulab) | 76 / 958 | Kai / タコ | **31.5** | 8.0 | 7.5 | 8.0 | 8.0 |
| 15 | [workplace-agent-simulation](20260511-1st-Hackathone-valuations/workplace-agent-simulation_eval.md) / [🔗](https://github.com/chuckyyyy2700/workplace-agent-simulation) | 50 | ちゃっきー | **31.5** | 8.0 | 9.0 | 6.5 | 8.0 |
| 17 | [lunar_simulation](20260511-1st-Hackathone-valuations/lunar_simulation_eval.md) / [🔗](https://github.com/kaii0624/lunar_simulation) | 2267 | kaii | **30.0** | 8.0 | 7.0 | 7.0 | 8.0 |
| 18 | [beyond-badminton](20260511-1st-Hackathone-valuations/beyond-badminton_eval.md) / [🔗](https://github.com/kohei-shimomura-aio/beyond-badminton) | 320 / 451 | シモムラ🏸\|梅虎 / TaE | **28.5** | 6.5 | 8.0 | 6.5 | 7.5 |
| 19 | [all-in-smoke](20260511-1st-Hackathone-valuations/all-in-smoke_eval.md) / [🔗](https://github.com/sfkmt/all-in-smoke) | 182 | えす | **28.0** | 5.0 | 8.0 | 7.0 | 8.0 |
| 20 | [kibo_crew_sim](20260511-1st-Hackathone-valuations/kibo_crew_sim_eval.md) / [🔗](https://github.com/hagaguragura/kibo_crew_sim) | 2142 | はぐら | **27.5** | 5.5 | 8.0 | 6.0 | 8.0 |
| 21 | [ai-homecare-simulation](20260511-1st-Hackathone-valuations/ai-homecare-simulation_eval.md) / [🔗](https://github.com/ytktci/ai-homecare-simulation) | 2095 | Rekishi | **25.5** | 4.5 | 8.0 | 6.0 | 7.0 |
| 21 | [singulabo-hackathon](20260511-1st-Hackathone-valuations/singulabo-hackathon_eval.md) / [🔗](https://github.com/Muburo/singulabo-hackathon) | 822 | 鶴子 | **25.5** | 3.5 | 9.0 | 6.0 | 7.0 |
| 23 | [2d-multi-places-simulation-on-fire](20260511-1st-Hackathone-valuations/2d-multi-places-simulation-on-fire_eval.md) / [🔗](https://github.com/shin-ai-inc/2d-multi-places-simulation-on-fire) | 52 | 柴田昌国 | **25.0** | 4.0 | 6.0 | 7.0 | 8.0 |
| 24 | [goodecho_r](20260511-1st-Hackathone-valuations/goodecho_r_eval.md) / [🔗](https://github.com/reireichel01/goodecho_r) | 46 | RERE ※別レポ | **24.5** | 5.5 | 8.0 | 6.0 | 5.0 |
| 25 | [mars-llm-simulation-wanko](20260511-1st-Hackathone-valuations/mars-llm-simulation-wanko_eval.md) / [🔗](https://github.com/AVACode123/mars-llm-simulation-wanko) | 1233 | Fumi@tsukuba | **23.5** | 5.0 | 7.0 | 5.5 | 6.0 |
| 25 | [risk-gap-llm-decision](20260511-1st-Hackathone-valuations/risk-gap-llm-decision_eval.md) / [🔗](https://github.com/takedashunsuke/risk-gap-llm-decision) | 1707 | たけし | **23.5** | 3.0 | 7.0 | 6.5 | 7.0 |
| 27 | [psychology-simulation](20260511-1st-Hackathone-valuations/psychology-simulation_eval.md) / [🔗](https://github.com/SakaiYuki-TA/psychology-simulation) | 1478 | 之 | **21.5** | 4.5 | 7.0 | 4.0 | 6.0 |
| 28 | [spring-park-simulation](20260511-1st-Hackathone-valuations/spring-park-simulation_eval.md) / [🔗](https://github.com/ichiko-trace/spring-park-simulation) | 1637 | だいすけ | **20.5** | 3.0 | 7.0 | 4.5 | 6.0 |
| 29 | [agi-job-simulator](20260511-1st-Hackathone-valuations/agi-job-simulator_eval.md) / [🔗](https://github.com/naoyayamamotodesu/agi-job-simulator) | 2007 | N | **19.5** | 2.0 | 6.5 | 5.0 | 6.0 |
| 30 | [meta-pop-agent](20260511-1st-Hackathone-valuations/meta-pop-agent_eval.md) / [🔗](https://github.com/npopstar/meta-pop-agent) | 828 | 沼澤拓也 | **7.0** | 1.0 | 4.0 | 2.0 | 0.0 |

## 🚫 未評価（ソースコード未添付）

以下の提出物はリポジトリにソースコードが含まれていないため、評価軸 A–D での採点が困難と判断し、ランキングから除外しています。

| プロジェクト（GitHub） | 研究員番号 | 研究員名 | リポジトリ内容 |
|------------------------|-----------:|----------|----------------|
| [echo-mortality-lab](https://github.com/Shilang46/echo-mortality-lab) | 1375 | フミマサ | README.md のみ |
| [fukushima-fire-sim](https://github.com/olivehouse20242024-arch/fukushima-fire-sim) | 2168 | 君枝 | LICENSE / README.md / 説明資料 (PDF) / デモ動画 (MP4) のみ |

## 📋 参考: リファレンス実装スコア

ランキング対象外ですが、参考までに併記します。

| プロジェクト | 総合点 | A. 創発設計 | B. 世界設定 | C. 発展性 | D. 技術実装 |
|--------------|--------|------------:|------------:|----------:|------------:|
| [reference](20260511-1st-Hackathone-valuations/reference_eval.md) | 30.5 | 8.5 | 7.0 | 7.0 | 8.0 |

## ⚖️ 同順位の扱い

総合点が同点の場合は同じ順位を割り当て、次の順位は同点者数の分だけスキップしています（例: 1位タイが2件あれば次は3位）。同点内の並び順は表示上の便宜的なものです。
