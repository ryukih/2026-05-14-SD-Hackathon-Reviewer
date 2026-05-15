# 評価レポート: all-in-smoke

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

ポーカーテーブルを中心に、テーブル群（table_a / b / c）・バー・倉庫・観戦席・2 つの出口（exit_a / b）を持つ屋内空間で進行する、ポーカーと火災危機が同時並走するマルチエージェント・シミュレーション。ハンドが進む最中に赤い危険リング（fire perimeter）がテーブル中央に向かって縮小していき、各エージェントは「プレーを続ける」「勝ち手札に固執する」「チップが惜しくて居残る」「危険が到達する前に席を立つ」のいずれを選ぶかを毎ターン LLM 判断する。火がかかる位置に座り続けたエージェントは `engulfed` 状態を経て `fatal` 状態へ段階的に悪化する設計。LLM は OpenRouter 経由の grok 系モデル等で駆動され、ポーカーフェーズと火災フェーズ双方の状況をフィードバックステートとして受け取りつつ、JSON 構造化出力で意思決定を行う。テーマは「不完全情報のゲーム的圧力（all-in 心理）」と「物理的危機（fire）」が同時に作用する状況で、エージェントの判断が合理的撤退・損切り・希望的観測のいずれに偏るかを観察することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  5   |  5   |  -   |    5.0    |
| B. 世界設定  |  8   |  8   |  -   |    8.0    |
| C. 発展性    |  7   |  7   |  -   |    7.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  28  |  28  |  -   |  **28.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 5.0/10

#### 評価
世界ルールは非常に精密に設計されている。ポーカーエンジン側は完備した不完全情報ゲームとして実装され、危機フェーズも `FIRE_CONTACT_DANGER=0.92`、`FATAL_EXPOSURE_TICKS=3`、シュリンクする `safe_radius`、`crisis_match_rule`（fatal_overrides_chips / stand_up_forfeits_stack / last_seated_claims_forfeited_chips）といった一貫したルール群で記述されている。一方、行動の自由度は二極化している。ポーカー判断は LLM に raw observation（hole_cards, board, legal_actions, stacks, recent_table_talk, session_context）だけを渡し、戦略は完全に委ねており高い自由度を持つ（OpenRouter/Ollama 構成時）。しかし作品のハイライトである「火の手にどう反応するか」は LLM が一切呼び出されず、`live_fire_simulation.py` の if/elif 分岐（`stood_up`/`clinging_to_stack`/`tempted_by_chips`/`engulfed`/`fatal`）と閾値計算で決定される。`inner_voice` も `_crisis_inner_voice` のテンプレ条件分岐で生成される。さらにデフォルトの `all_in_smoke_demo.yaml` は全員 scripted エージェントで LLM すら呼ばれない構成のため、エージェント自律性は実質ポーカーの数値プレッシャーを通じた間接的な反映に留まる。世界ルール精度は高（8 相当）だが、危機局面の行動自由度は低（3 相当）であり、ハーモニックバランスとして中央値帯となる。

#### 根拠
- `live_fire_simulation.py:1264-1331` — 火災フェーズの状態遷移（contact, leave_score, wants_last_hand, temptation_window 等）は完全に閾値分岐で決定され、LLM は呼ばれない。
- `live_fire_simulation.py:891-947` — `_crisis_inner_voice` は status/motive をキーにした条件分岐で固定文字列を選択する純粋なテンプレート生成。
- `live_fire_simulation.py:35-36` — `FIRE_CONTACT_DANGER = 0.92`、`FATAL_EXPOSURE_TICKS = 3` と物理ルールが厳密に定義されている。
- `poker_agents/llm_agent.py:33-101` — LLM プロンプトは raw observation（hole_cards, legal_actions, recent_table_talk, session_context）のみを渡し、行動指示を含まない（自由度高）。
- `configs/all_in_smoke_demo.yaml:3-77` — デフォルトデモは全員 scripted agent（TightAgent/AggressiveAgent/CallingAgent/RandomAgent）で LLM 呼び出しゼロ。
- `README.md` — "If fire reaches a still-seated agent and contact continues for several ticks, the state escalates from `engulfed` to `fatal`." — エスカレーションは時間とコンタクトの硬いルール。

---

### B. 世界設定 — 最終: 8.0/10

#### 評価
ポーカーの不完全情報ゲームとシュリンクする危険リング（火災）を同時進行で重ねる設定は独創性が高く、標準的なデモでは見ない構図である。「stand-up forfeits stack（席を立てばチップを放棄）」「last seated claims forfeited chips（最後まで座っていた者が放棄チップを得る）」「fatal overrides chips（死亡したら勝負終了）」という危機ルールは、サンクコスト、損失回避、執着、合理性が緊張するゲーム理論的セッティングとして機能している。チップ執着・損失追跡・rivalry 圧などポーカーで蓄積した心理圧が、避難行動の遅延要因として転移する設計は行動経済学・社会心理学（避難意思決定、群衆心理）の研究的射程を持つ。さらに TimeQL の lack/body プロファイル統合により、エージェント個別の「能力ギャップ」（fold_ability, situational_awareness, public_responsibility 等）を心理学的に基礎づけているのも踏み込みがある。完全に独創的というよりは、避難シミュレーション + ゲーム理論 + 心理プロファイルの組み合わせとして強い設定。

#### 根拠
- `README.md` — "Poker-phase incomplete information and disaster pressure now run at the same time. While the hand is still being played, a red danger ring shrinks toward the center of the table."
- `live_fire_simulation.py:692-797` — `_crisis_match_snapshot` で fatal_overrides_chips / stand_up_forfeits_stack / last_seated_claims_forfeited_chips の三つ巴ルールがエンコードされ、ヘッズアップ局面で「先に立つか粘るか」の意思決定が直接報酬構造に結びついている。
- `smoke_simulation.py:30-49` — 部屋レイアウト（table_a/b/c, bar, storage, spectators, exit_a/b）と座席配置で空間的危機モデルが提示される。
- `smoke_timeql_converter.py:92-197` — TimeQL の body_vector/lack_scores から `fold_ability` / `public_responsibility` / `situational_awareness` などの危機行動性能ギャップを派生させる詳細なマッピング。
- `live_fire_simulation.py:104-148` — `PokerFeedbackState` が chip_attachment, loss_chasing, entitlement, table_image_pressure, rivalry_pressure, fold_success_memory をポーカーから累積し、避難行動の遅延要因に転移する。

---

### C. 発展性 — 最終: 7.0/10

#### コード拡張性
モジュール分離は良好で、`poker_engine`（cards/deck/betting/showdown/hand_evaluator/pots/table）、`poker_agents`（base/scripted/llm/openrouter/endpoint/personas/session_state/voice_profile/card_language）、`smoke_simulation`、`live_fire_simulation`、`tools/` が独立している。エージェントは YAML manifest 経由でプラガブルで、scripted/endpoint/llm/openrouter の 4 種類をサポートしている。新しい persona やシナリオは設定追加で対応可能。`SCRIPTED_CLASSES` レジストリ、`crisis_profile.ability_gaps` の上書きメカニズム、TimeQL プロファイルのオプショナル統合など拡張ポイントが明確。テストカバレッジも 12 ファイル以上と厚い。一方で `live_fire_simulation.py` が 1398 行と巨大化しており、状態遷移ロジックの分割の余地がある。

#### 将来展望
README は TimeQL Layer 3 統合、OpenRouter デモ、MP4 エクスポートパイプライン、ブラウザビューワー、`smoke_timeql_converter` による身体・lack プロファイルから能力ギャップへのマッピングなど、具体的かつ技術的に信頼できる拡張点を提示している。`configs/` には複数の構成（demo, openrouter_grok_full/smoke, poker_mindsport, poker_llm_demo）が用意され、ユースケース別の運用方針が見える。ただし「この実験から何を学ぶか」「どの研究的・ビジネス的問いに答えるか」というハイレベルな展望の記述は薄く、技術 README に留まっている点が惜しい。

#### 根拠
- `poker_agents/manifest_loader.py:38-43` — `SCRIPTED_CLASSES` レジストリで新規エージェントクラスの登録が容易。
- `poker_agents/manifest_loader.py:66-137` — `AgentSpec.build` が scripted/endpoint/llm/openrouter を統一インターフェースで構築。
- `configs/` — 9 個の YAML 構成ファイル（demo, openrouter_grok_full/smoke, ollama_smoke, poker_demo, poker_llm_demo, poker_mindsport_6p/smoke/full）が並ぶ。
- `README.md` — "Layer 3" 節で TimeQL 連携の組み込み手順と persona profile のオプショナル化を具体的に示している。
- `tests/` — 12 ファイルのテストが session_state/poker_engine/poker_simulation/smoke_simulation/timeql_converter/replay_timing/llm_agent/openrouter_agent/byo_agent/card_language/commentator/tts_normalizer まで網羅。
- `live_fire_simulation.py` — 単一ファイルで 1398 行、危機判定ロジックの分割余地あり。
- README は研究的問い（"Does poker-table trust transfer into fire-alarm trust?" は `smoke_simulation.py:478` のログラベルにのみ登場）を本文の前面に出しておらず、ハイレベル展望の記述は薄い。

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
複数バックエンド（Ollama ローカル + OpenRouter）を抽象化し、構造化 JSON 出力（Ollama: `format: "json"`, OpenRouter: `response_format: {type: "json_object"}`）で出力スキーマを制約している。Schema rules はカードの日本語表記、`inner_voice` 必須性、`reasoning` と `inner_voice` の役割分離など 10 項目以上の制約を含み、プロンプトエンジニアリングが綿密。エラー処理は transport エラー / JSON パース失敗 / 空コンテンツ全てで fold にダウングレードする防御的設計。temperature・think・table_talk_allowed・persona など多くがエージェント単位で設定可能。`card_language.py` は LLM 出力の表記揺れを 20 種類以上の正規表現で正規化する力作。

#### メモリ設計
`SessionState` は tilt 減衰、rivalry note 蓄積、recent_outcomes ledger を扱い、ばっどビート検出（own hand >= two_pair で showdown 負け→ `_TILT_BADBEAT_BONUS` 加算）まで実装されている。LLM への露出は数値 tilt ではなく日本語 mood label（"落ち着いている" / "やや熱が入っている" / "明らかにイラついている" / "完全にティルト気味"）に限定する設計で、機械的自己修正を避ける配慮がある。危機フェーズの `PokerFeedbackState` も 9 次元の心理プレッシャーを累積・減衰する。voice_profile による TimeQL `inner_voice_directives` の注入も洗練されている。

#### コード品質・シミュレーション正確性
全モジュールで dataclass + 型ヒント、各ファイルに目的を明示した docstring。`JsonlLogger` の単調増加 step カウンタは action/memory_reasoning/table_talk を同期させる設計で再現性が高い。乱数はすべて seed で制御され、トーナメントもハンドごとに `seed + hand_index` で deterministic。`poker_engine` 側の `apply_action` / `legal_actions` / `start_hand` も契約が明確で、resolve_action の安全 fallback も丁寧。テストカバレッジが厚い。減点要素は live_fire_simulation の巨大化と、危機 inner_voice 生成が完全にテンプレ化されている点（D のスコープでは些細）。

#### 根拠
- `poker_agents/llm_agent.py:169-217` — `format: "json"` 強制、`urllib.error.URLError` / `TimeoutError` / `JSONDecodeError` / 非 dict ペイロードの全てを fold にフォールバック。
- `poker_agents/openrouter_agent.py:88-148` — `response_format: {type: "json_object"}`、`max_completion_tokens` 制御、API キー欠如時の早期 fold。
- `poker_agents/personas.py:15-54` — `SCHEMA_RULES` で 10 項目の制約（action enum, amount range, inner_voice 必須、card 表記、文字数上限など）を明示。
- `poker_agents/session_state.py:113-229` — `ingest_hand_result` で tilt 減衰・大負け加算・badbeat 検出・rivalry 自動記録を実装。
- `poker_agents/session_state.py:270-297` — `prompt_block` が numeric tilt を隠し mood label のみ露出。
- `live_fire_simulation.py:202-279` — `_feedback_snapshot_by_step` が action/memory_reasoning/hand_result/session_snapshot 各イベントから 9 次元の心理プレッシャーを累積。
- `poker_simulation.py:172-188` — `JsonlLogger.next_step` で action/memory_reasoning/table_talk を同一 step に結びつけ、再生時の整合性を確保。
- `poker_agents/card_language.py:25-103` — カード表記正規化が 20 種以上の正規表現で実装される徹底ぶり。
- `tests/` — 12 ファイル、合計 3000 行超のテストで主要モジュールをカバー。

---

## 総評

### 優れている点
- ポーカー（精密な不完全情報ゲーム）と火災危機（縮小する危険リング + 心理プレッシャー転移）という二層構造の世界設定が独創的で、行動経済学・避難心理学の研究的射程を備えている。
- LLM 統合は複数バックエンド対応、構造化 JSON 出力、persona/voice_profile/TimeQL の三層プロンプト合成、防御的エラー処理まで揃った成熟度の高い実装である。
- メモリ設計が秀逸：tilt は decay + badbeat 加算で時間進化し、LLM には raw 値ではなく mood label のみ露出する。rivalry note は showdown/payouts から自動生成され、次ハンドのプロンプトに再注入される。
- コード品質が一貫して高い（dataclass、型ヒント、明確な docstring、12k LOC + 厚いテスト）。
- TimeQL プロファイル → 危機能力ギャップへの変換 (`smoke_timeql_converter.py`) は単なるフレーバーではなく、能力ベクトルとして危機行動に作用する設計。

### 改善点・提言
- 危機フェーズ（火災下の意思決定）が完全に if/elif の閾値ロジックで決定され、LLM が呼ばれない。これは本作の最大の演出ポイントである「立つか・残るか」の判断を、ハードコードされたルールに委ねていることを意味する。ここで LLM に raw 圧力データ（danger, belief_fire, chip_temptation, crisis_match_forfeit_pressure 等）を渡して自由判断させれば、創発性が飛躍的に高まるはず。
- デフォルト `all_in_smoke_demo.yaml` が scripted agent のみで構成されており、LLM デモは別 yaml に分離している。新規読者が動かす最初の体験で LLM 駆動が見えにくい。
- `live_fire_simulation.py` が 1398 行の単一ファイルで、状態遷移・プレッシャー計算・voice 生成・stack 解決が混在している。`crisis_state_machine.py` / `pressure_model.py` / `crisis_voice.py` への分割が将来の保守性を高めるだろう。
- README が技術手順書中心で、「この実験から何を観察したいのか」「どの仮説を検証するのか」というハイレベル展望が薄い。トラスト転移、サンクコスト、避難遅延などの研究的問いを前面に出すと魅力が伝わりやすい。

### 一言コメント
ポーカーと火災危機を同時進行させる独創的な世界観と、LLM 統合・メモリ・テスト・モジュール構成すべてで高い完成度を示す力作。一方、ハイライトであるはずの危機局面の意思決定が LLM ではなくハンドコードされた閾値ロジックで処理されている点が、「創発設計」の評価を中央値に押し下げている。
