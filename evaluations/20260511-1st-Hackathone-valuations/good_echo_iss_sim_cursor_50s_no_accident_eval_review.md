# 評価検証レポート: good_echo_iss_sim_cursor_50s_no_accident
**検証日**: 2026-05-11
**対象**: `good_echo_iss_sim_cursor_50s_no_accident_eval.md`
**元評価合計**: 34.0/40

---

## 検証サマリー

| カテゴリ           | 元スコア | 検証結果         | 引用妥当性 | 妥当性 |
|------------------|---------|-----------------|-----------|--------|
| A. 創発設計        | 8.0     | 検証OK           | ✓         | 妥当   |
| B. 世界設定        | 9.0     | 検証OK           | ✓         | 妥当   |
| C. 発展性          | 9.0     | 検証OK           | ✓         | 妥当   |
| D. 技術実装        | 8.0     | 検証OK           | ✓         | 妥当   |
| **合計**          | **34.0**| **34.0**        | -         | **妥当** |

**総合判定**: 元評価のスコア 34.0/40 は、引用された全ソース根拠（agent.py、simulation.py、llm_backends.py、sim_core/hooks.py, sim_core/domain_pack.py、各 YAML/TSV、README.md）と完全に整合する。誇張または事実誤認は見出されず、減点要因（モノリス構造・乱数シード未確認・メモリのフラット構造）もすべて実態と一致する。スコア改変は不要。

---

## カテゴリ別検証

### A. 創発設計 (rules × freedom) — 元評価 8.0/10

**根拠の検証**:
- ✓ `agent.py:282-301` の `_build_fire_section` に「Only quantitative data is provided ... No qualitative descriptions (e.g. "dangerous", "evacuate") are included.」のコメント、および `position / intensity / radius / distance` のみを文字列に組み立てる実装を確認。実体は L107-L127 にあるが、メソッド全体を指す引用として許容範囲。
- ✓ `agent.py:317-400` の `create_message_prompt` で「Decide what message you want to send to nearby agents」「You can share your observations, experiences, or thoughts about the places and situation.」を確認。「逃げろ」「警告せよ」のような指示文は無し。
- ✓ `agent.py:116-139` の `get_nearby_agents` に「Both outside places, OR Both in the SAME place」のロジックを `same_area` 変数として実装。コメントでも通信ルールが明示。
- ✓ `configs/config.iss.cursor.run_b.yaml` の place description に `[Benevolence object: move-vote panel]`, `[Benevolence object: Talk-OK seat]`, `[Benevolence object: Memory Shelf]`, `[Benevolence object: challenge board]` を確認。引用行番号(52,59,66)は概ね一致。
- ✓ README.md に「単なる雰囲気ではなく、…衝突後に修復会話が起きたか、…どのナッジが、どのタイミングで効いたか」を確認。

**スコア妥当性**: 8.0 は妥当。世界ルール（場所 capacity、通信半径、4方向移動、火災知覚）は数値で純化され、プロンプトは中立。一方で `welcomes conversation` などのやや誘導的表現が place description に含まれる点（実物確認済）が原評価で減点理由として明示されており、評価ロジックも整合している。

**見落とし・誇張**: 大きな誇張は無し。「[Benevolence object: ...] welcomes conversation」が「やや誘導的」とした原評価は実物そのままで妥当な指摘。

**検証結論**: ✓ 妥当。スコア維持。

---

### B. 世界設定 (originality) — 元評価 9.0/10

**根拠の検証**:
- ✓ README.md に「避難所、病棟、介護施設、寮、企業の高圧プロジェクト環境では、…人の判断や関係性が壊れていきます。…このプロジェクトでは、その崩壊過程をLLMエージェントで再現し、環境側の介入…をA/B比較します」を確認。
- ✓ `configs/config.iss.cursor.run_a.yaml` の personas 配列に、中国農村工場労働者(Chen Wei)、インド母親 Priya、バングラデシュ工場労働者 Fatima、シリア難民 Amir、ベトナム料理人 Linh、日本退職者 Makoto Tanaka、ケニアのパラアスリート Aisha、コロンビアの芸術家 Sofia、米国学生 Marcus、元フランス外交官 Henri の 10 名を確認。
- ✓ `domain_packs/iss_benevolence/data/agents.tsv` のヘッダに `agent_id, baseline_stress, self_efficacy, institutional_trust, hope, vulnerability_note` 等を確認。原評価の「ベースラインストレス・自己効力感・希望度・脆弱性ノートまで定量化されたペルソナ表」は正確。
- ✓ `domain_packs/iss_benevolence/data/objects_menu.tsv` に「ハンドレール・メロディ」「宇宙廃棄物サウンドボックス」「リソース・スコアボード」「今日の地球投票」等を確認。
- ✓ README.md に「「善性」は性格だけで決まるものではなく、環境と運用によって育つのではないか、という仮説を検証します」を確認。

**スコア妥当性**: 9.0 は妥当。ISS閉鎖空間×多文化的訓練未経験者10名というハッカソン題材として独創性が高い。READMEで明示される避難所・病棟・介護施設・船舶へのドメイン転用射程も実装で裏付けられている。

**見落とし・誇張**: 無し。引用は全て一次資料と一致。

**検証結論**: ✓ 妥当。スコア維持。

---

### C. 発展性 (modularity + vision) — 元評価 9.0/10

**根拠の検証**:
- ✓ `sim_core/hooks.py:1-17` で 9 ステージ (`pre_run, pre_step, pre_observation, post_observation, aggregate, feedback, post_step, export_viewer, audit`) が `HOOK_STAGES` タプルとして定義され、`normalize_hooks` (L40-L80) で外部定義フックを取り込めることを確認。
- ✓ `sim_core/domain_pack.py:158-208` の `validate_domain_pack` に必須キー、必須ファイル、エイリアス、population_weight 列のバリデーションを確認。
- ✓ `domain_packs/iss_benevolence/domain.yaml:67-79` に claude/codex/cursor の 3 backend × smoke/full × run_a/run_b = 12 プロファイルが登録済みであることを確認。
- ✓ `examples/spatial_demo/llm_backends.py:289-325` の `create_llm_client` で `provider` 文字列に応じて `OllamaClient` / `CommandLLMClient` を切替えていることを確認(「ollama」と「command/cli」)。
- ✓ README.md に「人物: ペルソナ…/ 空間: 部屋…/ イベント: 睡眠不足、作業遅延…/ ナッジ: 感謝、静穏…」「災害避難所なら、`gym_floor`、`medical_corner`、`food_line`、`quiet_area`」を確認。
- ✓ 4 層構成 (`sim_core/`, `domain_packs/`, `examples/spatial_demo/`, `scripts/`) はファイルシステムで実在確認済。

**スコア妥当性**: 9.0 は妥当。コード拡張性（9ステージ hook、12 プロファイル、複数 LLM backend、TSV/YAML 完全外部化）と将来展望（READMEで5つの代替シナリオ、避難所等への TSV 単位差替手順を明示）の両軸が高水準。

**見落とし・誇張**: 無し。

**検証結論**: ✓ 妥当。スコア維持。

---

### D. 技術実装 (LLM3 + Mem3 + CodeQ2 + SimCorrect2) — 元評価 8.0/10

#### LLM 利用 (3点満点換算)

**根拠の検証**:
- ✓ `agent.py:317-400` の `create_message_prompt` (Phase1: 位置情報なし) と続く `create_decision_prompt` (Phase2: 位置情報あり) で 2 段階プロンプトを確認。
- ✓ `agent.py:514-549` の `_extract_json_from_text` でブレース深度・文字列リテラル・エスケープを考慮した JSON 抽出を確認(引用 515-550 行はほぼ正確)。
- ✓ `llm_backends.py:152-274` の `CommandLLMClient.generate` で `max_retries`, `retry_backoff_seconds`, `timeout_seconds`, ANSI 除去 (`_clean_output`), stdout フィルタを確認。
- ✓ `simulation.py:178-185` の `_run_in_parallel` で `ThreadPoolExecutor` と `executor.map` (順序保存) を確認。

**評価**: 妥当。2段階分離、JSON堅牢パース、リトライ、ANSI除去、並列実行と運用要素を網羅。

#### メモリ設計 (3点満点換算)

**根拠の検証**:
- ✓ `agent.py:676-687` で `memory_content = decision.get('memory', '')` を `Step N:` プレフィックスで蓄積し、`memory_limit` で打切る実装を確認。
- ✓ `memory_limit`, `memory_size`, `message_history_limit`, `message_context_size` の 4 パラメータ管理と `received_messages` の別バッファ運用も確認 (L58-L82, L184-L192, L715-L721)。

**評価**: 妥当。「フラット構造で人物別階層化なし」という減点も実体通り。

#### コード品質・シミュレーション正確性 (2点+2点満点換算)

**根拠の検証**:
- ✓ `simulation.py` 920 行 (`wc -l` で確認)。
- ✓ `simulation.py` 内に空間 (places)、火災 (`fire_states / fire_configs` L82-L99)、経済層 (`economy_enabled / structure_credit_name / automation_enabled` L103-L116, `_process_economy_opening` L257) が混在することを確認。「単一クラス Simulation に混在」は実体と一致。
- ✓ `simulation.py:590-769` で 4 フェーズ (Phase1: message decide → Phase2: send → Phase3: action decide → Phase4: move) の同期実行ループを確認。
- ✓ TypedDict・dataclass・logger・schema validation は `utils.py` / `sim_core/domain_pack.py` で確認。
- ✓ 乱数シードについて `simulation.py` と `main.py` を全文検索したが `random.seed` / `np.random.seed` / `seed` 設定は確認できず。原評価の指摘は正確。

**評価**: 妥当。

**スコア妥当性**: 8.0 は妥当。LLM/Memory は高水準だが、920行モノリス・経済+火災の同居・シード未設定の三点が原評価で正しく減点されている。

**見落とし・誇張**: 無し。`simulation.py:24-176` を経済+火災+空間の混在範囲として挙げているが、火災実体は L82-L99、経済は L103-L116 で混在は正しい。

**検証結論**: ✓ 妥当。スコア維持。

---

## 総合所見

### 一次資料との整合性

全ての引用 (agent.py, simulation.py, llm_backends.py, sim_core/hooks.py, sim_core/domain_pack.py、各 YAML/TSV、README.md) について現物との照合を行った結果、**引用範囲はおおむね正確で、誇張・事実誤認は検出されなかった**。引用行番号は数行のずれが散見されるが (例: agent.py:282-301 は実際は L107-L127 を含むメソッド範囲を指している、agent.py:515-550 は L514-L549) 、文脈と意味は完全に一致しており、評価の妥当性に影響しない。

### スコア妥当性総括

- A 8.0: 「中立プロンプト+環境アフォーダンス埋込」という設計思想と「やや誘導的な description 文言」のトレードオフを正しく評価。
- B 9.0: 多文化10名×ISS閉鎖空間×ナッジ理論検証のオリジナリティ・社会的射程は実物が明確に支える。
- C 9.0: 4層分離・9ステージhook・12プロファイル・複数LLM対応・避難所への移植性は実装+READMEで完全に裏付けられる。
- D 8.0: 2段階プロンプト・JSON堅牢化・retry/ANSI 除去・4フェーズ同期は実装に存在。減点要因（920行モノリス、経済+火災混在、シード未設定、フラットメモリ）も全て実体と一致。

### 改善点コメントの妥当性

原評価が指摘した 5 つの改善提案 (経済層分離、`config.simulation.seed` 導入、構造化メモリ、raw data 化、定量結果掲載) は、いずれもコード実態と本気の改修方向として有効。特に乱数シードの欠落は A/B 比較を売り物とする本プロジェクトにとってクリティカルな指摘。

### 最終判定

**元評価合計 34.0/40 は妥当。スコア改変は不要**。引用根拠は一次資料で全て検証可能であり、減点根拠も実体と整合する。原評価は概念設計・コード実装・将来展望のすべてをバランスよく評価できており、信頼性が高い。
