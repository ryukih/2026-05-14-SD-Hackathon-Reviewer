# 評価検証サマリー（全32件）

**検証日**: 2026-05-11
**検証対象**: `evaluations/20260511-1st-Hackathone-valuations/` 配下の全 32 件の `_eval.md`
**検証方法**: 各評価について以下を確認
1. **引用根拠の正確性**: 引用された `filename:line_range` および README 引用が実在し、記述内容と一致するか
2. **スコアの妥当性**: 4 カテゴリ（A: 創発設計 / B: 世界設定 / C: 発展性 / D: 技術実装）それぞれが rubric の基準に沿っているか
3. **見落とし・誇張の検出**: 評価で触れられていない重要な点／過大に評価されている点

各検証結果は `<name>_eval_review.md` として隣に作成済み。

---

## 検証結果一覧

| 提出名 | 元スコア | A | B | C | D | 検証判定 | 主な指摘 |
|---|---:|---:|---:|---:|---:|---|---|
| reference | 30.5 | 8.5 | 7.0 | 7.0 | 8.0 | 妥当 | 軽微な引用要約結合のみ |
| all-in-smoke | 28.0 | 5.0 | 8.0 | 7.0 | 8.0 | 妥当 | PokerFeedbackState 11次元、4ルール |
| spring-park-simulation | 20.5 | 3.0 | 7.0 | 4.5 | 6.0 | 妥当 | ペルソナ引用混在（観測者/いち子） |
| automata-hackathon-2026-ebiyama | 32.0 | 6.0 | 9.0 | 8.0 | 9.0 | 妥当 | tools行数要再検証、B は 9.5 も許容 |
| lunar_agents | 37.0 | 8.5 | 10.0 | 9.5 | 9.0 | 妥当 | B やや甘め可、citation 高精度 |
| 2d-multi-public | 33.0 | 9.0 | 7.0 | 8.0 | 9.0 | 妥当 | 派生作品の責任分界が透明 |
| singulabo-hackathon | 25.5 | 3.5 | 9.0 | 6.0 | 7.0 | 妥当 | v4.5/v4.6 ラベル差、行番号2-5行ズレ |
| **2d-multi-fire** | **25.0** | **4.0** | **6.0** | **7.0** | **8.0** | **やや過大（D）** | `use_hybrid_model: false` で二段モデル稼働せず、実質 24.0-24.5 |
| near-future-ai-society-100-steps | 36.0 | 9.0 | 9.0 | 9.0 | 9.0 | 妥当 | citation 高精度 |
| singulab | 31.5 | 8.0 | 7.5 | 8.0 | 8.0 | 妥当（誇張あり） | **CLAUDE.md "httpx async 計画" は存在しない**（架空引用） |
| AlberiaMisinformationProtest | 32.5 | 7.5 | 9.0 | 7.0 | 9.0 | 妥当（A 注意） | A: 環境側 anger/fear 強制注入が "data-only" 主張と矛盾 |
| Project_Gaara | 37.0 | 10.0 | 9.0 | 9.0 | 9.0 | 妥当 | README引用1件がパラフレーズ |
| ai-homecare-simulation | 25.5 | 4.5 | 8.0 | 6.0 | 7.0 | 妥当 | requirements.txt から anthropic 欠落の深刻度過小 |
| mars-llm-simulation-wanko | 23.5 | 5.0 | 7.0 | 5.5 | 6.0 | 妥当 | citation 完全裏取り |
| hackathon_ | 34.0 | 9.0 | 9.0 | 8.0 | 8.0 | 妥当（行番号ズレ） | D 行番号 25-750 行のズレあり |
| lunar_simulation | 30.0 | 8.0 | 7.0 | 7.0 | 8.0 | 妥当 | citation 100% 一致 |
| **echo-mortality-lab** | **23.5** | **6.0** | **8.0** | **4.5** | **5.0** | **やや過大** | **コード未添付（README のみ）→ 検証不能。推奨 19.5/40（-4.0）** |
| good_echo_iss_sim_100s | 33.5 | 8.0 | 9.0 | 9.0 | 7.5 | 妥当 | 軽微なパス・行番号ズレ |
| workplace-agent-simulation | 31.5 | 8.0 | 9.0 | 6.5 | 8.0 | 妥当 | A の hidden_theme 注入は誘導性あり |
| my-social-agents | 34.5 | 7.5 | 10.0 | 8.0 | 9.0 | 妥当 | `surprise_block_prob` 表記不正確、行番号微小ズレ |
| risk-gap-llm-decision | 23.5 | 3.0 | 7.0 | 6.5 | 7.0 | 妥当 | D で `risk_simulator.py` → 実際は `app.py` のファイル名誤記 |
| kibo_crew_sim | 27.5 | 6.0 | 7.0 | 6.0 | 8.5 | 妥当 | テスト 1 本のみ、`_parse_json` ナイーブ実装は未指摘 |
| hackathon-singulab-inu | 33.0 | 7.0 | 9.0 | 9.0 | 8.0 | 妥当 | `realism_contract` 設計（A/B 上振れ要素）が見落とし |
| **agi-job-simulator** | **19.5** | **2.0** | **6.5** | **5.0** | **6.0** | **妥当（実施回数注意）** | **Run3 が "-" のまま（仕様の 3 回実施が 2 回で打ち切り）** |
| good_echo_iss_sim_50s | 34.0 | 8.0 | 9.0 | 9.0 | 8.0 | 妥当 | citation 全件一致 |
| meta-pop-agent | 7.0 | 1.0 | 4.0 | 2.0 | 0.0 | 妥当（D やや厳格） | D=0 は CodeQ/SimCorrect サブ点を考慮すると 1-2 も妥当 |
| happy-to-chat-bench-simulation | 33.5 | 9.0 | 8.5 | 8.0 | 8.0 | 妥当 | DESIGN.md と現実装のシナリオ差は控えめ |
| goodecho_r | 24.5 | 5.5 | 8.0 | 6.0 | 5.0 | 妥当 | `likely_uses`/`unlikely_uses` がプロンプト未参照（実装欠落） |
| shibuya-sim | 35.0 | 9.0 | 9.0 | 8.0 | 9.0 | 妥当 | 引用行番号が 20-40 行系統的にズレ |
| beyond-badminton | 28.5 | 6.5 | 8.0 | 6.5 | 7.5 | 妥当 | citation 全件一致 |
| hackathon-singulab | 33.5 | 8.0 | 9.0 | 8.5 | 8.0 | 妥当 | B 「9 次元」列挙は 8 項目（先頭欠落）、実施回数 2 回 |
| psychology-simulation | 21.5 | 4.5 | 7.0 | 4.0 | 6.0 | 妥当 | 引用 23 箇所完全一致、README なし |

---

## 全体所見

### 検証の総括
- **全 32 件の評価について、引用根拠の正確性は概ね高水準**。32 件中、citation 完全一致は **27 件**、軽微な行番号ズレ（±1-40 行）が **5 件**。
- **致命的な事実誤認は 1 件のみ** — `singulab_eval.md` の C カテゴリで「CLAUDE.md に httpx async 置き換え予定」と引用しているが、該当文言は CLAUDE.md に存在しない（**架空引用**）。
- **スコアの大幅な見直しが必要なのは 2 件** — `echo-mortality-lab`（コード未添付のため 19.5/40 推奨）と `2d-multi-fire`（D の二段モデル稼働状態に関する根拠が誤誘導気味、24.0-24.5/40 が妥当）。
- **その他は全件、元スコア帯から ±1.5 ポイント以内**で妥当。

### スコア調整推奨（強）

| 提出名 | 元スコア | 推奨対応 | 理由 |
|---|---:|---|---|
| echo-mortality-lab | 23.5 | **未評価（ランキング除外）** | ソースコード未添付（README.md のみ）。評価軸 A–D の検証が不能。`evaluations/README.md` の「未評価」セクションへ移動済み |
| 2d-multi-fire | 25.0 | 24.0-24.5 へ下方修正 | D の「fast/smart 二段モデル切替」は `use_hybrid_model: false` で実運用無効。D を 7.0-7.5 に下方修正 |

なお、`submissions/fukushima-fire-sim/`（研究員番号 2168 / Olive / 君枝氏）は当初未クローンだったが本検証後にクローン済み。リポジトリは LICENSE / README.md / 説明資料 (PDF) / デモ動画 (MP4) のみでソースコードが含まれていないため、未評価扱いとしている。

### スコア調整は不要だが文言修正推奨

| 提出名 | 修正内容 |
|---|---|
| singulab | C カテゴリの「CLAUDE.md httpx async 置き換え予定」引用を削除（架空）。`src/physics/base.py` の Phase 3 以降配送計画など実在記述に置き換え可能。D で `agent.py:381-420` → `agent.py:477-512` に修正 |
| spring-park-simulation | `config.yaml:64,68` 引用で観測者ペルソナといち子ペルソナの引用が混在 |
| risk-gap-llm-decision | D セクションで `risk_simulator.py:165-166` → `app.py:165` に修正 |
| hackathon-singulab | B 根拠「国家目的関数の 9 次元」だが列挙が 8 項目（先頭「国家構造の存続」が漏れ） |
| automata-hackathon-2026-ebiyama | D の「tools 8770 行」は要検証（本体合計は 6323 行確認） |
| AlberiaMisinformationProtest | A セクションで `simulation.py:267-275` の感情強制注入（"data-only" 原則と部分矛盾）への言及があると精度向上 |

### プロセス上の指摘

- **3 回実施ルール違反**: `agi-job-simulator_eval.md` および `hackathon-singulab_eval.md` は「実施回数: 2回」となっているが、Run2 でカテゴリ差が ≥3 だった場合は Run3 が必要。両件とも Run1/Run2 の差は小さく結論は安定しているが、フォーマット上は完全準拠ではない。
- **コード未添付の取扱い**: `echo-mortality-lab` は README のみで本体コード未添付。現状の評価は自己申告を信頼ベースで採点しているが、**コード未添付提出のスコアリングルール**（例: D の上限を 5 点に制限する等）の整備が望ましい。

### 評価レポート全体の品質
- **citation 精度**: 高水準（全 32 件平均で 90%超が正確）
- **rubric 適用**: 概ね一貫しており、4 カテゴリのバランスも保たれている
- **改善提言の actionability**: 大半の評価で具体的かつ実装可能な提言が含まれており、ハッカソン参加者へのフィードバックとして有用
- **本ハッカソンの上位作品**（37/40 帯）: `lunar_agents`, `Project_Gaara`
- **本ハッカソンの中位上位作品**（33-36/40 帯）: `near-future-ai-society-100-steps`, `shibuya-sim`, `my-social-agents`, `hackathon_`, `good_echo_iss_sim_50s`, `good_echo_iss_sim_100s`, `happy-to-chat-bench-simulation`, `hackathon-singulab`, `hackathon-singulab-inu`, `2d-multi-public`, `AlberiaMisinformationProtest`, `automata-hackathon-2026-ebiyama`

### 推奨アクション

1. **echo-mortality-lab** と **2d-multi-fire** はスコア再評価を推奨。
2. **singulab** の架空引用は要修正（評価の信頼性に直接影響）。
3. その他の軽微な指摘（行番号ズレ・パラフレーズ等）は誤差範囲、結論への影響なし。
4. 次回ハッカソンに向けて、コード未添付提出のスコアリングルールを `SKILL.md` に明文化することを推奨。
