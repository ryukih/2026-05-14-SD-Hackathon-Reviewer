# 評価レポート: 2d-multi-places-simulation-on-fire

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

LLM エージェントの創発性と自律性を発現させることを目的とした、2 次元空間マルチエージェント・シミュレーション。フィールドに左バー・右バーの 2 つの場所（capacity 12 / 10）を配置し、15 体のエージェントが共存する。シミュレーション途中で複数の火災イベント（fire_1: step 35 / intensity 0.8 / radius 15、fire_2: step 45 / intensity 0.5 / radius 8）が発火し、エージェントは自律的に危険を察知・伝達・避難を行う。「ハイブリッド知性」設計を採用し、安全判断（曖昧ゾーン外）はルールベース、曖昧ゾーンの社会的判断は LLM 呼び出しで処理する二段ゲート構造になっている。エージェント間メッセージは日本語で生成され、`known_fires` の信頼度階層・減衰・幻覚フィルタ・ユニーク報告者追跡、`canonical_fire_id` による位置ベース正規化など、火災情報の堅牢な伝播メカニズムが実装される。62 件のテストによる TDD で品質保証され、Ollama ローカル実行で API コスト 0 円。テーマは、定量情報のみを与えられたエージェント群から、指揮者・情報中継者・生存確認者などの役割分化が自然発生するかを観察することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  4   |  4   |  -   |    4.0    |
| B. 世界設定  |  6   |  6   |  -   |    6.0    |
| C. 発展性    |  7   |  7   |  -   |    7.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  25  |  25  |  -   |  **25.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 4.0/10

#### 評価
世界ルール（半径、定員、通信半径、火災の物理）は精度高く定義されているが、エージェント行動の自由度が極端に低い。`decide_action_by_rule`（agent.py:1585-1857）が大半の意思決定をルールで決定し、LLMは「曖昧ゾーン」かつ「火災言及メッセージあり」という二重ゲートを通った場合のみ呼び出される。さらにLLMプロンプト自体が"Computed optimal escape: X. Move in this direction"（agent.py:1031-1033）や"PRIORITY: Move to maximize distance from fire"（agent.py:1286）など、計算済みの最適方向を明示的に指示しており、LLMは事実上テキスト整形器として機能している。メッセージプロンプトも「1. 火災位置: Fire at (X, Y)... 4. 安全な方向の推奨」（agent.py:1126-1133）と内容構造を細かく規定する。世界ルール軸≈8、行動自由度軸≈3で調和平均は5未満。

#### 根拠
- `agent.py:1585-1857` — `decide_action_by_rule()`：In-place時の組織的ドリフト・火災時の合成反発ベクトル・避難バー選択をすべてルールで決定
- `agent.py:1020-1034` — `_build_evacuation_decision_guidance()`：「Computed optimal escape: <DIRECTION>. Move in this direction to maximize distance from fire.」とLLMに具体的指示
- `agent.py:1048-1056` — `_build_fundamental_purpose()`：「Priorities: 1. Survive 2. Warn others 3. Coordinate」と価値順序を明示
- `agent.py:1124-1133` — メッセージプロンプトに「1. 火災位置 2. 強度と半径 3. あなたの避難方向 4. 他のエージェントへの推奨」を箇条書き指示
- `agent.py:1281-1284` — In-place時に「STAY HERE unless BAR SAFETY STATUS shows this bar as DANGER」と直接指示
- `README.md` — "安全判断はルール、曖昧な社会的判断はLLMが担当"（ハイブリッド知性設計が、設計者自身の意思としてLLMの自由度を縮小）

---

### B. 世界設定 — 最終: 6.0/10

#### 評価
2次元グリッド上で15エージェント、2つのバー（定員12/10）、2つの火災（step35/45発生）が干渉する避難シナリオ。定員制約とダブル火災のタイミングがルートプランニング上の緊張を生み出し、情報伝播・避難先選択・キャパシティ調整の問題が交差する点は分析的価値がある。ただし火災避難シミュレーション自体はマルチエージェント研究の典型題材で独自性は限定的。実世界の災害避難・群衆行動の縮図として一定の意義は持つ。

#### 根拠
- `config.yaml:21-34` — 2つのバー（容量12/10、左右配置）が定員制約を生む
- `config.yaml:112-124` — 2火災が時間差発生（step 35と45）し、避難先の動的再選択を要求
- `README.md` — "フィールドに複数の場所（左右のバー）が存在し、シミュレーション途中で複数の火災イベントが発生する"
- `simulation.py:511-571` — `get_fire_info_for_agent()` 半径内のみ知覚＋バーセンター警報、知覚モデルが情報非対称を生成
- `README.md` — "創発的役割分化：指揮者・情報中継者・生存確認者など役割が自然発生" の主張（ただし設計上の役割付与は確認されず）

---

### C. 発展性 — 最終: 7.0/10

#### コード拡張性
モジュール分離は良好。`agent.py`/`simulation.py`/`ollama_client.py`/`llm_router_sync.py`/`utils.py`/`visualization.py` が責務ごとに分離され、`PlaceConfig`/`FireConfig` TypedDict（utils.py:7-24）で型安全な拡張が可能。`config.yaml`で場所・火災・LLMモデル・最適化フラグを外部化しており、新しい場所タイプ（cafe, library等）の追加もコード変更なしに可能。62テスト（README記載）による回帰保護も拡張容易性に寄与。ただし`agent.py`が2083行と巨大化しており、長期的には責務分割が望ましい。

#### 将来展望
READMEには「パフォーマンス実績」「重要な最適化」「アーキテクチャ」セクションは詳細だが、明示的な「将来展望」「Future Work」セクションは欠落。研究的・事業的な発展方向（例：実建物の避難シミュレーション、群衆動力学への接続、より多様な災害タイプ）の議論がない。

#### 根拠
- `agent.py`、`simulation.py`、`ollama_client.py`、`llm_router_sync.py`、`utils.py` — モジュール分離
- `config.yaml:21-33` — 場所のYAML設定で新規場所追加が容易
- `config.yaml:110-124` — 火災のYAML設定で複数火災・任意座標を指定可能
- `utils.py:17-24` — `PlaceConfig` TypedDictで型付き場所定義
- `tests/`（10テストファイル） — TDDの構造的拡張安全性
- `README.md` — 将来展望に関する独立セクションなし

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
高度に最適化されたLLM統合。OllamaClient（ollama_client.py）はwarmupによるコールドスタート回避、Qwen向け`/no_think`プレフィックス、JSON出力フォーマット、適切なタイムアウト設定を実装。fast（llama3.2:3b）/smart（qwen3.5:4b）二段モデル切替（agent.py:170-206 `_get_llm_client`）で火災文脈の有無により推論コストを最適化。`_strip_think_tokens`（agent.py:1329-1340）と`_extract_json_from_text`（agent.py:1342-1368）で思考トークン除去と頑健なJSON抽出。冷却（cooldown）、飽和スキップ、近傍受信者ゼロ時のスキップなど、LLM呼び出し総量を抑える複数の最適化が共存。

#### メモリ設計
`known_fires`に`source`（direct/sys_warn/agent_report）と`confirmed_count`、`last_seen_step`を持たせた信頼度階層付きメモリ（agent.py:96-102, 304-350）が秀逸。境界外座標やradius欠落の幻覚フィルタ（agent.py:276-291）、`STALE_STEPS=8`での非直接知覚の減衰（agent.py:354-377）、報告者ユニーク追跡で中継チェーンの過大計上を防ぐ仕組み（agent.py:319-330）など、マルチホップ情報の信頼度管理が体系的。`evacuation_history`でPTSD的回避バッファ倍率を変える（agent.py:903-904）など、長期記憶の意思決定への結合も明確。

#### コード品質・シミュレーション正確性
コメント・docstringが豊富で意図が読み取りやすい。一方`agent.py`が2083行と単一ファイルとして肥大化しており、責務分割の余地あり。シミュレーション正確性は4フェーズ同期パイプライン（メッセージ決定→送信→行動決定→移動、simulation.py:665-696）で順序が明確。`ThreadPoolExecutor`を使うがOllama直列性を踏まえ`parallel_workers=1`を推奨設定（config.yaml:101, simulation.py:114）。バー占有スナップショットを同フェーズで一括取得（simulation.py:858-860）してレース条件を回避。`_canonical_fire_id`（simulation.py:195-231）で位置ベースの正規IDを与え、直接知覚者とLLM伝聞者の名前差異を統一する設計は実装品質が高い。

#### 根拠
- `ollama_client.py:52-78` — `warmup()` でコールドスタート回避
- `ollama_client.py:102-104` — Qwen向け`/no_think`プレフィックス
- `agent.py:170-206` — `_get_llm_client()` fast/smart二段ルーティング
- `agent.py:1329-1368` — think token除去とブレース深さ追跡によるJSON抽出
- `agent.py:96-102, 304-350` — `known_fires`の信頼度階層メモリ
- `agent.py:276-291` — 境界外座標・radius欠落の幻覚フィルタ
- `agent.py:354-377` — 非直接火災の8ステップ減衰
- `simulation.py:195-231` — `_canonical_fire_id` 位置ベース正規ID
- `simulation.py:665-696` — 4フェーズ同期パイプライン
- `tests/`（10ファイル、README記載62テスト） — TDDによる正確性保証

---

## 総評

### 優れている点
- 信頼度階層付きメモリ設計（direct/sys_warn/agent_report + confirmed_count + 減衰）は実装品質が高く、マルチホップ情報伝播の幻覚耐性を真剣に追求している
- LLM呼び出し最適化（warmup、二段モデル、cooldown、飽和スキップ、文脈スキップ）の体系性とコスト感覚
- 位置ベースのCanonical Fire IDで直接知覚者とLLM伝聞者の名前差を吸収する設計は秀逸
- 62テストでTDDが実施されており、回帰保護が手厚い
- YAML設定による場所・火災・最適化フラグの外部化

### 改善点・提言
- 「ハイブリッド知性」設計は技術的には合理的だが、ハッカソンの「創発性・自律性の発現」という目的に対しては逆効果。ルールエンジンが避難判断の大半を決定し、LLMプロンプトも"Computed optimal escape"等の最適解を予告するため、LLMは実質的にテキスト整形器となっている。LLMに数値データのみを提示し、判断・指示語（"STAY HERE", "Move in this direction"）を排除すれば、設計上の創発ポテンシャルは飛躍的に高まる
- READMEに「将来展望」「Future Work」セクションを追加し、研究的・事業的な発展方向を明示すると評価が上がる
- `agent.py`が2083行と肥大化しているため、火災知識管理・ナビゲーション・プロンプト構築・LLM応答パースを別モジュールに分割するとさらに保守性が向上

### 一言コメント
技術実装は本提出群の中でも上位クラスで、メモリ設計・LLMコスト最適化・正確性への配慮が際立つ。一方でハッカソン主旨である「創発性・自律性の発現」を強調しながら、設計の中枢はルールエンジンに依存しておりLLMの裁量を意図的に絞っている点が、創発設計スコアの抑制要因となった。
