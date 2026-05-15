# 評価検証レポート: 2d-multi-places-simulation-on-fire

**検証日**: 2026-05-11
**対象**: `2d-multi-places-simulation-on-fire_eval.md`
**元評価合計**: 25.0/40

## 検証サマリー

| カテゴリ | 元スコア | 検証判定 | 推奨スコア帯 | 主な指摘 |
|---------|---------|----------|--------------|---------|
| A. 創発設計 | 4.0/10 | 妥当 | 3.5〜4.5 | ルール優位の指摘は正確。引用も実装と一致 |
| B. 世界設定 | 6.0/10 | 妥当 | 5.5〜6.5 | 二重火災・定員制約の指摘は正確。独自性評価も妥当 |
| C. 発展性 | 7.0/10 | 概ね妥当 | 6.5〜7.5 | モジュール分離・設定外部化は正しい。Future Work 不在も実証 |
| D. 技術実装 | 8.0/10 | やや誇張 | 7.0〜7.5 | 「fast/smart 二段モデル切替」は config.yaml で無効化されており、断定はやや過大 |

**総合判定**: 25.0/40 は概ね妥当だが、D の根拠の一部（hybrid モデル）が現行設定下では稼働しておらず、0.5〜1.0 pt 程度の上振れ要因がある。実質スコアは 24.0〜24.5 /40 が妥当。

## カテゴリ別検証

### A. 創発設計（元: 4.0/10）

**根拠の検証**:
- ✓ `agent.py:1585-1857` — `decide_action_by_rule` は 1585 行目に定義、次関数は 1859 行目。範囲は正確。
- ✓ `agent.py:1020-1034` — `_build_evacuation_decision_guidance` は 1020 行目から、引用文 "Computed optimal escape: {DIRECTION}. Move in this direction to maximize distance from fire." は実装どおり完全一致。
- ✓ `agent.py:1048-1056` — `_build_fundamental_purpose` は 1049 行目から、"Priorities: 1. Survive... 2. Warn others... 3. Coordinate..." も正確。
- ✓ `agent.py:1124-1133` — 「1. 火災位置 2. 強度と半径 3. あなたの避難方向 4. 安全な方向の推奨」4 段構造は 1127〜1133 行に存在。
- ✓ `agent.py:1281-1284` — `STAY HERE unless BAR SAFETY STATUS shows this bar as DANGER` 等の指示語も実装どおり。
- ✓ README "安全判断はルール、曖昧な社会的判断はLLMが担当" — 完全一致。

**スコア妥当性**: LLM 呼び出しは「曖昧ゾーン × 火災メッセージあり」二重ゲート（agent.py:1840-1847）通過時のみ。LLM 呼び出し時も "Computed optimal escape" で最適方向を予告するため自由度は構造的に制限される。世界ルール軸≈8、行動自由度軸≈3。4.0 は厳しめだがルブリック「3-4: thin+scripted」「5-6: one weak」の中間で妥当。

**見落とし・誇張**: 「LLM はテキスト整形器」は強い表現だが、ambiguous-zone 分岐と日本語メッセージ生成では実質的 LLM 推論が残るため、より正確には "判断空間を絞られたコンテキスト生成器"。改善提言部に同旨記載あり、致命的誤誘導ではない。

**検証結論**: 妥当（4.0 は妥当。3.5〜4.5 のレンジ）。

### B. 世界設定（元: 6.0/10）

**根拠の検証**:
- ✓ `config.yaml:21-34` — places セクション（実際 21-33 行）に left_bar/right_bar、capacity 12/10 が存在。1 行ズレだが実害なし。
- ✓ `config.yaml:112-124` — fires セクション（実際 112-122 行）。`fire_1: start_step=35, intensity=0.8, radius=15` と `fire_2: start_step=45, intensity=0.5, radius=8` は実装どおり。
- ✓ README "フィールドに複数の場所...複数の火災イベントが発生する" — 一致。
- ✓ `simulation.py:511-571` — `get_fire_info_for_agent` は 510 行目開始、半径内知覚＋バーセンター警報を 547-571 行に実装。
- ✓ README "創発的役割分化..." — 記載確認。
- ✓ 「設計上の役割付与は確認されず」— `grep` で `role|指揮者|leader|coordinator` 該当 0 件。

**スコア妥当性**: 15 エージェント、ダブル火災、定員 12/10、120 ステップは多層的設定。バー単位アラームによる情報非対称性は構造的に発生。典型題材ながら良設計。6.0 は素直。

**見落とし・誇張**: バー単位アラーム（building fire alarm の物理アナロジー）は世界ルール上独自性のある工夫だが根拠リストに未記載。+0.5 pt 可能性。

**検証結論**: 妥当（6.0 は妥当。バー単位アラームを加点要素として認めるなら 6.5 も許容）。

### C. 発展性（元: 7.0/10）

**根拠の検証**:
- ✓ `agent.py`/`simulation.py`/`ollama_client.py`/`llm_router_sync.py`/`utils.py`/`visualization.py` — 全ファイル存在。
- ✓ `utils.py:17-24` — `PlaceConfig` TypedDict（17-24 行）正確。`FireConfig` も 7-13 行に存在。
- ✓ `config.yaml:21-33` および 110-124 — 外部化は実装どおり。
- ✓ tests/ — 9 個の test_*.py + conftest.py = 10 ファイル。README 表合計 62 テスト（7+7+5+10+7+11+6+4+5）一致。
- ✓ README に Future Work セクションなし — `grep` 0 件。
- ✓ `agent.py` 2083 行 — `wc -l` で確認、正確。

**スコア妥当性**: モジュール構造・YAML 外部化・TypedDict・62 テストの組合せは拡張性に明確に寄与。一方 agent.py 肥大化・Future Work 不在は減点。7.0 は妥当。

**見落とし・誇張**: なし。指摘も実証可能で公平。

**検証結論**: 妥当（7.0 は妥当）。

### D. 技術実装（元: 8.0/10）

**根拠の検証**:
- ✓ `ollama_client.py:52-78` — `warmup()` は 52 行目から、keep_alive を使ったロード実装が 78 行目まで存在。
- ✓ `ollama_client.py:102-104` — Qwen 向け `/no_think\n` プレフィックスは 103 行目に実在。
- ⚠ `agent.py:170-206` — `_get_llm_client` は 170-206 行に存在。範囲正確。**重要注意**: `config.yaml:87` で `use_hybrid_model: false`、`simulation.py:127-144` で `smart_llm_client = None` となり、**この二段ルーティングは実装はされているが本番運用では稼働していない**。評価本文の「fast/smart 二段モデル切替で...推論コストを最適化」は条件付きで真。
- ✓ `agent.py:1329-1368` — `_strip_think_tokens`（1329 行目）と `_extract_json_from_text`（1342 行目）は実装と完全一致。
- ✓ `agent.py:96-102, 304-350` — `known_fires` 初期化と corroboration ロジック（`source`、`confirmed_count`、`reporting_agents` ユニーク追跡）は実装どおり。
- ✓ `agent.py:276-291` — 境界外座標フィルタ（276-283）と radius 欠落フィルタ（284-291）は実装どおり。
- ✓ `agent.py:354-377` — `STALE_STEPS=8` 減衰は 354-377 行に正確に実装。
- ✓ `simulation.py:195-231` — `_canonical_fire_id` は 195-231 行、位置ベース正規 ID マッピング。完全一致。
- ✓ `simulation.py:665-696` — `step_simulation` は 665 行目開始、4 フェーズパイプライン実装。範囲概ね正確。
- ✓ `simulation.py:858-860` — `bar_occupancies` スナップショットは 858-860 行に正確に実装。
- ✓ `simulation.py:114, config.yaml:101` — `parallel_workers` は両方とも正確。

**スコア妥当性**: メモリ設計（信頼度階層 + 減衰 + 幻覚フィルタ + ユニーク報告者追跡）は本提出群中でも極めて精密。Canonical Fire ID は秀逸。一方で「fast/smart 二段モデル切替」が現行 config では無効である点は減点要素として認識すべき。8.0/10 はややオプティミスティック、7.0〜7.5 が正確。

**見落とし・誇張**:
- 誇張: 二段モデル切替を無条件かつ現役機能のように描写しているが `use_hybrid_model: false` で単一モデル運用。-0.5 pt 程度。
- 軽微: 評価本文の `llama3.2:3b/qwen3.5:4b` 表記は agent.py コメント由来。実際の config は `llama3.2-masa-primary:3b`、`qwen3.5-masa-optimized:4b` の custom 名。事実関係同一だが厳密性に欠ける。

**検証結論**: やや過大（8.0 はやや甘い。7.0〜7.5 が妥当。減点幅 0.5〜1.0 pt）。

## 総合所見

検証の結果、25.0/40 は概ね妥当である。引用された 18 件の file:line のうち致命誤りはゼロ件、行数ズレは ±2 行以内に収まり、本質的な実装乖離もゼロ件。一方、D カテゴリの「fast/smart 二段モデル切替」は実装上は存在するが現行 `config.yaml` 設定では稼働しておらず、無条件断定はやや過大評価である。

A の「LLM はテキスト整形器」という強い表現は、二重ゲート（agent.py:1840-1847）と "Computed optimal escape" プロンプト（agent.py:1031-1033）の実装事実から十分に正当化できる。総評の改善提言（指示語排除、数値データのみ提示）は具体的かつ建設的で、ハッカソンの「創発性発現」目的に対する設計上のトレードオフを的確に指摘している。

調整後の妥当な合計スコアは 24.0〜24.5/40 のレンジ（D を 7.5 に下方修正）。原評価 25.0 はその上限近傍に位置し、実害のない範囲の上振れに留まる。

**推奨アクション**: 評価は技術的に正確、ルブリック適用も妥当、根拠引用も実装と整合。微修正（D の二段モデル運用状態の明示）を加えれば申し分ない品質。
