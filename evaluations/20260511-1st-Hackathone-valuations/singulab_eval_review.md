# 評価検証レポート: singulab

**検証日**: 2026-05-11
**対象**: `singulab_eval.md`
**元評価合計**: 31.5/40

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨スコア帯 |
|---|---|---|---|
| A. 創発設計 | 8.0 | 妥当 | 7.5-8.5 |
| B. 世界設定 | 7.5 | 妥当 | 7.0-8.0 |
| C. 発展性 | 8.0 | 妥当（軽微な誇張あり） | 7.5-8.0 |
| D. 技術実装 | 8.0 | 妥当（行番号ズレあり） | 7.5-8.5 |

**総合判定**: 元評価は全体として妥当。論点（エコー対策・3階層フレーム・LRU 認知・複数 seed 未取得）は的確に押さえられている。ただし、(1) 行番号が複数箇所でズレている、(2) "CLAUDE.md に httpx async 計画あり" という記述は CLAUDE.md 上で確認できない、という不整合がある。総合 31.5 はやや甘めに見える局面もあるが、5 体 vs 100 体の独立変数比較と豊富な検証ログを考慮すると妥当範囲内。

## カテゴリ別検証

### A. 創発設計（元: 8.0/10）

**根拠の検証**:
- ✓ `src/physics/cognition.py:60-110` — LRU + memory_evicted/re_encountered（実51-100行、ほぼ一致）
- ⚠ `src/events/alien.py:103-150` — 距離依存知覚。「near/mid/far の 3 帯」とあるが実装上は「near/mid の 2 バンド + 圏外（他者経由）」で表現がやや誇張
- ✓ `src/world/world.py:80-140` — capacity 違反 `place_entry_denied` 実在
- ✓ `src/agent/agent.py:139-160` — `_build_messages_context` 直前 1 件のみ（167行付近）実在
- ✓ `src/simulation.py:434-470` — `personal_contexts` 状態記述コメント実在
- ✓ `src/agent/agent.py:286-320` — task_section 誘導（305-322）実在

**スコア妥当性**: ルール × 自由度のバランス評価はコード実態と整合。プロンプト誘導（質問・確認・提案）が確認でき、9-10 帯には届かない判断は正しい。8.0 は妥当。

**見落とし・誇張**:
- 見落とし: `_limit_message_words` による語数制限の言及なし。
- 誇張: 「3 帯」表現は厳密には 2 帯 + 圏外。

**検証結論**: 妥当（7.5-8.5）

### B. 世界設定（元: 7.5/10）

**根拠の検証**:
- ✓ README "3階層フレームワーク" 実在
- ✓ `config/base.yaml` — 国籍 9 ヵ国 + 90/1.25×8、MBTI 16、年齢 20-60（31-37行）一致
- ✓ `config/scenario_alien_5/zero_gravity_5/alien_100.yaml` 実在
- ✓ 提出物サマリ「役割の自然発生・ハブの自然発生・3 層構造の量的分化」（14行目）完全一致
- ✓ `src/world/environment.py:17-23` — economy good/bad/neutral 状態記述 実在

**スコア妥当性**: 社会学的観察軸の評価は強気だが、5 体 vs 100 体比較の成果回収を踏まえると許容範囲。7.5 は妥当。

**見落とし・誇張**:
- 誇張: 「UFO/無重力/火事の比較」と書かれているが、火事（`scenario_bar_fire.yaml`）は 5/100 体比較に組み込まれておらず、主比較は alien と zero_gravity のみ。やや誇張気味。

**検証結論**: 妥当（7.0-8.0）

### C. 発展性（元: 8.0/10）

**根拠の検証**:
- ✓ `src/events/base.py` — `Event` 抽象基底（`@abstractmethod` で `maybe_activate` / `perceived_info`）実在
- ✓ 3 種イベント実装 実在
- ✓ `src/config_loader.py` — extends + deep_merge 実在
- ⚠ `src/agent/persona.py:97-126` — PersonaFactory + `_is_legacy_config` 実在だが行番号ズレ（実際 88-130）
- ✓ 提出物サマリ「複数 seed 未実施 §7.3」 実在（文言は若干異なるが意味一致）
- ✗ `CLAUDE.md` の "Phase 2 で `src/llm/base.py` の抽象化 + httpx async に置き換え予定" — **存在しない**。CLAUDE.md には httpx / async / 抽象化 / llm/base.py いずれもヒットせず。**創作または誇張**

**スコア妥当性**: モジュール分離・extends 継承・legacy 拒否は確実に存在。ただし将来展望の半分を CLAUDE.md 架空引用に依拠しており差し引くと 7.5 寄り。

**見落とし・誇張**:
- 誇張: CLAUDE.md 引用 ✗ が最大の問題。
- 代替案: `src/physics/base.py` の "message_delay 強制配送は Phase 3 以降" は ✓実在しており差し替え可能。

**検証結論**: 妥当だが軽微な誇張（7.5-8.0）

### D. 技術実装（元: 8.0/10）

**根拠の検証**:
- ✓ `src/llm/ollama.py:60-95` — Ollama + think + try/except（59-94）実在
- ✗ `src/agent/agent.py:381-420` — JSON 抽出ブレース対応。**行番号誤り**。実際は `_extract_json_from_text` が 477-503、`_extract_direction_from_text` が 506-512。約 100 行ズレ
- ✓ `src/physics/cognition.py:48-100` — LRU + re_encountered 実在
- ✓ `src/simulation.py:60-65` — random.seed + np.random.seed（58-62）実在
- ✓ `src/runlog/metrics.py:79-200` — Gini/silent/pair_stability/event_counts（各 74/128/216/47 行）実在
- ✓ `tests/test_phase5_events.py` 実在
- ✓ `config/base.yaml` — max_tokens 512→100, temperature 0.5→0.8（64-68）完全一致
- ⚠ `src/agent/agent.py:557-580` — LLM 失敗フォールバック 軽微ズレ（553-577）

**スコア妥当性**: LLM(3): think:false + チューニング根拠ログ、Memory(3): 二重メモリ + LRU 社会記憶、CodeQ(2): モジュール分離 + 型ヒント、SimCorrect(2): seed 同時固定 + run_metadata は全て確認できる。8.0 は妥当。

**見落とし・誇張**:
- 誇張: `agent.py:381-420` の行番号誤りが信頼性を損なう。
- 妥当: リトライ無し指摘は ✓妥当。テスト 6 ファイルは実数と一致 ✓。

**検証結論**: 妥当（7.5-8.5）

## 総合所見

- **元評価の品質**: 論点抽出は的確（エコー対策・3 階層フレーム・LRU 社会記憶・5 vs 100 比較）。改善提言も実装根拠あり。一方、行番号引用の精度が不安定で、CLAUDE.md からの「httpx async 計画」引用は実在しない。
- **主要な指摘点**:
  1. `src/agent/agent.py:381-420` は誤り（正しくは 477-512）
  2. `CLAUDE.md` の "httpx async 置き換え予定" は ✗ 創作・誇張
  3. "near/mid/far の 3 帯" は実装上 2 バンド+圏外
  4. 提出物サマリの seed 未取得引用は文言相違あるが意味一致 ✓
- **推奨アクション**:
  - 元スコア 31.5/40 は維持可。
  - C の根拠から CLAUDE.md 架空引用を削除し、`src/physics/base.py` の Phase 3 以降配送計画など実在記述に置き換え。
  - D の行番号を `agent.py:477-512` に修正。
  - A の "near/mid/far" を "near/mid + 圏外（他者経由）" に統一。
  - 推奨総合スコア帯: 30.0-32.0
