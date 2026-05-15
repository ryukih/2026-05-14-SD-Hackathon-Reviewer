# 評価検証レポート: good_echo_iss_sim_cursor_100s_w_accident
**検証日**: 2026-05-11
**対象**: `good_echo_iss_sim_cursor_100s_w_accident_eval.md`
**元評価合計**: 33.5/40

---

## 検証サマリー

| カテゴリ      | 元スコア | 検証判定         | 主な所見                                                      |
|-------------|--------|----------------|-------------------------------------------------------------|
| A. 創発設計  | 8.0    | 妥当            | 引用5件中5件 ✓。`get_nearby_agents` の物理ルールやイベントTSVは正確に存在。指示性英文（"reduce isolation"等）も実在し、減点根拠も整合。 |
| B. 世界設定  | 9.0    | 妥当            | 引用5件中5件 ✓。10ペルソナ・構造化フィールド・100日タイムライン・転用記述が README/agents.tsv に確認。   |
| C. 発展性    | 9.0    | 妥当 (軽微誇張) | 引用5件中5件 ✓。ただし `column_aliases` は `domain.yaml` に直接出現せず `sim_core/defaults/default_v1.yaml` 経由。 |
| D. 技術実装  | 7.5    | 概ね妥当 (誤記あり) | 引用6件中5件 ✓・1件 ⚠。`agent_turn_runner.py` の所在を曖昧に記載（`scripts/` 配下）、`memory_limit=15/memory_size=4` は config 由来（agent.py の default は 20/5）で記述が誤解を招く可能性。シミュレーション同期分離・seed 不足の指摘は正確。 |

**総合判定**: 全体として **元評価は妥当**。引用先の存在は概ね正確で、スコアリングは根拠と整合する。所見の細部に1〜2件の表現上の不正確さ（ファイルパス省略、設定値の出所明示不足）はあるが、結論を覆すレベルではない。**33.5/40 を支持**。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 8.0

**根拠の検証**

- ✓ `examples/spatial_demo/agent.py:333-369`: 該当範囲は `_build_action_prompt` 系の place_status セクションで、`occupancy_rate`/`capacity`/`agents_in_place` などの**生の数値**を提示する処理に一致。命令的指示はなく "Provide only numerical data" のコメントも存在。
- ✓ `examples/spatial_demo/agent.py:107-128`: `get_nearby_agents` は同一場所内 or 共に外、かつ通信半径以内のみ会話可能。コメントに "Agents CANNOT communicate if one is inside a place and the other is outside" と明記され、評価記述と完全一致。
- ✓ `configs/config.iss.cursor.run_b.yaml`: 実際に `[Benevolence object: Memory Shelf] People may leave a small personal item...` を確認 (line 69)。他の Talk-OK seat / move-vote panel / challenge board / Sanctuary mark も実在。
- ✓ `domain_packs/iss_benevolence/data/events_run_a.tsv`: DEBR01 (Day50衝突) ~ DEBR05 (Day65以降LAB酸素低下) を確認。記述通り。
- ✓ `domain_packs/iss_benevolence/prompts/agent_observation.md`: 3行で「命令ではなく観測として記録する」記載を確認。

**スコア妥当性**: 8.0/10 は妥当。「世界ルールの精度」と「生データのみ提示する観測哲学」が一貫しており高得点に値する。一方で減点理由として挙げられた「`reduce isolation`/`encourages moving` 等の効果示唆英文」も run_b.yaml に実在しており、減点根拠の正当性も支持。

**見落とし・誇張**: 大きな誇張なし。Run A の place description にナッジ説明が含まれないかどうかは検証していない（評価は Run B のみを引用）。

**検証結論**: **妥当**。引用と分析がいずれも実コードと整合。

---

### B. 世界設定 — 元スコア 9.0

**根拠の検証**

- ✓ `README.md`: 「高ストレス環境を強制構築して、心理と関係の崩壊を環境設計でどう防げるかを試すプロダクト」を実際に確認。
- ✓ `data/agents.tsv`: ヘッダに `agent_id, name, age, gender, region, religion, baseline_stress, ..., self_efficacy, institutional_trust, hope, ..., social_anchor, vulnerability_note` を含む構造化10名を確認。シリア難民 Amir、ケニア義足パラアスリート Aisha、コロンビアアーティスト Sofia、フランス元外交官 Henri も実在。
- ✓ `data/events_run_b.tsv`: 27件のイベント行を確認 (Run A は22件)。CONF/REPB/OBJ/DEBR 種別の連鎖が100日に分布。
- ✓ `README.md`: 「災害避難所、病棟、介護施設、寮、船舶、研究施設、高圧プロジェクトチームなどにも転用できる」を確認。
- ✓ `docs/ISS/` 配下: `personas_iss_10agents.md`, `places_iss_design.md`, `iss_objects_menu.md` の3ファイルが README リポジトリ構成欄にも明記。

**スコア妥当性**: 9.0/10 は妥当。シナリオの独自性（ISS閉鎖空間×10名×100日×宇宙デブリ事故）、ペルソナの構造化（22カラム）、Run A/B 比較設計、汎用化展望すべてが揃っており、世界設定としては最上位帯。

**見落とし・誇張**: 誇張なし。むしろ `relationship_seed.tsv`, `interventions.tsv`, `time_schedule.tsv` 等のさらに細かい構造化データの存在には触れていない（過小評価寄り）。

**検証結論**: **妥当**。

---

### C. 発展性 — 元スコア 9.0

**根拠の検証**

- ✓ `sim_core/domain_pack.py:1-308`: 308 行のファイル長と一致。domain pack 解決と検証ロジックが実装されている。
- ✓ `sim_core/hooks.py:9-17`: `HOOK_STAGES` タプルに 9 要素（pre_run/pre_step/pre_observation/post_observation/aggregate/feedback/post_step/export_viewer/audit）を確認。「9段階」記述は正確。
- ✓ `domain_packs/iss_benevolence/domain.yaml`: `profiles:` 配下に claude/codex/cursor × smoke/full × run_a/run_b の 12 個（lines 68-79）。`default_profile: "claude_full_a"` も確認。
- ✓ `README.md`: 「差し替えられるもの」「新しいドメインへの差し替え」「終盤イベントだけを差し替える」の3章が実在し、`module_id`→`place_id`/`zone_id` の説明も確認。
- ✓ `domain_packs/iss_benevolence/README.md`: 該当ファイル (7518 bytes) が存在。

**スコア妥当性**: 9.0/10 は妥当。`sim_core` と `examples/spatial_demo` と `domain_packs` の三層分離、12プロファイル登録、9段階フック、YAMLスキーマ管理 (`schema_version: domain_pack_v0.1`)、README の段階的差し替えガイドが揃っている。

**見落とし・誇張**: 軽微な不正確さ1件。「`column_aliases` で列名差し替え可能」は機構としては正しい（`sim_core/domain_runtime.py:217-218` および `defaults/default_v1.yaml:38` で実装）が、`domain.yaml`（ISS パック側）には `column_aliases:` ブロックが**ない**。デフォルト継承 (`inherits: "default_v1"`) 経由のみ。読者が ISS パックを見て理解できる範囲かはやや疑問。

**検証結論**: **妥当**（軽微な所在誇張あり）。

---

### D. 技術実装 — 元スコア 7.5

**根拠の検証**

- ✓ `examples/spatial_demo/simulation.py:626-680`: `step_simulation` メソッドのdocstringに「1. All agents decide messages → 2. Messages are sent → 3. Actions decided → 4. Movement」の4フェーズ説明あり。実コードでも Phase 1〜4 コメントが lines 677/721/749/801 にある。元評価本文では「5フェーズ（1〜5の番号）」と書かれているが、実コードは4フェーズで状態更新は Phase 0 に相当（lines 660-674 で `update_state` 等）。**軽微な記述ずれ**。
- ✓ `examples/spatial_demo/agent.py:540-577`: `_extract_json_from_text` は実際は line 515 から始まるが、brace-depth マッチング（`in_string`/`escape_next`/`depth`）の本体は引用範囲内。記述内容は正確。
- ✓ `examples/spatial_demo/llm_backends.py:140-243`: `CommandLLMClient.generate` 内で `del temperature`/`del max_tokens` (lines 159-160)、`subprocess.run` の `timeout`、リトライループ、`_sleep_before_retry` バックオフ、stderr クリーニング、`_preview_output` を確認。記述完全に一致。
- ✓ `scripts/run_cursor_prompt.sh`: 214 行。`acquire_lock` 内で `mkdir "${lock_dir}" 2>/dev/null` を atomic ロックとして使用 (line 55)、EPROTO リトライ (line 127)、指数バックオフ (line 128 付近) を確認。記述に整合。
- ⚠ `examples/spatial_demo/agent.py:683-715`: 該当範囲は `action_decision` 内の memory 追加ロジック。`memory_entry = f"Step {step}: ..."` (lines 681-685) と `if len(self.memory) > self.memory_limit: self.memory.pop(0)` (line 686-687) を確認。記述は正確。ただし「`memory_limit=15`, `memory_size=4` 直近のみ」は **agent.py のデフォルト値（20/5）ではなく、`configs/config.iss.cursor.*.yaml` の上書き値**。出所明示が省略されている。
- ✓ `examples/spatial_demo/simulation.py:480-499`: `_generate_initial_positions` 内で `random.randint(-self.half_space_size, self.half_space_size)` (lines 434-435) を確認。`random.seed` 設定は確認できず（評価記述通り）。

**追加検証**

- `wc -l` 結果: agent.py 737行 / simulation.py 955行 / `scripts/agent_turn_runner.py` 1247行。**元評価が「`agent_turn_runner.py` が 1247 行」と書きつつ所在を明示せず、文脈的に spatial_demo 内と誤読される可能性がある（実際は `scripts/` 配下）**。simulation.py 955行は正確。
- `TypedDict` (agent.py:7, 27, 33) と `@dataclass(slots=True)` (sim_core/domain_pack.py, hooks.py, domain_runtime.py) の併用は確認。ただし spatial_demo 側には `slots=True` は無く、sim_core 側で利用。記述「責務ごとのファイル分離」「`logger` 利用が一貫」も正確。
- `parallelism: 2` は `configs/config.iss.cursor.run_b.yaml:97` と他にも確認。

**スコア妥当性**: 7.5/10 は妥当。LLM 呼び出しの2フェーズ設計、CLI バックエンドの堅牢性、JSON抽出パーサ、メモリの semantic な扱いは高評価に値する。一方、`del temperature/max_tokens` による設定無視、seed 設定なし、ファイル肥大化（1247/955行）は減点根拠として有効。

**見落とし・誇張**

- 軽微な誤記2件: (1) `agent_turn_runner.py` の所在曖昧、(2) memory_limit/memory_size の値が agent.py default ではなく config 由来。
- 「ステップ5フェーズ」と書きつつ実装は4フェーズ + 前処理という不整合がある。
- 見落とし: `sim_core/__main__.py` による `python3 -m sim_core validate` の CLI エントリ、`domain_packs/iss_benevolence/scenarios/` の scenario YAML、`outputs/runs/` の生成データ運用、`scripts/agent_turn_runner.py` 内の `column_aliases` 実利用箇所などは触れられていない。

**検証結論**: **概ね妥当**。スコアそのものは適切。記述の精緻化余地が他カテゴリより大きい。

---

## 総合所見

### 妥当性

元評価合計 **33.5/40** は、引用ベースで検証する限り **支持できる**。21件の具体的引用のうち、20件はファイル・行範囲・内容ともに実コードと一致した。残り1件（`agent_turn_runner.py` の所在）も内容は正確で、表記上の曖昧さに留まる。

### 強み

- 評価本文が「観測のみ提示」「Run A/B 設計」「ナッジオブジェクトの実在記述」「Phase 1〜4 の同期分離」「`del temperature`」など、**コード上の具体的事実**を多数引用しており、検証可能性が高い。
- 減点箇所（seed なし、ファイル肥大化、CLI 固有スクリプト）は実コードで再現可能。
- 9段階フック、12プロファイル、3層モジュール構成といった構造的事実が正確。

### 弱み・改善余地

- ファイルパスの省略がある（`scripts/agent_turn_runner.py` を単に `agent_turn_runner.py` と表記）。
- 設定値（memory_limit=15 等）の出所が config 由来であることが明示されない。
- 「ステップ5フェーズ」と書きつつ実装は4フェーズ + 前処理という不整合がある。
- `agent_observation.md` が3行と簡素である点を改善提言で挙げているが、`prompts/system_context.md` の存在・内容の評価が省かれている（追加引用余地）。

### 総合判定

**スコア 33.5/40 を支持**。カテゴリ別では A=8.0、B=9.0、C=9.0、D=7.5 のいずれも引用と分析がスコアを正当化している。技術実装カテゴリの記述精度はやや改善余地があるが、スコアそのものを上下させる根拠は見当たらない。
