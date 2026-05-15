# 評価検証レポート: automata-hackathon-2026-ebiyama

**検証日**: 2026-05-11
**対象**: `automata-hackathon-2026-ebiyama_eval.md`
**元評価合計**: 32.0/40

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨スコア帯 |
|---|---|---|---|
| A. 創発設計 | 6.0 | 妥当 | 6-7 |
| B. 世界設定 | 9.0 | 妥当（やや過小） | 9-10 |
| C. 発展性 | 8.0 | 妥当 | 7-8 |
| D. 技術実装 | 9.0 | 妥当 | 9-10 |

**総合判定**: 妥当（軽微な誇張あり、修正不要。32.0/40 は維持可、強いて言えば 33.0 まで上振れ許容）

## カテゴリ別検証

### A. 創発設計（元: 6.0/10）

**根拠の検証**（全件 ✓実在・記述一致）:
- `agent.py:922-989` — WORKING STATE block 確認
- `agent.py:2317-2332` — 残り5step帰路強制ロジック確認
- `agent.py:1079-1091` — school_fit='不適応' 発話抑制確認
- `agent.py:1393-1394` — "strictly quantitative data" 文言確認
- `simulation.py:1952-1974` — host=stay 制約確認
- `simulation.py:984-1056` — awareness 3 経路実装確認
- `place_types.py:20-140` — registry 構造確認

**スコア妥当性**: 「strictly quantitative data」原則と豊富な情報チャネル（3 経路 awareness）は精度高い世界ルール設計を示唆。一方、host=stay や残り5step帰路強制など強い行動制約があり、行動自由度はやや限定的。6.0 はバランスの取れた妥当評価。

**見落とし・誇張**:
- 見落とし: simulation.py:1326-1346 で host が学生近接していないと即 silent を返す追加 gate（コスト最適化兼 scaffolding）も未言及。
- 誇張: 「3 経路 awareness 伝播」の表現は厳密だが、Path 2 (conversation) は coarse なキーワード一致のみ（コードコメント "Deliberately coarse"）であり、評価本文はこの限界に触れていない。

**検証結論**: 妥当

### B. 世界設定（元: 9.0/10）

**根拠の検証**（全件 ✓実在・記述一致）:
- README 4 引用すべて存在を確認
- `configs/config_shinagawa_field_elementary_v2_100step.yaml:1030-1250` — perceive_pass/perceive_enter 42 件確認
- `docs/shinagawa_context.md` 存在確認
- `agent.py:1140-1156` — `_build_premise_block_for_system` で品川二面性挿入確認
- `tools/run_survey.py:1-300` — 観察者LLM経由設計確認

**スコア妥当性**: 品川の実地理（perceive_pass/enter 42 件）と二面性（教育/再開発）を世界設定に深く統合。観察者LLM・触媒人物（佐藤航陽氏）を介した教室シミュは独創性高い。ルブリック「9-10: highly original」に該当。9.0 は妥当だが 9.5-10 の方が公平な可能性あり。

**見落とし・誇張**:
- 見落とし: README に明記されている佐藤航陽氏 (Singulab 代表) を「触媒人物」とする教室シミュ条件 A/B 比較設計が拾われておらず、B カテゴリの独創性評価をやや押し下げている。
- 誇張: なし。

**検証結論**: 妥当（やや過小評価気味、9.5 も許容範囲）

### C. 発展性（元: 8.0/10）

**根拠の検証**（全件 ✓実在・記述一致）:
- ファイル行数完全一致: `agent.py` 2609 / `simulation.py` 2245 / `navigation.py` 442 / `place_types.py` 154 / `llm_client_factory.py` 439 / `utils.py` 154
- `configs/` 13 個確認
- `tools/` 21 個確認
- `llm_client_factory.py:405-439` — `create_llm_client` 確認

**スコア妥当性**: モジュール分離（agent/simulation/navigation/place_types/llm_client_factory/utils）と 13 個の configs / 21 個の tools による外部化は明確な拡張性を示す。一方、README に Future Work セクションが明示的に整理されているかは強くない。コード拡張性は高く、ビジョンは中庸。8.0 は妥当だが 7.5 寄り。

**見落とし・誇張**:
- 軽微な誇張: D セクションで「tools 8770 行」と書かれているが検証側では未確認（本体合計は wc -l で 6323 行確認）。tools 全体行数は別途要検証。
- 見落とし: なし。

**検証結論**: 妥当

### D. 技術実装（元: 9.0/10）

**根拠の検証**（全件 ✓実在・記述一致）:
- `llm_client_factory.py:210-241` — sha256 double-checked locking 確認
- `llm_client_factory.py:30-72 + 261-264` — `_pick_gemini_schema` 動作確認
- `agent.py:228-240` — 3層メモリ確認
- `agent.py:1004-1034` — `maybe_compress_memory` 確認
- `agent.py:415-458` — `_build_memory_context` 確認
- `agent.py:477-509` — Jaccard 0.50 silent 化確認
- `agent.py:540-575` — ⚠警告挿入確認
- `agent.py:696-710` — truncation 検出確認
- `agent.py:2229-2249` — behavior_layer 確認
- `simulation.py:56-62` — per-sink lock 確認
- `simulation.py:65-73` — dual seed 確認
- `simulation.py:1790-1797` — 双方向同時発話排除（大きい id silent）確認

**スコア妥当性**: LLM クライアントの sha256 + double-checked locking、Gemini schema 自動選択、3層メモリ + 圧縮、Jaccard ベース重複検出、dual seed の再現性管理など、本ハッカソン提出群の中でも最上位の技術実装。9.0 は妥当、9.5 も許容。

**見落とし・誇張**:
- 見落とし: tools 行数の検証不足（評価では「8770 行」と記載）。
- 誇張: なし。

**検証結論**: 妥当

## 総合所見

- **元評価の品質**: 引用 18 件超のうち、致命誤りゼロ件、行数ズレも ±2 行以内。本評価は本ハッカソン提出群の中でも citation 精度・スコア重み付け説明・改善提言の actionability すべてにおいて最上位レベルの品質。
- **主要な指摘点**:
  1. B カテゴリの独創性は 9.5 寄りで評価できる余地あり（佐藤航陽氏触媒設計の言及補強）。
  2. D カテゴリの「tools 8770 行」は要再検証（本体は 6323 行確認）。
  3. A カテゴリで Path 2 awareness の coarse 性質と host silent gate の言及があるとさらに精度向上。
- **推奨アクション**: 元スコア 32.0/40 をそのまま採用可。修正不要。微調整するなら A=6→6.5、B=9→9.5 で合計 33.0 まで上振れも許容範囲。
