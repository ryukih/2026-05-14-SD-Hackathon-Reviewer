# 評価検証レポート: lunar_simulation

**検証日**: 2026-05-11
**対象**: `lunar_simulation_eval.md`
**元評価合計**: 30.0/40

---

## 検証サマリー

| カテゴリ | 元スコア | 引用検証 | スコア妥当性 | 総合判定 |
|---------|---------|---------|-------------|---------|
| A. 創発設計 (rules x freedom) | 8.0/10 | ✓ 全引用一致 | 妥当 | 維持 |
| B. 世界設定 (originality) | 7.0/10 | ✓ 全引用一致 | やや甘めだが妥当 | 維持 |
| C. 発展性 (modularity+vision) | 7.0/10 | ✓ 全引用一致 | 妥当 (vision側にやや厳しいが筋が通っている) | 維持 |
| D. 技術実装 (LLM+Mem+CodeQ+SimCorrect) | 8.0/10 | ✓ 全引用一致 | 妥当 | 維持 |
| **合計** | **30.0/40** | — | — | **30.0/40 を支持** |

総合判定: 元評価は引用も論理も精度が高く、提示されたスコアは適正範囲に収まっている。1点だけ補強できる箇所 (A の「定員10人 vs 20人」という設計圧力の鋭さ、D の MockLLMClient の規模感) があるが、最終スコアを動かすほどではない。

---

## カテゴリ別検証

### A. 創発設計 (rules x freedom) — 元スコア 8.0/10

**根拠の検証**: ✓ 全引用一致

- `agent.py:340-348` の「Decide what message to send...」最小指示プロンプト → 確認 (該当箇所は `create_message_prompt` 末尾、YOUR TASK セクション)。
- `agent.py:443-451` の「stay | move: up/down/left/right」JSON仕様 → 確認 (`create_decision_prompt` の AVAILABLE ACTIONS / RESPOND IN JSON 部分)。「定員超過なら退去せよ」のような命令的指示は存在しない。
- `agent.py:236-254` の `_build_oxygen_section` で `steps_to_death = int(self.oxygen_level / current_consumption_rate)` を事前計算し `~N steps until depletion` として渡している → 確認 ✓。これは指摘通り軽い行動誘導。
- `agent.py:325-326` の `collective_prompt` フラグによる "operating as part of a group under survival constraints" 付与 → 確認 (`create_message_prompt` 内)。`create_decision_prompt` にも同じ条件分岐が存在 (重複適用)。
- `simulation.py:497-506` の自動酸素消費 (`_apply_oxygen_consumption`) → 確認。エージェントは酸素を「使う」アクションを持たず、世界側ルールで強制される設計。
- `config.yaml:5-13` の通信半径・入口越境半径などの外部化 → 確認 (`communication_radius: 5`, `entry_communication_radius: 2`)。
- `utils.py:78-106` の `is_position_near_place_entry` (正方基地+入口ゾーン) → 確認 ✓。

**スコア妥当性**: 8.0/10 は妥当。世界ルールの精密さ (3層酸素消費、定員10/20人、入口ゾーン)、LLM への raw 数値提示、行動戦略の非規定性が高水準で揃っている。`steps_to_death` 前計算と `collective_prompt` という軽い誘導要素を勘案して 9 点ではなく 8 点に落とすロジックは健全。

**見落とし・誇張**: 軽微な見落としあり。20人 vs 定員10人 (`config.yaml:6,27`) は単なる「シェルター不足」ではなく数学的に必ず半数が締め出される強い対立圧力で、創発のドライバとしてはより高評価でもよい。一方、評価本文がそれを過小評価しているわけでもなく、スコアには反映されている。誇張は見当たらない。

**検証結論**: A=8.0 を支持。

---

### B. 世界設定 (originality) — 元スコア 7.0/10

**根拠の検証**: ✓ 全引用一致

- README の月面 2D 空間 / 20人 / 定員10人 / 中央基地という説明 → 一致。
- `config.yaml:21-27` の `central_lunar_base` のみ、`capacity: 10` → 確認。
- `config.yaml:6` の `num_agents: 20` (定員の倍) → 確認。
- `agent.py:19-24` の 4 方向移動のみ → 確認 (`DIRECTION_MAP` は up/down/left/right の4要素のみ、対角線・斜め移動なし)。
- `events/rescue_announcement.py` (Mission Control 救助メッセージによる介入) → 該当ファイル存在を確認。`event_catalog.yaml` にも `rescue_announcement` が記述されている。

**スコア妥当性**: 7.0/10 は妥当。月面酸素制約というテーマは標準的な town/village 系デモから明確に逸脱しており originality は高い。一方、「火災・嵐シナリオに置換可能な抽象度」「物理は1基地・地形なし・4方向のみ」という指摘は事実に即している。

**見落とし・誇張**: 誇張は無い。指摘される「ICUベッド配分・避難所運用」アナロジーは設定上自然で妥当。`rescue_announcement` 1種類しかイベント実装が存在しない (`events/` ディレクトリは `base.py` `registry.py` `rescue_announcement.py` の3ファイルのみ) という事実は評価本文の「単純さ」評価と整合的。

**検証結論**: B=7.0 を支持。

---

### C. 発展性 (modularity + vision) — 元スコア 7.0/10

**根拠の検証**: ✓ 全引用一致

- `events/registry.py:10-37` の `EVENT_TYPES` 辞書登録パターン → 確認 (`EVENT_TYPES: Dict[str, Type[SimulationEvent]] = {RescueAnnouncementEvent.event_type: RescueAnnouncementEvent,}` および `build_events()` でのファクトリ生成)。
- `events/base.py:13-55` の `SimulationEvent` 基底クラスと `should_fire` / `apply` / `manifest` メソッド分離 → 確認。
- `simulation.py:54-78` の `places` リスト型バリデーション (`isinstance(self.places, list)`, `required_fields` チェック) → 確認 ✓。複数施設対応の構造が確かに存在。
- `llm_client.py:748-803` の `build_llm_client` 関数で ollama/lmstudio/gemini/mock の4プロバイダ分岐 + auto fallback → 確認 ✓。
- `event_catalog.yaml:1-43` のイベント拡張カタログ (`event_types` / `config_fields` / `example`) → 確認。
- README に「Future Work」セクションが無い → 確認。README は実行方法 / 出力 / 提出用成果物 / イベント拡張の構成で、応用領域や研究展望の記述は存在しない。
- `config_collective.yaml`, `config_low_thinking.yaml` の存在 → 確認 (`config_low_thinking_collective.yaml` も加えると 3 + 1 バリアント)。
- `make_20run_comparison_report.py` の存在 → 確認 (約 33KB の比較レポート生成スクリプト)。実は `make_10run_comparison_report.py`, `make_collective_comparison_report.py`, `make_collective_integrated_report.py`, `make_integrated_report.py`, `make_swap_report.py` など多数の集計スクリプトがあり、評価本文の「実験フレームワークとしての完成度は高い」は正しい。

**スコア妥当性**: 7.0/10 は妥当。「コード拡張性」だけ見れば 8〜9 相当だが、評価本文は意図的に「将来展望が README で弱い」点で減点しており、修正基準 (modularity+vision の合算) に従っている。

**見落とし・誇張**: 見落としは少ない。誇張も見当たらない。やや細かい点として、`config_low_thinking_collective.yaml` の存在が引用に挙がっておらず、合計 4 バリアントである事実が省かれているが、スコアへの影響は無い。

**検証結論**: C=7.0 を支持。README に展望章を追加すれば 8.0 まで上振れする余地あり、という改善提言も的確。

---

### D. 技術実装 (LLM3 + Mem3 + CodeQ2 + SimCorrect2) — 元スコア 8.0/10

**根拠の検証**: ✓ 全引用一致

- `agent.py:270-348` (`create_message_prompt`) — 座標を含めない (`include_position=False`、`current_state_lines` に Position 行が無い) → 確認 ✓。これは指摘通り「メッセージ判断時はプライバシー保護」設計。
- `agent.py:350-452` (`create_decision_prompt`) — `current_state_lines = [f"Position: ({self.position[0]}, {self.position[1]})", ...]` で座標明示、`BASE LOCATIONS` セクションで基地座標も提示 → 確認 ✓。
- `llm_client.py:357-366` の Gemini 構造化出力 (`responseMimeType`) と `thinkingConfig.thinkingLevel` → 確認 ✓。
- `llm_client.py:378-433` のリトライ・指数バックオフ (`for attempt in range(1, 4)`, `time.sleep(1.5 * attempt)`, HTTP 429/500/502/503/504 のみリトライ) → 確認 ✓。
- `simulation.py:291-306` の `ThreadPoolExecutor` 並列実行 (`_run_llm_jobs`) → 確認 ✓。`max_workers = min(self.llm_parallel_workers, len(jobs))` で適切に上限制御。
- `agent.py:454-489` のネスト対応 JSON 抽出 (波括弧深度カウント + 文字列リテラル内のエスケープ処理) → 確認 ✓。堅牢な実装。
- `agent.py:621-630` の LLM 出力 `memory` フィールドを次ステップに引き継ぐ自己フィードバック (`memory_entry = f"Step {step}: {memory_content}"`, `self.memory.append(...)`, `self.memory.pop(0)` で FIFO 上限) → 確認 ✓。
- `simulation.py:572-738` の同期 4 フェーズ step (`Phase 1` メッセージ判断 / `Phase 2` メッセージ送信 / `Phase 3` アクション判断 / `Phase 4` 移動) → 確認 ✓。docstring が "New order: 1.〜4." として明示。
- `simulation.py:523-570` の `_move_with_entry_limit` (定員チェックで `place_counts[target_place_name] >= target_capacity` なら移動拒否、`agent.remember` でログ) → 確認 ✓。
- `main.py:102-114` の `_redact_sensitive_config` (`api_key`, `apikey`, `token`, `secret` を `[REDACTED]` に置換) → 確認 ✓。
- `main.py:117-178` の `write_experiment_record` (config_used.yaml + experiment_conditions.md 出力) → 確認 ✓。

**スコア妥当性**: 8.0/10 は妥当。LLM 統合の成熟度 (2段階プロンプト・構造化JSON・リトライ・並列化・Mock) はサブミッション群の中で上位レベル。再現性 (random.seed なし) のマイナス指摘も実コードと整合 (`simulation.py` の `_generate_random_position` は `random.randint` を直接呼んでおり、シード固定機構なし)。

**見落とし・誇張**: 軽微な見落としあり。`MockLLMClient` (`llm_client.py:482-746`) は約 260 行に及ぶ近傍探索・基地誘導ヒューリスティクスを実装した本格的なフォールバックで、評価本文の「オフライン検証も可能」より一段強い意義を持つ (API 課金なしでフル比較実験ができる)。ただしスコアは既に 8.0 でこれ以上の押し上げは過剰と判断され、現スコアを支持する。誇張は見当たらない。

**検証結論**: D=8.0 を支持。

---

## 総合所見

元評価 `lunar_simulation_eval.md` は、引用されたファイル名・行番号・記述内容のすべてが実コードと一致しており、検証ヒット率は 100% に近い。Run1/Run2 とも全カテゴリで同点 (8/7/7/8) で安定しており、平均化による分散低減も適切に機能している。

スコア面では、A=8.0 / B=7.0 / C=7.0 / D=8.0 = 計 30.0/40 は本作の特徴 (鋭い構造的ジレンマ + 精密な世界ルール + 成熟した LLM 統合 + やや浅い世界物理と希薄な vision) を素直に反映しており、調整の必要は感じられない。

改善提言 (README への Future Work 章追加、シード制御、世界物理の厚み、`steps_to_death` の raw 提示オプション化、`collective_prompt` 実験結果の README 記載) も、いずれもコード根拠から導かれる現実的かつ建設的な内容で、提出者が次ステップで取り組める粒度になっている。

最終判定: **元評価 30.0/40 を支持**。検証側からの修正提案なし。
