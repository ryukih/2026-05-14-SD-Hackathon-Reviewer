# 評価検証レポート: my-social-agents

**検証日**: 2026-05-11
**対象**: `my-social-agents_eval.md`
**元評価合計**: 34.5/40

## 検証サマリー

| カテゴリ | 元スコア | 検証判定 | 推奨スコア帯 | 主な所見 |
|---|---|---|---|---|
| A. 創発設計 | 7.5/10 | おおむね妥当 | 7-8 | 規範的プロンプトと Maslow ハードゲートは事実。「奇襲時 block_prob=0」は実コードでは `surprise_block_prob` という設定可変 |
| B. 世界設定 | 10.0/10 | 妥当 | 9-10 | Hobbes/Jervis/Malthus を量的検証する独創性は明確。「7倍ジャンプ」は REPORT、README は「5倍」と表記揺れあり |
| C. 発展性 | 8.0/10 | 妥当 | 8-8.5 | 行数 (2016/1350)、scenario preset、sweep/analysis 分離はすべて実コードと一致 |
| D. 技術実装 | 9.0/10 | 妥当（やや甘め） | 8.5-9.0 | LLM/メモリ部分は精密に裏取り済み。「tests 未確認」は事実（旧 v1/v2 にのみ存在） |

**総合判定**: 元評価は citation 正確性が極めて高く、コード行番号・関数名・出力値が大半そのまま再現できる。スコア配分も rubric に整合的。34.5/40 は妥当。

## カテゴリ別検証

### A. 創発設計（元: 7.5/10）

**根拠の検証**:
- ✓ `PLANNER_PROMPT (poc_world.py:207-446)` — 207行で `"""\` 開始、446行付近で閉じる。「★ 基本姿勢」「★ 攻撃の倫理コスト」「★ 平和的代替手段」が文字通り存在
- ✓ `Maslow Lv1-3 hard gate (:323-360)` — 「未充足のレベルがあれば、それより上位は選択不可」「★禁止: agent / weapon」明示
- ✓ `_hp_threshold / _attack_for_food_policy (poc_12agents.py:945-958)` — N → 150/200/250、A → 可/緊急時/不可、離散3段階一致
- ✓ `_expected_hits / _kill_probability (:67-112)` — `cap = W // 2` 防御天井実装一致
- ⚠ `_resolve_hit 奇襲ペナルティ (:1101-1163)` — 実コードは `surprise_block_prob` という別変数（デフォルト 0 だが設定可能）。「奇襲時 block_prob=0」は実質正しいが厳密ではない
- ✓ `FUTURE_WORK.md` 自己批判 — III-1 / II-1 (b) に該当文言実在

**スコア妥当性**: 7.5 は妥当。rules 軸 9-10 / freedom 軸 6 → 平均 7-8 が rubric 整合的。

**見落とし・誇張**: PLANNER_PROMPT 「約 250 行」は実 240 行弱、軽微。`surprise_block_prob` 表記の不正確。

**検証結論**: 妥当

### B. 世界設定（元: 10.0/10）

**根拠の検証**:
- ✓ README 冒頭で Hobbes/Jervis/Malthus 引用、4 法則明示
- ✓ REPORT_2026-05-06.md Abstract で「weapon ≥ 24 で攻撃数 7 倍相転移」「Pattern 1 は weapon ≥ 48」確認
- ✓ `sweep_weapon.py:24-31` — LEVELS / SEEDS / MAX_TICKS 一致
- ✓ `_classify_attack (:964-982)` — 優先順位 Revenge > P1 > P2 > Rumor > Mixed 実装
- ✓ 5 SCENARIO (:1738-1794) — hobbes に `"rumor_seed": "all"` 一致

**スコア妥当性**: 10.0 は妥当。phase diagram 実験はハッカソン標準を大きく超える。

**見落とし・誇張**: 「7倍ジャンプ」は REPORT 採用（README は「5倍」と書く）。原典内の表記揺れまでは検出していない。Schelling/Axelrod は評価で言及済み。

**検証結論**: 妥当

### C. 発展性（元: 8.0/10）

**根拠の検証**:
- ✓ 行数 2016 / 1350 を `wc -l` で確認一致
- ✓ monolithic 構造の指摘は実態と合致
- ✓ SCENARIOS dict + env var override (:1738-1794, :1825-1840) 一致
- ✓ sweep/analysis 分離（4 ファイル）一致
- ✓ V3_PLAN.md Sprint 1-4 (:104-225) 具体タスク一致
- ✓ FUTURE_WORK.md I-1/II-1 の作為性列挙、I-2/II-2 の改善方向 一致

**スコア妥当性**: 8.0 は若干保守的だが許容範囲（8-8.5）。

**見落とし・誇張**:
- 見落とし: `imgs/cover.png`, `docs/SLIDES_2026-05-06.md` 等のドキュメント整備度は未言及。`pyproject.toml` に `testpaths = ["tests"]` 設定はあるが `tests/` ディレクトリが空（`old_project/v1/tests/` に旧テスト 3 ファイル `test_brain_mock`, `test_perception`, `test_physics` のみ存在）。
- 誇張: なし。

**検証結論**: 妥当

### D. 技術実装（元: 9.0/10）

**根拠の検証**:
- ✓ 3 Agent 異なる temp (0.7 / 0.0 / 0.3) `:1836-1853` 一致。全 3 役割とも `gemini-2.5-flash-lite`、temperature のみ分化
- ✓ SpeakAction ToM フィールド（`poc_12agents.py:124-152`）一致
- ✓ timeout 10s + 503/429/UNAVAILABLE/RATE_LIMIT retry `wait = 5 + (2 ** attempt)` 最大 5 attempt (`:830-855`) 一致
- ✓ `_apply_memory_dedup (:684-732)` カテゴリ別 dedup + LRU 10 件 一致
- ✓ `render_common_ground (:622-668)` provenance 追跡 + 「再発話禁止」「お礼禁止」一致
- ⚠ `_apply_memory_decay (poc_world.py:783-796)` — 実際は `:763-779`。内容は一致、行番号がわずかにずれている
- ✓ `_record_attack_memory (:865-919)` 3 視点（被害者 -50 / 攻撃者 -5 / 目撃者 -20）memory + 関係性更新一致
- ✓ tick loop シーケンス (:754-771) — 実コード `:755-770`。評価の列挙では `_decay_relationships` が省かれている
- ✓ SEED 再現性 (`:1829-1832`) 一致
- ✓ runs.csv pattern別列 (`:1976-2012`) 一致
- ✓ logfire instrumentation（poc_12agents.py:36, poc_world.py:507）確認
- ✓ fire-and-forget `_maybe_replan` (`:1683-1716`) 確認

**スコア妥当性**: 9.0 は妥当（やや甘め）。LLM 3/3、Memory 3/3、CodeQ 1.0-1.5/2（2016 行 monolithic + 240 行ヒアドキュメント + tests 未整備）、SimCorrect 2/2 → 合計 9.0-9.5。

**見落とし・誇張**:
- 見落とし: `_decay_relationships`（100 tick おき 0.5% decay）が tick loop 列挙時に省かれている。モデル共通化（temperature のみ分化）も明示するとより正確。
- 誇張: なし。

**検証結論**: 妥当

## 総合所見

- **元評価の品質**: citation 正確性が極めて高く、コード行番号・関数名・出力値が大半そのまま再現できる。スコア配分も rubric に整合的。
- **主要な指摘点**:
  1. 「奇襲時 block_prob=0」は `surprise_block_prob` 設定可能変数（デフォルト 0）。
  2. 「7倍ジャンプ」は REPORT 出典、README は「5倍」と原典内不整合あり（評価は出典に忠実）。
  3. `_apply_memory_decay` 等の行番号 5-20 行ずれ（本質的影響なし）。
  4. brain/extractor/planner はすべて `gemini-2.5-flash-lite` で temperature のみ分化。
- **推奨アクション**: 元スコア 34.5/40 は維持可。CodeQ を厳格化するなら D を 8.5 に微調整して合計 34.0 も許容範囲。Hobbes/Jervis/Malthus を量的検証する独創性、精密な物理層、3 役割 LLM + 構造化メモリ + 共通基盤追跡の技術実装、`FUTURE_WORK.md` の批判的自己分析の質を統合的に反映した妥当なスコア。
