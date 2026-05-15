# 評価検証レポート: all-in-smoke

**検証日**: 2026-05-11
**対象**: `all-in-smoke_eval.md`
**元評価合計**: 28.0/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨スコア帯 |
|---|---|---|---|
| A. 創発設計 | 5.0 | 妥当 | 5-6 |
| B. 世界設定 | 8.0 | 妥当 | 7-8 |
| C. 発展性 | 7.0 | 妥当 | 7-8 |
| D. 技術実装 | 8.0 | 妥当 | 8-9 |

**総合判定**: 妥当

---

## カテゴリ別検証

### A. 創発設計（元: 5.0/10）

**根拠の検証**:
- `live_fire_simulation.py:1264-1331` — ✓実在・記述と一致。L1264 で `contact = state.danger >= FIRE_CONTACT_DANGER` から始まる純粋な閾値分岐の連鎖。LLM 呼び出しは一切なく、`if/elif` のチェーンで `stood_up` / `engulfed` / `fatal` / `clinging_to_stack` / `tempted_by_chips` / `hesitating` / `playing` を決定する。
- `live_fire_simulation.py:891-947` — ✓実在・記述と一致。`_crisis_inner_voice` は status/motive をキーに条件分岐で固定文字列テンプレを返す。LLM 不使用。
- `live_fire_simulation.py:35-36` — ✓実在・完全一致。`FIRE_CONTACT_DANGER = 0.92`、`FATAL_EXPOSURE_TICKS = 3` の定数定義。
- `poker_agents/llm_agent.py:33-101` — ✓実在・記述と一致。`_format_observation` は raw observation（hole_cards, board, legal_actions, recent_table_talk, session_context）を JSON 化するのみで、戦略指示なし。
- `configs/all_in_smoke_demo.yaml:3-77` — ✓実在・記述と一致。6 エージェント全てが `type: scripted` で `TightAgent`/`AggressiveAgent`/`CallingAgent`/`RandomAgent` のいずれか。LLM は呼ばれない。
- README "If fire reaches a still-seated agent..." — ✓README L7 付近に該当文言が存在（"fire reaches" でグレップ命中）。

**スコア妥当性**:
ハーモニックバランス評価のロジックが適切。世界ルール設計（ポーカーエンジン + 火災閾値ルール）は精密だが、作品ハイライトであるはずの「火災下意思決定」が if/elif の硬いルールで決定され、LLM が呼ばれない構造が決定的な減点ポイント。創発設計ルーブリック上は「世界ルール強・自由度弱」のミスマッチで 5-6 帯が適切。5.0 は妥当。

**見落とし・誇張**:
- 見落とし: `LlmAgent._build_messages` の `DEFAULT_SYSTEM_PROMPT`（personas.py の `SCHEMA_RULES`）は inner_voice の感情側面（恐怖、執着、後悔等）を明示的に要求しており、ポーカー判断側ではむしろ強めのプロンプト誘導がある点に未言及。これは「自由度高」評価をやや弱める材料だが、本文は「raw observation のみ」と紹介。ただし `reasoning` と `inner_voice` の役割分離は技術側評価で言及済み。
- 誇張: 特になし。

**検証結論**: 妥当

---

### B. 世界設定（元: 8.0/10）

**根拠の検証**:
- README "Poker-phase incomplete information..." — ✓README L3 に完全一致で存在。
- `live_fire_simulation.py:692-797` — ✓実在・記述と一致。`_crisis_match_snapshot` は L692-798 にかけて定義され、L704-707 / L719-723 で `fatal_overrides_chips` / `stand_up_forfeits_stack` / `last_seated_claims_forfeited_chips` / `poker_result_breaks_tie_after_survival` の 4 ルールが明示的にエンコードされている（評価は 3 つを主軸として言及、追加の `poker_result_breaks_tie_after_survival` には言及なし）。
- `smoke_simulation.py:30-49` — ⚠軽微な不整合。実際は L31-40 が `DEFAULT_ROOM_LAYOUT`（table_a/b/c, bar, storage, spectators, exit_a/b の 8 箇所）、L42-49 が `DEFAULT_SEAT_LOCATIONS`。L30 はちょうど直前の `CRISIS_ABILITY_KEYS` タプルの末尾近傍で評価が指定する範囲は若干前後拡張だが、主張（部屋レイアウト + 座席配置）は正しい。
- `smoke_timeql_converter.py:92-197` — ✓実在・記述と一致。`_compute_timeql_profile_to_ability_gaps` 関数本体で、`fold_ability` / `public_responsibility` / `situational_awareness` / `trust_calibration` / `help_seeking` / `self_control` / `reciprocity` / `meaning_update` の 8 種類の能力ギャップを body/lack/traits ベクトルから派生させる詳細なマッピング。
- `live_fire_simulation.py:104-148` — ✓実在・記述と一致。`PokerFeedbackState` dataclass で `chip_attachment` / `loss_chasing` / `entitlement` / `confidence` / `table_image_pressure` / `rivalry_pressure` / `fold_success_memory` / `tilt` / `recent_delta` / `hands_played` / `eliminated` の 11 次元を保持。評価は「9 次元」と言及するが実際は 11 フィールド（うち圧力 7 + tilt + eliminated = 9 が float の心理プレッシャー、recent_delta/hands_played は実数カウンタ）。意味は妥当。

**スコア妥当性**:
ポーカー（ゲーム理論）× 火災危機（避難心理学）の融合は確かに独創的で、行動経済学の研究設定として強い。`fatal_overrides_chips` 等の三つ巴ルールは「サンクコスト vs 命」を直接モデル化しており、ルーブリック 8-9 帯（実世界ダイナミクスを明確にモデル化）に合致。一方、「全くの新規性」というよりは既存の避難シミュレーション + ゲームの組み合わせなので 9-10 には届かない。8.0 は妥当。

**見落とし・誇張**:
- 見落とし: `_crisis_match_snapshot` には `poker_result_breaks_tie_after_survival` という 4 つ目のルールも存在（L707, L723）。「生存後にポーカー結果が tie-break する」というルールは「生存しただけでは勝てない」という追加圧力を生む重要な設計だが未言及。
- 見落とし: `DEFAULT_ROOM_LAYOUT` は exit_a が `safe: False`、exit_b が `safe: True` という非対称設計（L38-39）。これも避難経路の選択に意味を与える重要な設定だが言及なし。
- 誇張: 「行動経済学・社会心理学の研究的射程を持つ」は、`PokerFeedbackState` の `chip_attachment`/`loss_chasing` 等が実際に避難遅延に変換される実装が確認できるため妥当な表現。

**検証結論**: 妥当

---

### C. 発展性（元: 7.0/10）

**根拠の検証**:
- `poker_agents/manifest_loader.py:38-43` — ✓実在・記述と一致。`SCRIPTED_CLASSES` dict で 4 種の scripted agent をマッピング。
- `poker_agents/manifest_loader.py:66-137` — ✓実在・記述と一致。`AgentSpec.build` メソッドで scripted/endpoint/llm/openrouter の 4 種を統一的に構築。
- `configs/` (9 個の YAML) — ✓実在・記述と一致。配下を ls すると `all_in_smoke_demo.yaml`, `all_in_smoke_ollama_smoke.yaml`, `all_in_smoke_openrouter_grok_full.yaml`, `all_in_smoke_openrouter_grok_smoke.yaml`, `poker_demo.yaml`, `poker_llm_demo.yaml`, `poker_mindsport_6p.yaml`, `poker_mindsport_smoke.yaml`, `poker_mindsport.yaml` の 9 ファイル。
- README "Layer 3" — ✓README L111 に "## Layer 3" の節が存在。
- `tests/` 12 ファイル — ✓実在・正確。`test_*.py` ファイルは 12 個（conftest.py 除く）で、列挙された全モジュール（session_state, poker_engine, poker_simulation, smoke_simulation, timeql_converter, replay_timing, llm_agent, openrouter_agent, byo_agent, card_language, commentator, tts_normalizer）を網羅。
- `live_fire_simulation.py` 1398 行 — ✓正確。`wc -l` で 1398 行を確認。
- README が研究的問いを前面に出していない・`smoke_simulation.py:478` にのみ "Does poker-table trust transfer into fire-alarm trust?" が存在 — ✓正確。grep で確認できる唯一の出現箇所。

**スコア妥当性**:
コード拡張性は確かに高く、レジストリパターン・YAML プラガブル・テスト網羅・複数LLMバックエンド対応・persona/voice_profile/TimeQL の三層化など、拡張ポイントが体系的。一方、「将来展望」セクションは技術手順書中心で、ハイレベルなビジョン・研究仮説（避難心理学の何を検証したいか、どう測定するか）の記述が薄い。コードの強さと展望の薄さの平均で 7-8 帯。7.0 は妥当だが、コード拡張性側を重視すれば 7.5 寄りでも擁護可能。

**見落とし・誇張**:
- 見落とし: `replay_timing.py` モジュールが独立してテストされており、リプレイの再現性・タイミング制御という研究プラットフォーム機能が含まれている点が拡張性評価を強化する材料。
- 見落とし: `tools/` ディレクトリ（MP4 エクスポート等）の存在も拡張性の評価材料。
- 誇張: 「research/business 的問いを前面に出していない」は実態と合っているが、README 全体（特に Layer 3 節）は仮説検証フレームの示唆を含んでおり、完全否定するほどではない。微小誇張だが致命的ではない。

**検証結論**: 妥当

---

### D. 技術実装（元: 8.0/10）

**根拠の検証**:
- `poker_agents/llm_agent.py:169-217` — ✓実在・記述と一致。L169-180 で `format: "json"` 強制、L189-217 で `urllib.error.URLError` / `TimeoutError` / `json.JSONDecodeError` / 非 dict ペイロードの全てを fold にフォールバック。
- `poker_agents/openrouter_agent.py:88-148` — ✓実在・記述と一致。L94 で `response_format: {"type": "json_object"}`、L96 で `max_completion_tokens` 制御、L102-107 で API キー欠如時の早期 fold、L119-147 で同様のフォールバック群。
- `poker_agents/personas.py:15-54` — ✓実在・記述と一致。`SCHEMA_RULES` 定数に 10 項目の制約（action enum, amount range, inner_voice 必須, カード表記等）。
- `poker_agents/session_state.py:113-229` — ✓実在・記述と一致。`ingest_hand_result` の本体で tilt 減衰（L129）、大負け加算（L148-153）、badbeat 検出（L156-180）、rivalry 自動記録（L171-212）、recent_outcomes ledger（L221-228）が実装。
- `poker_agents/session_state.py:270-297` — ✓実在・記述と一致。`prompt_block` メソッドで `mood_label()` のみ露出し numeric tilt は非露出。docstring（L274-277）にも「Hides the numeric tilt」と明記。
- `live_fire_simulation.py:202-279` — ✓実在・記述と一致。`_feedback_snapshot_by_step` 関数本体で `action`/`memory_reasoning`/`hand_result`/`session_snapshot` 各イベント種別から累積。
- `poker_simulation.py:172-188` — ✓実在・記述と一致。`JsonlLogger.next_step` メソッド（L172-174）が `self.step += 1` で単調増加、`log` メソッド（L176-187）が action/memory_reasoning/table_talk を同期。
- `poker_agents/card_language.py:25-103` — ✓実在・記述と一致。`normalize_card_language` 関数本体で `re.sub` ベースの正規化が 20 個以上（L35-100 で約 18 個の `re.sub` 呼び出し + 5 個程度の `str.replace`）。
- `tests/` 12 ファイル、3000 行超 — ✓ファイル数は正確。行数は確認していないが妥当な見積もり。

**スコア妥当性**:
- LLM 利用（3 点満点）: 複数バックエンド対応、構造化 JSON 出力強制、10 項目のスキーマ規則、防御的フォールバック、persona/voice_profile/TimeQL の三層プロンプト合成 — 高水準。約 2.7/3。
- メモリ設計（3 点満点）: SessionState の tilt 減衰、badbeat 検出、rivalry 蓄積、prompt_block での mood label 露出、PokerFeedbackState による 9 次元プレッシャー転移 — 設計の意図が明確で技術水準が高い。約 2.7/3。
- コード品質（2 点満点）: 全モジュール dataclass + 型ヒント、JsonlLogger による同期 step、テスト網羅 — 高品質。約 1.7/2。減点要素は `live_fire_simulation.py` の 1398 行単一ファイル。
- シミュレーション正確性（2 点満点）: 乱数 seed 制御、ハンド毎の `seed + hand_index`、契約明確な poker_engine — 高い。約 1.7/2。

合計約 8.8/10 → 8.0 はやや控えめ寄りだが、危機 inner_voice 完全テンプレ化を D の範囲に含めて減点する解釈なら 8.0 は妥当。8-9 帯。

**見落とし・誇張**:
- 見落とし: `_crisis_match_snapshot` の同じ `seat[0] == loser` / `winner = seats[1] if seats[0] == loser else seats[0]` パターンの繰り返しは軽微なリファクタ余地。技術評価には影響しない軽微な点。
- 見落とし: テストカバレッジが「3000 行超」と推定されているが実測ではないため、誇張の可能性。ただし重要な減点理由ではない。
- 誇張: 「危機 inner_voice 生成が完全にテンプレ化されている点（D のスコープでは些細）」と自己注記しており、慎重な記述。問題なし。

**検証結論**: 妥当

---

## 総合所見

- 元評価の品質: 引用された 22 件の `filename:line_range` および README 引用のうち、ほぼすべてが実在・内容一致で、軽微な不整合（行範囲の前後拡張、PokerFeedbackState の次元数を「9」と表現する点）は 2 件程度に留まる。記述の精度は高く、創発設計のハーモニックバランス評価（5-6 帯）も妥当に運用されている。
- 主要な指摘点:
  1. B カテゴリで `_crisis_match_snapshot` の 4 つ目のルール `poker_result_breaks_tie_after_survival` が言及されていない（評価は 3 つに絞っている）。世界設定の独創性をさらに強化する材料。
  2. `PokerFeedbackState` の次元数は厳密には 11 フィールド（うち心理プレッシャー float 9 個 + recent_delta + hands_played）。評価の「9 次元」はおおむね正しい。
  3. A カテゴリで `SCHEMA_RULES` の inner_voice 強要（personas.py L34）が「raw observation のみ」という主張をやや弱める材料だが、本文では言及されていない。
- 推奨アクション: スコア・記述ともそのまま採用可能。改善するなら、B で 4 つ目のルールと exit_b の非対称安全性に触れる、A で `SCHEMA_RULES` の感情側面プロンプト誘導も補足する、の二点で精度が一段上がる。
