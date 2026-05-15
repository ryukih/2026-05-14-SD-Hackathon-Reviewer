# 評価レポート: lunar_agents

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

月南極の永久影領域（PSR）における水氷探索を抽象化した 2D シミュレーション基盤で、複数の LLM ローバーエージェントが永久影クレーター、斜面、照度分布、フレア等を含む環境上で行動する。各エージェントはグローバル情報や中央コーディネータを持たず、自身の観測、近傍通信、チームの確認済み発見、バッテリー残量や地形などの定量情報のみを与えられ、move / survey / mark_resource / deposit_at_base などの行動を LLM が選択する。主要メカニズムとして、persona と個人ミッションによる個体差、エージェントごとに別個に蓄積される観測履歴・自己メモリ・コミットメント状態、message_type / location / value / confidence / requested_action を持つ構造化通信、バッテリー経済とサンプル品質の指数減衰による環境コストが組み込まれている。シナリオは Phase 1 の `water_exploration`、コミットメントと分業の観察を狙う `sample_return_micro`、南極風地形・基地充電・フレア・サンプル預入を含む Phase 1.5 主要シナリオ `south_pole_survival_survey` の三系統が用意されている。研究テーマは「個体差 → 非対称性 → コミュニケーション → 協調・競争 → 創発」という連鎖の検証であり、明示的な協調指示なしに集団レベルの分業や情報伝播が立ち上がるかどうかを問う。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  9   |  8   |  -   |    8.5    |
| B. 世界設定  | 10   | 10   |  -   |   10.0    |
| C. 発展性    | 10   |  9   |  -   |    9.5    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    | 38   | 36   |  -   |  **37.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 8.5/10

#### 評価
世界ルールの設計精度は非常に高い。月南極の層別地形モデル（crater rim/wall/floor、slope/passability/illumination/water_potential を独立に保持）、バッテリ経済（行動別コスト、太陽光ゲイン、斜面ペナルティ、フレア・ドレイン、ベース充電）、サンプル品質減衰、構造化メッセージ・スキーマ、軌道データの非対称配布（visibility_fraction と per-agent noise）が、定量的な制約と圧力を生み出している。行動の自由度については、プロンプトは座標・水含有率・距離・バッテリー残量などの数値を渡し、「危険／豊富／安全」のような評価語を避ける方針が明示されている。一方で persona の description（例: "prioritize battery margin and proximity to base"）や、Navigation Guidance / Personal Lens / loop detection 等のヒント・セクションは行動を緩やかに方向付けており、完全な「データのみ」とは言えない。それでも、行動決定（move/survey/mark_resource/deposit_at_base/...）、メッセージ内容、target/commitment はすべて LLM 側に委ねられ、構造的に創発の余地を残す優れた設計である。

#### 根拠
- `environments/lunar_2d/map.py:143-217` — クレーター層別モデル（rim/wall/floor）、斜面・通行性・照度の精密な定義
- `environments/lunar_2d/agent.py:49-58` — 行動別バッテリーコスト辞書（move/survey/mark_resource/stay/message/step）
- `environments/lunar_2d/agent.py:41-44` — サンプル品質の指数減衰（0.85/step、floor=0.10）で「return now」の圧力を環境側から生成
- `environments/lunar_2d/orbital_data.py:99-146` — エージェント毎に異なる軌道データ・ビュー（visibility_fraction と per-agent noise）
- `environments/lunar_2d/agent.py:461-523` — メッセージ・プロンプトは座標・水含有・バッテリー等の数値を渡し、type 選択のみ提供
- `README.md` — "LLM プロンプトには、座標、距離、バッテリー、含水率、確度などの定量情報を渡します。"
- `README.md` — "「危険」「豊富」「安全」などの評価語はできるだけプロンプトに含めず、判断は LLM 側に委ねます。"
- `environments/lunar_2d/personas.py:36-44` — persona description は短いが行動傾向を示唆（"prioritize battery margin..."）→ わずかなスクリプティング
- `environments/lunar_2d/agent.py:1360-1372` — "WARNING: at east boundary — right is blocked, move left/up/down" のような行動方向ヒント

---

### B. 世界設定 — 最終: 10.0/10

#### 評価
月南極の永久影領域（PSR）における水氷探索という、現実の LUPEX・Artemis・ILRS・Chang'e 7 等のミッションから逆算した極めて独自性の高いシナリオ。Haworth 様コールドトラップ、Nobile rim、scarp、ベース、リレーステーション、軌道センサーといった実在物理を組み込み、バッテリー経済・サンプル品質減衰・太陽フレア回避・シェルター選択を統合している。複数シナリオ（water_exploration / sample_return_micro / south_pole_survival_survey）でフェーズ別の検証が可能。商業・宇宙開発・社会科学（情報の非対称性、ICT インフラ共有、制度差）への発展余地が明確で、極めて高い分析価値を持つ。

#### 根拠
- `README.md` — "Lunar Agents は、LLM エージェント群が 2D 月面環境で水氷資源を探索し、観測・記憶・通信を通じて協調や競争を形成するかを調べるためのシミュレーション基盤です。"
- `CLAUDE.md` — "シミュレーションは国際月探査ロードマップ（LUPEX・Artemis・Chang'e 7等）から逆算した5類型のミッションに対応する。"
- `scenarios/south_pole_survival_survey/scenario.yaml:1-139` — 月南極相当の地形（Haworth, Nobile, deep_psr_west/east, scarp）、ベース、illumination ridge を細部まで定義
- `environments/lunar_2d/orbital_data.py:17-25` — 軌道データの仕様（terrain 既知、水含有確率は ±0.15 ノイズ、in-situ 計測のみが確度高）
- `environments/lunar_2d/relay_stations.py` — リレーステーションによる充電・通信拡張、共同建設アクション

---

### C. 発展性 — 最終: 9.5/10

#### コード拡張性
依存方向が一方向に厳格に保たれており（tools → scenarios → environments → core）、`core/` は環境非依存。新規シナリオは `scenarios/<name>/scenario.yaml` を追加するだけで `runner.py` が読み込む。LLM クライアントは `LLMClient` 基底（`core/llm/base.py`）から Ollama・Anthropic 両対応。メモリ機構は `MemoryConfig` で size/timeline/stream をフラグ切り替え可能、`PsychState` も opt-in。30+ の YAML config と 10+ の plan で実験を網羅し、新地形（illumination/rough/blocked regions）も config だけで追加可能。テスト群（約3,700 行）が豊富で回帰検証可能。

#### 将来展望
README と CLAUDE.md に Phase 1.6（LUPEX/VIPER 型）、Phase 1.7（Artemis 初期運用、複数 PSR・インフラ共有）、Phase 2.0（Artemis vs ILRS 制度差）、Phase 2.5（ISRU）、3D Lunarcraft 環境への移行が具体的に提示されており、研究的・産業的価値の深化が明確。「アトラクタ問題」「reasoning 均質化問題」「14b モデルへの移行」など、既知の課題と次の改善箇所が memo/ と docs/ に詳細に記録されている。

#### 根拠
- `README.md` — "tools -> scenarios -> environments -> core" 一方向依存
- `core/llm/base.py:9-38` — 抽象基底＋async ラッパー＋セマフォ制御
- `core/llm/anthropic_client.py`, `core/llm/ollama.py` — 2 つの provider 実装
- `environments/lunar_2d/memory.py:33-68` — MemoryConfig による機能フラグ
- `scenarios/runner.py:546-552` — emergence.memory / emergence.psych_state を YAML から動的構築
- `experiments/configs/` — 30+ の実験 config が並ぶ
- `README.md` — "その後は、LUPEX 型の資源品質判定、通信・電力インフラ共有、Artemis/ILRS 的な制度差、複数 LLM の協調・競争、3D Lunarcraft 環境へ拡張する計画です。"
- `CLAUDE.md` — Phase 1.5→1.6→1.7→2.0→2.5 のロードマップ
- `tests/` — 約 3,700 行の pytest スイート（core/environments/tools 全カバー）

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
- 構造化 JSON 出力、Required フィールド検証、エイリアス対応（`core/llm/parsing.py`）。
- thinking-tag 除去（複数記法対応、未閉じ tag への defensive strip）。
- `parse_status` を `ok/partial/fallback/empty_response` の 4 値で観測可能化し、`parse_fallback_rate > 0.05` のラン除外を明示（`tools/emergence_metrics.py:224-247`）。
- Ollama のモデルサイズ別 `max_concurrency` ヒューリスティック（`ollama.py:14-27`）でキュー詰まりを防止。
- Anthropic API キー検証は値を読まずに boolean のみ返す等のセキュリティ配慮（`anthropic_client.py:16-22`, `CLAUDE.md` セキュリティポリシー）。

#### メモリ設計
- 3 層のメモリ機構: 旧来の self.memory リスト、`step_history`/`timeline` による時系列再現、Smallville 風 MemoryStream（append-only + recency × importance × relevance retrieval + 周期的 reflection）。
- `discovery_log`, `blocked_tiles`, `surveyed_positions`, `marked_resources`, `team_discoveries` などのドメイン固有構造化メモリを併用。
- reflection prompt が定量質問形式（Q1=最頻アクション数、Q2=有益数、Q3=continue/change）で、ループ脱出を促す設計。
- PsychState（mutable な needs/emotion ベクトル）で AgentSociety 論文の知見を取り込み、静的 persona ラベルの限界を補う設計。

#### コード品質・シミュレーション正確性
- 2-phase（メッセージ決定→配信→行動決定→行動実行）シミュレーションでレースコンディションを排除（`core/simulation.py:69-110`、`_run_step_async_two_phase`）。
- seed 指定可能で再現性を確保（resources.seed、OrbitalData の seed_base）。
- 各モジュールに docstring と "Why" 説明が豊富で意思決定の経緯が記録される。
- 一方で `environments/lunar_2d/agent.py` は 1,997 行と巨大で、責務分離（prompt 構築・解析・apply_action・メモリ更新・psych 更新）の細分化余地がある。

#### 根拠
- `core/simulation.py:69-110` — Phase 1=message decide / Phase 2=deliver / Phase 3=action decide / Phase 4=apply の 4 段階
- `core/llm/parsing.py:30-36` — parse_status の 4 値仕様
- `core/llm/parsing.py:57-71` — thinking-tag 除去（unclosed 対応）
- `core/llm/ollama.py:14-27` — モデル別 max_concurrency
- `environments/lunar_2d/memory.py:105-156` — MemoryStream の recency × importance × relevance スコアリング
- `environments/lunar_2d/memory.py:165-212` — 構造化 reflection prompt（Q1/Q2/Q3/Q4）
- `environments/lunar_2d/psych_state.py:1-100` — PsychState 設計と AgentSociety 引用
- `scenarios/runner.py:615-792` — 包括的なログ出力（state/metrics/events/messages/memory_reasoning）
- `tools/emergence_metrics.py:64-217` — individuality_index / reasoning_diversity / spatial_entropy / info_propagation_rate / role_specialization の定義
- `environments/lunar_2d/agent.py` — 1,997 行（コード量過大の指摘点）

---

## 総評

### 優れている点
- 月南極 PSR 水氷探索という現実ミッション（LUPEX / Artemis / ILRS）から逆算した、極めて独自性が高く分析価値の大きい世界設定。
- 環境側のコスト・物理（バッテリ経済、サンプル品質減衰、フレア、シェルター、軌道データ非対称）で行動圧力を生み、プロンプト側の指示を最小化する一貫したデザインポリシー。
- LLM 出力解析の徹底した observability（4 値 parse_status、parse_fallback_rate、thinking-tag 防御）と、Cycle 10 失敗からの根本対応の物語性。
- 3 層メモリ（legacy/timeline/Smallville stream + reflection）と PsychState（AgentSociety 由来）の opt-in 構成。
- 厳格な一方向依存アーキテクチャ、30+ config、~3,700 行のテスト、複数 LLM provider 対応、再現性のための seed 管理。
- Phase 1→1.5→1.6→1.7→2.0→2.5 と続く具体的・段階的なロードマップ。

### 改善点・提言
- `environments/lunar_2d/agent.py` が 1,997 行と肥大化している。prompt 構築層・パーサー層・apply_action 層・メモリ更新層・PsychState 連携層への責務分離リファクタリングが望ましい。
- persona の description（"prioritize battery margin and proximity to base" 等）や Navigation Guidance / Personal Lens セクションは、定量データのみを渡す方針に対しわずかに行動を方向付けている。完全な「データのみ」を目指すなら、persona は数値パラメータのみ渡し description を除去する選択肢もある（現状の `behavioral_params` は数値だが persona description はテキスト）。
- README の主要成果（individuality_index 0.044→0.513、情報伝播率 0.04→0.34、サンプル品質 0.923 wt%）は研究的にインパクトがあるが、再現に必要な run ID・seed・config を README に直接列挙すると外部評価がしやすい。
- アトラクタ問題（「コストが低くやった感のある行動」への引き込み）への対策（環境コスト調整、構造化 reflection、PsychState）は実装済みだが、定量的に効果が示された比較データが README に欲しい。

### 一言コメント
世界設計・実装品質・観測可能性・将来展望のいずれにおいても、ハッカソン水準を大きく超える完成度を持つ研究用プラットフォームである。特に「定量データのみを渡し、創発を環境圧力で誘発する」という設計思想と、それを裏付ける詳細な失敗ログ・改善履歴（memo/docs/）が秀逸。
