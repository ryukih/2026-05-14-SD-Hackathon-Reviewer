# 評価検証レポート: hackathon-singulab-inu

**検証日**: 2026-05-11
**対象**: `hackathon-singulab-inu_eval.md`
**元評価合計**: 33.0 / 40

---

## 検証サマリー

| カテゴリ | 元スコア | 引用検証 | スコア妥当性 | 総合判定 |
|---|---|---|---|---|
| A. 創発設計（ルール×自由度） | 7.0 / 10 | ✓ 全行番号一致 | やや厳しめだが妥当 | 妥当 |
| B. 世界設定（独創性） | 9.0 / 10 | ✓ 全引用照合済 | 妥当 | 妥当 |
| C. 発展性（汎用性＋ビジョン） | 9.0 / 10 | ✓ 全引用照合済 | 妥当 | 妥当 |
| D. 技術実装（LLM3+Mem3+CodeQ2+SimCorrect2） | 8.0 / 10 | ✓ 主要箇所一致／一部位置ずれ | 妥当 | 妥当 |
| **合計** | **33.0 / 40** | — | — | **妥当（微調整余地あり）** |

**総合判定**: 引用された行番号・パス・記述内容はおおむね正確で、スコアは過大評価ではなく公平。一部に「位置のずれ」「他層の機能を spatial_demo 内のものとして語る微妙な誤認」が見られるが、結論には影響しない。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 7.0/10

#### 根拠の検証 ✓

- `examples/spatial_demo/agent.py:371-400`（message プロンプト）→ 実ファイル 371 行近辺は `place_section_text` 構築直後で、message プロンプトの実体は **およそ 372-413 行** に存在。`message`/`reasoning` のみを返す JSON 設計は **完全一致** ✓。位置の誤差は±数行で許容範囲。
- `agent.py:476-512`（action プロンプト）→ action プロンプトは実際には **行 414-512 付近** に存在。引用範囲は `decide_action` 周辺寄りだが、内容（`action`/`direction`/`memory`/`reasoning` のみ）は一致 ✓。
- `domain_packs/iss_benevolence/domain.yaml:131-156`（`auto_pressure_rules`）→ 該当行に閾値ルール（`resource_pressure>60` / `privacy_pressure>55` / `interpersonal_tension>56` / `routine_fatigue>55` / `communication_delay>58`）と "緊張↑ 協力必要性↑" 表現が **完全一致** ✓。
- `prompts/agent_observation.md` の "命令ではなく観測として記録する" → **完全一致** ✓（実ファイル 1 行目）。
- `scripts/agent_turn_runner.py:601-606`（neutral_v2「全員を協力方向へ誘導しない」）→ 実際は **行 605-609 付近**にあるが、内容は **完全一致** ✓。
- `scripts/generate_habitat_conversations.py:336-345, 444-456` → 行 336-345 に `run_condition_hint`、行 401-457 付近にプロンプト本体。"ナッジ、善性オブジェクト、持ち寄り棚…という語を出さないでください" の制約は **実在** ✓。
- `config.iss.claude.run_b.yaml:52-84` の "Memory Shelf" 等の用途示唆 → **完全一致** ✓。

#### スコア妥当性
- ルール設計の精密さ（半径ベース・占有率・閾値・観測のみ）と、ポストプロセスでの語彙制限の同居を冷静に評価しており、減点理由が引用で裏付けられている。
- ただし「移動行動」と「メッセージ内容」の両方が完全自由記述（move/stay + 自由 memory/reasoning）であり、温度・トーン辞書を agent 側に押し付けない点は **満点級**。
- `generate_habitat_conversations.py` は UI 用 **会話再生成** スクリプトであり、シミュレーション本体（`spatial_demo/simulation.py`）の自由度を必ずしも下げていない点を「コアと別レイヤー」として扱えば 7.5～8.0 でも妥当。**7.0 はやや厳しめ**だが、UI で観測者が見るのは再生成済み会話なので、評価者の論理は筋が通る。

#### 見落とし・誇張
- 見落とし: `domain.yaml` の `realism_contract`（"会話しない・沈黙する・相手を避ける状態も有効な観測として表示する" "Run Bの改善は万能にせず…ナッジの押しつけ感も残す"）が言及されていない。これは**自由度を強く保証する**設計で、A の加点材料になり得る。
- 誇張は見当たらない。

#### 検証結論
スコア 7.0 は**やや厳しめだが妥当**。realism_contract を加味すれば 7.5 まで上振れ余地あり。

---

### B. 世界設定 — 元スコア 9.0/10

#### 根拠の検証 ✓

- README "GOOD ECHO は…空間・物・ルール・会話の設計によって支えられる…" → **完全一致** ✓。
- README "ナッジは仲直りを作ったのではない。仲直りしなくても戻れる道を作った。" → **完全一致** ✓。
- `docs/ISS/experiment_design_iss_20agents_100steps.md:38-94` の HCD 20 ペルソナ → 実ファイル該当行に Amir（難民）、Aisha（義足）、Tariq（識字率低め）を含む 20 人ペルソナテーブルが **完全に存在** ✓。
- `:289-307` の HCD レビュー解決状況テーブル → public/private 二層、Crew Quarters 追加、ファトワ原則（"礼拝方向は地球に向けばOK"）など **完全一致** ✓。
- README "災害避難所、病棟、介護施設、寮、船舶…" → **完全一致** ✓。
- `objects_menu.tsv:2-12` の 10 オブジェクト → OBJ01-OBJ10（ハンドレール・メロディ、リソース・スコアボード、話しかけてOKサイン、持ち寄り棚、個室聖域マーク、モジュール移動投票パネル等）の具体設計が **完全に存在** ✓。

#### スコア妥当性
- 5 条件 + 3 補助軸（修復なし／全期間修復なし／喧嘩なし）、20 名 HCD ペルソナ、ファトワ原則準拠の宗教空間設計、転用先列挙まで揃っており、ハッカソンとしては最上位水準。
- 「仲直りしなくても戻れる道」という再フレーミングは社会科学的な独自性が高く、9.0 は妥当。
- 1.0 減点余地: 仮説検証はメッセージ語彙の単純集計に依存しており、因果推論として厳密ではない（評価者は触れていないが、満点でない理由として整合的）。

#### 見落とし・誇張
- 誇張なし。むしろ `realism_contract`（"Run Bの改善は万能にせず、短い摩擦・遅れた修復・ナッジの押しつけ感も残す"）の存在は、設計者が**ナッジ礼賛にならない自制**を働かせている点で **加点材料**だが評価には反映されていない。

#### 検証結論
スコア 9.0 は **妥当**。9.5 でも擁護可能。

---

### C. 発展性 — 元スコア 9.0/10

#### 根拠の検証 ✓

- `sim_core/domain_pack.py:57-67, 70-86, 89-94, 211-244` の `deep_merge` / `replace_tokens` / `resolve_domain_pack` → **完全一致** ✓。継承・トークン置換・YAML マージ・バリデーションの汎用化が確認できた。
- `sim_core/defaults/default_v1.yaml:27-35` の `hooks` スロット（pre_run / pre_step / pre_observation / post_observation / aggregate / feedback / post_step / export_viewer / audit）→ **完全一致** ✓。
- `domain_packs/agi_youth_japan/domain.yaml` の存在 → **完全に確認** ✓（71 ステップの別ドメインで、agents が `youth_agents.tsv` + `working_agents.tsv` の 2 ファイル合成）。
- `iss_benevolence/domain.yaml:74-99` の 20+ profile → 実数は **24 profile**（Claude smoke/full、Codex smoke/full/fast、Cursor smoke/full、各 run_a/b/c/d/e と補助軸を組み合わせ）。引用は控えめで **妥当** ✓。
- README "差し替えられるもの: 人物… 空間… イベント… ナッジ… 評価指標…" → **完全一致** ✓。
- README 内 paper draft などへの参照 → 該当ファイルが docs 下に多数存在し、参照リンク一致 ✓。

#### スコア妥当性
- `sim_core` がドメインパック汎用フレームとして実装され、`default_v1` 継承、`${pack}/${root}/${scenario}` トークン置換、バリデーション、フック 9 種、2 つ目のドメインパック実証、24 ランタイムプロファイル登録、共通スキーマでの UI フレーム再利用まで揃う。
- 「コード拡張性」と「将来展望（README の差し替え可能性 + 転用先 + paper draft）」の両面で 9.0 は **妥当**。満点でない理由: agi_youth_japan ドメインの完成度（実行ログや実験報告）が iss_benevolence ほどは整っていないため、汎用性はまだ「実証中」段階。

#### 見落とし・誇張
- 誇張なし。`source_docs` セクションで参照ドキュメント群（7 種の experiment_design）も整備されており、研究プロトタイプとしての継続性は強い。

#### 検証結論
スコア 9.0 は **妥当**。

---

### D. 技術実装 — 元スコア 8.0/10

#### 根拠の検証 ✓ / ⚠

- `llm_backends.py:152-273`（CLI クライアントの timeout / retry / JSON 抽出 / ANSI 除去）→ 行 152-275 にかけて `subprocess.run` + `TimeoutExpired` リトライ、`_clean_output`、`_extract_json_field` が **完全に存在** ✓。
- `llm_backends.py:22-41` の `LLMClientProtocol` → 行 21-40 付近に Protocol 定義が **完全に存在** ✓。
- `agent.py:515-550` のブレース深度追跡で JSON 抽出 → 実体は **行 514-553 の `_extract_json_from_text`** で `depth`/`in_string`/`escape_next` を追跡。**完全一致** ✓。
- `agent.py:676-692` の LLM 生成 memory を FIFO 蓄積 → 実体は **行 681-693** に `memory_entry = f"Step {step}: {memory_content}"` → `self.memory.append` → `if len(self.memory) > self.memory_limit: self.memory.pop(0)`。**完全一致** ✓。
- `simulation.py:568-748` の 4 フェーズ同期実行 → 実体は **行 569-748** で `step_simulation()` 内に Phase 1 (message decisions) → Phase 2 (send) → Phase 3 (action decisions) → Phase 4 (movement) が明示。**完全一致** ✓。
- `simulation.py:178-185` の `ThreadPoolExecutor` を `llm_parallelism` で制御 → 実体は **行 178-188 の `_run_in_parallel`**。**完全一致** ✓。
- `simulation.py:386-406` の `memory_reasoning.jsonl` バッチ出力 → 実体は **行 388-407 の `_log_memory_reasoning_batch`**。**完全一致** ✓。
- `simulation.py:606-618` の previous_state 凍結 → 実体は **行 609-622** で `previous_fire_exposure` / `previous_places` / `previous_place_occupancy` を step 開始時に固定。**完全一致** ✓。
- `agent_turn_runner.py:625-720` neutral_v2 構造化 → 該当範囲付近に thought / private_talk / social_post の 3 層化プロンプトが存在 ✓（位置は微差）。

#### スコア妥当性
- **LLM 利用 (3点満点)**: Ollama + 3 種 CLI 抽象化、JSON 強制 + フォールバック、リトライ・ANSI 除去・並列実行。**満点付近**。
- **メモリ設計 (3点満点)**: LLM 自身が「次ステップで覚えたいこと」を生成 → FIFO で構造化保持、`memory_reasoning.jsonl` バッチ書き出し、persona/relationship_seed/baseline_stress などの静的コンテキスト併用。**満点級**。ただし `relationship_seed` / `baseline_stress` は **`spatial_demo/agent.py` ではなく `scripts/agent_turn_runner.py` 側で扱う**。評価者は両者を一体として扱ったが、厳密には別レイヤー → ⚠ **やや誇張**。
- **コード品質 (2点満点)**: 型ヒント・TypedDict・Protocol、ファイル分割、約 3500 行（spatial_demo のみ計上で 737+407+325+328+129+897+74+606 ≈ 3503 行。「約 4000 行」は **若干切り上げ**だが範囲内）。**妥当**。
- **シミュレーション正確性 (2点満点)**: 4 フェーズ同期、previous_state 凍結。一方で `random.seed` の明示は `simulation.py` 内に**確認できず**、再現性は config と LLM CLI の決定性に依存。減点は妥当。

#### 見落とし・誇張
- ⚠ 誇張: メモリ設計の根拠に `relationship_seed` / `baseline_stress` を挙げているが、これらは `examples/spatial_demo` の agent プロンプトには直接組み込まれておらず、`scripts/agent_turn_runner.py` 系の別パイプライン（後付け観測再生成）でのみ参照される。"プロンプトに供給され、構造化メモリ" の表現はやや盛っている。
- ⚠ 見落とし: `_clean_output` の ANSI 除去（ANSI_ESCAPE_PATTERN）まで踏み込んでいるが、`response_json_field` の dotted-path 抽出機能や `start_new_session` で SIGINT 伝播を防ぐ細かい配慮は触れられていない。
- ⚠ 見落とし: `simulation.py` 内に `random` を使う処理（`_generate_random_position` 等）がありながら、シミュレーション側に明示シード初期化がないため、評価者の指摘どおり再現性は弱い。

#### 検証結論
スコア 8.0 は **妥当**。LLM 利用とメモリ設計はほぼ満点、再現性とコード規模で 1～2 点減という配分は合理的。memory 設計の根拠の一部に**位置ずれ**があるが、結論に影響なし。

---

## 総合所見

### 引用品質
- ファイルパスと行番号の引用は全体に **高精度**。±数行のずれは数件あるが、引用された記述内容はすべて実ファイル上で確認できた。捏造引用は **ゼロ**。
- 引用箇所のセクション選択は的を射ており、エビデンス・ドリブンな評価姿勢が見える。

### スコア配分の妥当性
- A (7.0) は実行コアの自由度と UI 再生成段階の制約の同居を冷静に評価。`realism_contract` を加味すれば 7.5 まで擁護可能だが、現スコアも妥当。
- B (9.0) は HCD ペルソナ・5 条件・補助軸・転用先列挙・ファトワ原則準拠の完成度を妥当に評価。
- C (9.0) は `sim_core` の汎用化、2 つ目のドメイン実証、24 profile 登録、共通スキーマ再利用まで踏まえて妥当。
- D (8.0) は LLM 抽象化と 4 フェーズ同期、LLM 自己生成メモリを高評価、`random.seed` 欠如・CLI 非決定性で減点。妥当。

### 主な指摘点
1. **位置の微ずれ**: D の `agent.py:476-512`（実体 414-512）、`agent_turn_runner.py:601-606`（実体 605-609）など、±数行のずれが散見される。引用範囲を「decide 系メソッド全体」に丸めたためと推察され、悪意ある誤引用ではない。
2. **`relationship_seed` / `baseline_stress` の所属レイヤー混同**: `spatial_demo/agent.py` のメモリ設計の根拠としてこれらを挙げているが、実際は `scripts/agent_turn_runner.py` 側のパイプライン文脈。厳密には記述を修正すべきだが、スコアへの影響は軽微。
3. **コード行数**: 「約 4000 行」は spatial_demo のみで **約 3500 行**、scripts/sim_core を含めると約 9300 行。評価者の意図する spatial_demo に限れば「約 3500-4000 行」が正確。

### 総合判定
**33.0 / 40 は妥当な評価**。微細な引用ずれと一部の根拠位置の混同はあるが、スコア自体は控えめで誇張がなく、むしろ `realism_contract` などの加点材料を見落としている。再評価しても 32-34 の範囲に収まる可能性が高い。
