# 評価レポート: shibuya-sim

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

渋谷駅周辺 500m 四方を舞台に、06:00-22:00 の 16 時間を 10 分刻み 96step で進める都市流動シミュレーション。OpenStreetMap から取得した実 POI（店舗・施設約 100 件）を地理空間として配置し、200 体の LLM エージェント（gpt-4o-mini）を動作させる。エージェントは年齢×性別×職業×アーキタイプ（OL、エンジニア、高校生、観光客等 12 種）から生成された個別ペルソナを持ち、事前ウォームアップで類似 10 体＋ランダム 5 体との初期関係性を構築する。主要メカニズムは「7 ニーズ×2 層の副作用方式」「5 層の会話開始確率モデル」「POI 滞留・移動コスト」「2 時間バケットの環境・イベント層」を多層的に組み合わせる。SNS 層あり（condition B）／なし（condition A）の 2 条件 ×2 seed の対照実験設計を採用し、物理距離と情報伝搬の関係を計測する。テーマは、現実都市の地理的・社会的制約下における集合的注目、規範発話、架空 POI 言及などの創発現象の検出。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  9   |  9   |  -   |    9.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  8   |  8   |  -   |    8.0    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    |  35  |  35  |  -   |  **35.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 9.0/10

#### 評価
世界ルールの設計精度と行動の自由度の両軸ともに高水準で両立している。世界側は「7ニーズ×2層の副作用ルール」「5層会話開始確率」「POI滞留・移動コスト」「環境バケット＋イベント」「事前ウォームアップで構築される関係性」「SNS層の物理距離メタ計測」など多層的な制約と圧力を厳密に定義している。一方エージェントには、位置・距離・近傍POI・近傍人物・ニーズの自然言語表現・記憶など「生データと制約」のみを渡し、行動（移動先、発話、SNS投稿、中期目標）は完全にLLMの判断に委ねている。プロンプト中の「prefer to move」「typical dwell range」等は軽い記述的ヒントに留まり指示ではない。創発ポテンシャルは設計上極めて高い。

#### 根拠
- `src/needs.py:36-65` — POI種別ごとの `_POI_NEED_DELTA` と `_PER_STEP_DRIFT` がニーズを副作用として更新する明確な世界ルール
- `src/conversation_prob.py:79-125` — 関係性×トリガー×性格×文脈×ニーズの5層モデルで会話確率を厳密計算
- `src/simulation.py:217-308` — `build_prompt` で渡されるのは距離・最終発話・ニーズ自然文・記憶であり、行動指示ではない
- `src/simulation.py:283-291` — schema_fields は出力の型のみを規定（"action": "move"|"stay" 等）し、内容を指定しない
- `README.md` — 「`--internet` でSNS層あり(condition B)、`--no-internet` でなし(condition A)」と対照群を明示し、創発検出スクリプト `detect_emergence.py` を別途持つ
- `src/simulation.py:329-336` — 「Choose stay when … prefer to move」というやや方向付けのある一文が含まれる（減点要因の軽微なヒント）

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
題材は「渋谷駅周辺500m四方の流動人口」という極めて具体的な実地設定で、OSMから取得した実POI（`pois.geojson`、約100施設）を地理データとして用い、人口分布は国勢調査citation付きでハンドエンコードされている。研究的問いは「SNS層の有無が物理空間での集合的注目・架空地名生成・規範発話などの創発に与える影響」というメディア社会学・情報拡散の中核的トピックを直接モデル化している。デモ的な抽象都市ではなく、社会科学研究としてのオリジナリティと実問題への接続が極めて明瞭。

#### 根拠
- `config/demographics_shibuya.yaml:1-30` — 「Shibuya station 500m square (35.6595, 139.7005), weekday 15:00-22:00 JST」と空間・時間ターゲットを明示、citation `[R][C][T][S][P]` 付きの人口分布
- `config/archetypes.yaml:9-180` — 12アーキタイプ（OL／IT／高校生／大学生／観光客／文化系等）と渋谷駅前の流動人口比率を整合させた重み
- `config/config.yaml:25-37` — `conditions: A_physical_only / B_physical_internet` の対照群設計
- `scripts/detect_emergence.py:11-15` — 「L1 fiction（架空POI）／L1 attention（集合注目）／L3 norms（規範発話）」の3指標で創発を測る研究的設計
- `data/pois.geojson` — 40KBの実OSM POIキャッシュが同梱され実地性が担保される

---

### C. 発展性 — 最終: 8.0/10

#### コード拡張性
モジュール分離が極めて明確。`src/agent.py`（データクラス）、`src/simulation.py`（メインループ）、`src/llm_client.py`（プロバイダ抽象）、`src/needs.py`、`src/conversation_prob.py`、`src/geo.py`、`src/timeline.py` と単一責任で分割されており、新規ニーズ・会話レイヤ・POI種別の追加が局所改修で完結する。設定は8種のYAMLに完全外部化（`config.yaml/archetypes/demographics/environment/events/conversation/personas/personas_index`）。`llm_client.get_client` は provider 文字列で分岐し、`llm.fleet` 設定で tier ごとに別LLMを混在させるスキャフォールドも既に用意されている。

#### 将来展望
README のトップに「プロジェクト概要（ここはご自分で記述してください）」というプレースホルダが残されており、研究の問いや具体的将来計画は明文化されていない。一方で、コードレベルでは方向性が読み取れる：`config.llm.fleet`（gpt-4o-mini 60%/gemini-flash 30%/claude-haiku 10%の混在艦隊）は `NotImplementedError` で未実装、`scripts/detect_emergence.py` は架空地名・集合的注目・規範発話の3層指標を準備済み、`aggregate_runs.py` は条件比較を実装済み。明文化された将来展望文書がない分、ロードマップ提示としては減点要因。

#### 根拠
- `src/llm_client.py:111-145` — provider/model で動的にクライアントを切り替えるファクトリ
- `config/config.yaml:53-66` — `llm.fleet` に3プロバイダ×3tierの混在設定がスキャフォールドされている
- `src/llm_client.py:117-119` — `if mode != "single": raise NotImplementedError`（fleetモードは未実装）
- `README.md` — 「`<!-- # プロジェクト概要（ここはご自分で記述してください: 研究の問い、対照群A/B、創発指標などの要点）-->`」の冒頭プレースホルダが残置
- `scripts/` 配下に `generate_pool.py / warmup_relationships.py / aggregate_runs.py / detect_emergence.py / estimate_cost.py / build_personas_index.py` 等、パイプライン各段階のツールが揃う

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
`OpenAIClient`/`OllamaClient` が Protocol で統一インターフェース。`format_json=True` で `response_format=json_object` を指定し構造化出力を強制、コードフェンス／前置プロセを許容する `parse_decision` が頑健性を担保。`timeout`/`max_retries` が外部設定可能で、ループ本体でも `try/except` で生成失敗時のフォールバック決定（stay）を用意。プロンプトはセクション分離（"# Who you are" / "# Your current state" / "# Nearby places" 等）と末尾近接の重要フィールド配置で小型モデルの脱落に配慮されている。

#### メモリ設計
3層のメモリが意味的に機能。①rolling memory（`MEMORY_WINDOW=5` の自由記述行）、②`current_goal`（中期目標、LLMが各ステップで更新可能）、③`relationships`（ウォームアップで構築された warmth・common_topics・first_impression・remembered）。単なるログダンプではなく、過去会話の印象が次の遭遇時にプロンプトに具体的に再注入される（`src/simulation.py:243-251` の `[past acquaintance: warmth=…]`）。ニーズ状態も自然言語表現でメモリ的に作用。

#### コード品質・シミュレーション正確性
同期2フェーズ（収集→適用）で競合状態を回避。シード決定性（persona sampling、phone check、RNG分離）、run 毎の `manifest.json` + `config_snapshot`、JSONL による構造化出力。コードは docstring 充実、命名一貫、`from __future__ import annotations` 使用。減点ポイントとして `src/simulation.py` が約1000行と大きく、プロンプト構築／IO／apply phase が同一ファイルに集中している点。

#### 根拠
- `src/llm_client.py:55-79` — Ollama クライアントのリトライループと timeout
- `src/llm_client.py:101-108` — OpenAI `response_format={"type": "json_object"}` の指定
- `src/simulation.py:507-518` — `parse_decision` の `{`/`}` 抽出による頑健 JSON パース
- `src/simulation.py:798-806` — LLM 失敗時のフォールバック決定（stay）
- `src/simulation.py:243-251` — 関係性メモリのプロンプト再注入（warmth, common_topics, 印象）
- `src/simulation.py:610-682` — manifest.json と出力ディレクトリの管理、run_id の一意化
- `src/simulation.py:822-823` — 同期2フェーズ実行（全エージェントの `decisions` 収集後に apply phase）
- `src/agent.py:48-60` — relationships/needs/memory/current_goal の明示的フィールド

---

## 総評

### 優れている点
- 渋谷駅前という具体的実地に対し、OSM実POI＋citation付き人口分布＋12アーキタイプで世界を厳密に組み上げ、SNS有無のA/B対照群と複数seedで研究的に再現可能な実験設計を実現している
- 7ニーズ×2層の副作用ルール、5層会話開始確率、関係性ウォームアップという3層の世界規則をエージェントの行動には直接介入させず「状態と環境」のみ提示することで、構造的に高い創発ポテンシャルを担保している
- モジュール分離（src/ 7ファイル）と設定外部化（YAML 8種）が徹底され、`detect_emergence.py` の3指標（架空POI／集合的注目／規範発話）など創発検出パイプラインまで一貫している
- LLM呼び出しの頑健性（format_json、tolerant parser、retry、フォールバック）、メモリ多層化（rolling/goal/relationships）、シード決定性、構造化JSONL+manifest出力など技術実装の完成度が高い

### 改善点・提言
- README冒頭の「プロジェクト概要」プレースホルダを埋め、研究問いと対照群A/B設計、創発指標の解釈、想定される結果のパターンを明文化することで、再現性と外部評価のしやすさが大幅に向上する
- `src/simulation.py`（約1000行）はプロンプト構築・IO・apply phaseを別モジュールに分割すると保守性が上がる
- `llm.fleet` モード（cognitive_tier ごとの混在LLM）が `NotImplementedError` のまま残されているため、これを実装するか、READMEで「未実装・将来計画」と明示することで誤解を避けられる
- 創発検出が現状 stdlib-only の正規表現＋類似度ベースで保守的。実験規模に対して埋め込み類似度ベースの集合注目検出など、より定量的な追加指標を準備すると分析の説得力が増す

### 一言コメント
研究的問い・世界設計・モジュラなコード・LLM統合の完成度のいずれも高水準で揃った、ハッカソン提出物としては突出したクオリティ。READMEの研究目的を明文化すれば公開研究プロトタイプとして十分通用する仕上がりである。
