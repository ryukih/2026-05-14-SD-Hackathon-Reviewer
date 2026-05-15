# 評価検証レポート: hackathon-singulab

**検証日**: 2026-05-11
**対象**: `hackathon-singulab_eval.md`
**元評価合計**: 33.5/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証判定 | 推奨スコア | 主な所見 |
|---------|---------|---------|----------|---------|
| A. 創発設計 | 8.0 | 妥当 | 8.0 | 引用は全て一致。観測器フレーミングはプロンプト・コード両側で確認。 |
| B. 世界設定 | 9.0 | 妥当 | 9.0 | 8シナリオ、83step、構造持続通貨(nat)、ストレステスト8体すべて実装裏付けあり。 |
| C. 発展性 | 8.5 | 妥当 | 8.5 | ドメインパック機構、9フックステージ、4runner重複も全て確認。 |
| D. 技術実装 | 8.0 | 妥当 | 8.0 | 3層プロンプト、3回リトライ、step_complete冪等性、構造化メモリ全て事実。 |
| **合計** | **33.5** | **妥当** | **33.5** | 引用精度は極めて高く、誇張・捏造は確認されず。 |

**総合判定**: 元評価のスコアは妥当。引用は概ね全て事実に基づいており、評価の論拠と根拠引用にズレがない。Run1=34, Run2=33で評価実施回数は2回(目標3回未達)だが、本検証では33.5を維持する根拠が十分存在する。

---

## カテゴリ別検証

### A. 創発設計 (元: 8.0/10)

**根拠の検証**: ✓ 全て一致

- `agent_observation.md:3-8` の「観測器」明示文 ✓ 一致 (実ファイル冒頭3-5行で確認)
- `run_civilization_os_llm_demo.py:926-930` の event.direction 説明 ✓ 一致 (実コード927-930行に同一文)
- `run_civilization_os_llm_demo.py:917-920` の3層分離(出力契約/世界条件/人間反応) ✓ 一致
- `run_civilization_os_llm_demo.py:976-985` の観測原則・二極化要求 ✓ 一致
- `structure_stress_agent_observation.md:8-11` の機能エージェント宣言 ✓ 一致
- `normalize_emotion` (1256-1282) による語彙固定 ✓ 一致 (1256行から確認)
- `run_closed_loop_llm_demo.py:487-654` の閉ループ実行 ✓ 一致

**スコア妥当性**: 8.0は妥当。プロンプトレベルでの「命令しない」フレーミング、コードレベルでの三層分離コメント、`normalize_*` による出力契約固定、いずれも実在し、評価ロジックと整合する。語彙正規化が自由度を制約する点を-2と捉えるのは合理的。

**見落とし・誇張**: 特に確認されず。「観測原則」「二極化を出してください」など、評価本文の引用文は実装プロンプトと逐語的に一致している。

**検証結論**: 8.0/10 維持。

---

### B. 世界設定 (元: 9.0/10)

**根拠の検証**: ✓ 全て一致 (1点軽微な省略あり)

- README の主張文「制度導入前に、シミュレーションでデバッグする」 ✓ 一致 (README冒頭)
- 8つのシナリオ表 ✓ 一致 (README内シナリオ表、コード `run_civilization_os_llm_demo.py:1492-1494` 等で参照)
- ストレステストエージェント8体 ✓ 一致 (`domain_packs/agi_youth_japan/README.md` 「受益者、支援者、企業、行政、ゲーム化探索者、監査者、市民社会、金融/物価」)
- 構造持続通貨(nat)発行ルール・ガードレール ✓ 一致 (domain_pack README で詳細記述、`structure_sustain_*` TSV群が裏付け)
- 年齢階層・100年観測 ✓ 一致 (`domain_packs/agi_youth_japan/README.md` 内、ステップ1-60月次、61-83で年/5年単位)
- 国家目的関数9次元 ⚠ 軽微な引用不整合: 実コード(`run_country_llm_demo.py:41-52`)の `OBJECTIVE_FIELDS` は9要素で、評価本文は「9次元」と述べつつ列挙は8項目のみ。最初の「国家構造の存続」が省略されている (1項目欠落の表記ミス)。
- `interesting_observations.md` 存在 ✓ 一致 (`structure_hope_family_package_83steps_panel48/` 下に存在を確認)

**スコア妥当性**: 9.0は妥当。題材独自性(AGI後の日本/家族形成/次世代記憶継承)、観測対象の幅(国家30/組織15/個人60+次世代コホート)、8シナリオの比較設計、構造持続通貨(nat)というオリジナル概念装置の実装まで揃っており、社会的意義・独自性ともに極めて高い。

**見落とし・誇張**: 国家目的関数の9次元列挙が8項目にとどまる(「国家構造の存続」が漏れている)が、評価結論には影響しない範囲の誤記。誇張・捏造は確認されず。`docs/EXPERIMENT_FACTS_EVIDENCE.md`、`docs/SUSTAINABLE_REWARD_EXPLAINER.md`、参考実装側の`examples/spatial_demo/` など追加根拠への言及が欠落している点はむしろ控えめ。

**検証結論**: 9.0/10 維持。

---

### C. 発展性 (元: 8.5/10)

**根拠の検証**: ✓ 全て一致

- `sim_core/domain_pack.py:37-67` `ResolvedDomainPack`/`load_yaml`/`deep_merge`/`replace_tokens` ✓ 一致 (実コード37-90行)
- `validate_domain_pack` (158-208) 必須ファイル・列・エイリアス検証 ✓ 一致
- `sim_core/defaults/default_v1.yaml:1-46` パック定義 ✓ 一致 (47行で完結、prompts/viewer/hooks/column_aliasesすべて存在)
- `domain_packs/agi_youth_japan/domain.yaml:1-85` 継承・prompts/viewer/hooks/scenarios/column_aliases完全分離 ✓ 一致 (85行で完結)
- `sim_core/hooks.py:7-81` 9段階フック仕様 ✓ 一致 (`HOOK_STAGES` に pre_run/pre_step/pre_observation/post_observation/aggregate/feedback/post_step/export_viewer/audit の9段階)
- 4runner重複 ✓ 一致 (`run_civilization_os_llm_demo.py`/`run_country_llm_demo.py`/`run_organization_llm_demo.py`/`run_policy_planner_llm_demo.py` 各々に `extract_json_from_text` / `is_codex_model` / `run_codex` / `run_claude` が独立実装されていることをgrepで確認)
- 各種docs (ドメインパック差し替えガイド/製品パッケージ等) ✓ 一致 (`docs/製品パッケージ/01_製品コンセプト/`、`02_実装仕様/`、`03_LLM観測仕様/`、`04_発表シナリオ/` 配下を実確認)

**スコア妥当性**: 8.5は妥当。ドメインパック機構(`domain_pack.py`, `hooks.py`, `default_v1.yaml`)が形式的かつ宣言的に整備され、`inherits` による継承・`scenarios/*.yaml` での差し替えが第一級概念として実装されている。一方で、本線4runnerの同型コード重複(LLMクライアント未抽象化)が-1.5の制約として妥当に評価されている。

**見落とし・誇張**: 評価本文の `run_civilization_os_llm_demo.py:1084-1162` という行範囲は厳密には `extract_json_from_text` から `run_claude` 末尾までで1042-1162が正確。内容(重複実装)の指摘は正確だが、行番号がやや`run_codex`関数開始位置寄りにずれている。

**検証結論**: 8.5/10 維持。

---

### D. 技術実装 (元: 8.0/10)

#### LLM3 (LLM利用)

**根拠の検証**: ✓ 全て一致

- 3層プロンプト構造 ✓ 一致 (`run_civilization_os_llm_demo.py:917-920`)
- JSON抽出多段ロジック ✓ 一致 (1042-1069行、外側JSON→result→raw text→`{` 検索フォールバック)
- リトライ最大3回 ✓ 一致 (1123-1162行内、`for attempt in range(1, 4):`)
- Claude CLI / Codex CLI 切替 ✓ 一致 (`is_codex_model`, `run_codex` 実装あり)
- `--max-budget-usd` / `--timeout` 制御 ✓ 一致 (`run_claude` 内コマンド構築で確認)

#### Mem3 (メモリ設計)

**根拠の検証**: ✓ 全て一致

- 個人エージェントの `memory_update`/`carryover_concern`/`side_effect`/`adjustment_request`/`fairness_perception`/`policy_fatigue` 持ち越し ✓ 一致 (`run_civilization_os_llm_demo.py:842-865` の `compact_previous` で確認)
- 国家・組織層の前ステップ記憶保持 ✓ 一致 (`run_organization_llm_demo.py:175-216`, `run_country_llm_demo.py:186-202`)
- 次世代コホート継承係数(希望/不信/連帯) ✓ 一致 (`run_civilization_os_llm_demo.py:627-712` の `build_generation_agents` 内で `hope_inheritance`/`distrust_inheritance`/`solidarity_inheritance` から初期評価生成)

#### CodeQ2 (コード品質)

**根拠の検証**: ✓ 全て一致

- 1705行級の長大ファイル ✓ 一致 (`run_civilization_os_llm_demo.py` 実測1705行、他runnerも593-749行)
- 関心の分離が部分的 ✓ 一致 (同一ファイル内に prompt組立/JSON処理/正規化/I/O が同居)
- dataclass・型ヒント・docstring ✓ 一致 (`sim_core/domain_pack.py` `@dataclass(slots=True)`、`hooks.py` 同様)

#### SimCorrect2 (シミュレーション正確性)

**根拠の検証**: ✓ 全て一致

- `step_complete`/`log_skipped_step`/`run_log.tsv` による冪等再開 ✓ 一致 (`run_closed_loop_llm_demo.py:122-157` 区間に `step_complete`、`log_skipped_step` 実装)
- 正規化 (`normalize_emotion`/`normalize_action_category`/`normalize_side_effect`/`normalize_adjustment_request`) ✓ 一致 (1256-1316行内)
- step単位順次、step内 `ThreadPoolExecutor` 並列 ✓ 一致 (`run_stateful_generation` が1360行から、`ThreadPoolExecutor` 1390行で使用)

**スコア妥当性**: 8.0は妥当。LLM利用は3層プロンプト+JSON契約+3回リトライで堅牢、メモリ設計は意味ラベル付き多層構造で秀逸、シミュレーション正確性は冪等再開実装で良好、ただしコード品質は1700行級のモノリスで分離度が中程度、という配点に違和感はない。

**見落とし・誇張**: 評価本文の `run_civilization_os_llm_demo.py:1084-1162` がやや実際の範囲(1042-1162)と異なるが、内容指摘は正確。`run_civilization_os_llm_demo.py` 1705行という記述も実測一致。誇張は確認されず。

**検証結論**: 8.0/10 維持。

---

## 総合所見

### 評価の信頼性

引用精度は極めて高い水準にある。全カテゴリで根拠引用文・行番号がほぼ実コードと逐語一致しており、評価者は実際にソースを読んで論拠を組み立てたと判断できる。誇張や捏造、確認できない事実主張は本検証では見つからなかった。

### 軽微な指摘事項

1. B(世界設定)で「国家目的関数の9次元」と述べつつ列挙は8項目(最初の `国家構造の存続` が漏れている)。9を主張するなら9つ列挙すべきだが、評価結論には影響しない範囲の誤記。
2. C/D で `run_civilization_os_llm_demo.py:1084-1162` という行範囲が `1042-1162` のほうが冒頭からの引用として正確。
3. 実施回数は仕様(3回)に対し2回で済まされている。Run1=34, Run2=33と+1のブレ幅にとどまるため最終33.5の信頼性は実用上高いが、3回目を回せば標準偏差をさらに圧縮できた可能性がある。

### 提出物の評価妥当性

総合33.5/40は妥当な水準と判断する。本提出物は、

- LLMを「観測器」として扱う設計思想がプロンプト・コード・ドキュメント全レイヤーで徹底
- ドメインパック分離(`sim_core/` + `domain_packs/`)を第一級概念として配置
- 国家×組織×個人×次世代+政策プランナーの閉ループ実装
- 構造持続通貨(nat)、制度ストレステストエージェント、100年観測など独自概念の実装着地
- 意味ラベル付き構造化メモリ(memory_update/carryover_concern/side_effect/fairness_perception/policy_fatigue)
- `step_complete`による冪等再開、3回リトライ等の堅牢性

を備え、ハッカソン提出物としては非常に完成度が高い。元評価の「優れている点」「改善点」の列挙はバランスがよく、4runnerのLLMクライアント抽象化未整理・モノリス化という指摘も正鵠を射ている。

### 最終判定

**スコアは33.5/40を維持する。**
