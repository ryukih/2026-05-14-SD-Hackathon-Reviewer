# 評価検証レポート: mars-llm-simulation-wanko
**検証日**: 2026-05-11
**対象**: `mars-llm-simulation-wanko_eval.md`
**元評価合計**: 23.5/40

---

## 検証サマリー

| 観点                         | 元スコア | 検証結果 | 妥当性 | 補正の必要性 |
|------------------------------|---------|---------|--------|--------------|
| A. 創発設計（ルール×自由度） |  5.0    | OK      | 妥当    | なし         |
| B. 世界設定（オリジナリティ）|  7.0    | OK      | 妥当    | なし         |
| C. 発展性（モジュラリティ＋将来展望） | 5.5 | OK | 概ね妥当 | なし（やや辛めだが許容範囲） |
| D. 技術実装（LLM＋Mem＋CodeQ＋SimCorrect）| 6.0 | OK | 妥当 | なし |
| **合計**                     | **23.5/40** | OK | 妥当 | なし |

**総合判定**: 元評価のスコア・根拠記述ともに高精度。引用は全て一次資料に照らして実在を確認できた。捏造・誇張は見られず、軽微な記述差はあるが補正は不要。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 5.0/10

**根拠の検証** (OK)
- `agent.py:451` 付近のストレス0.7/0.9 行動指示 → 実コードに完全一致して存在。「ストレスレベルが0.7以上の場合は、原則としてhabitatに戻って休息・回復することを優先してください。ストレスレベルが0.9以上の場合は、生命維持上の危険があるため、外部探査を継続しないでください」が確認できる。
- `agent.py:474-509` 役職別責任の埋め込み → Commander/Pilot/Mechanical Engineer/Life Scientist/Geologist/EVA Specialist/Communications Officer/Medical Doctor/Dog の9役職について、各々具体的な行動方針が長文で記載されている。Pilot「安全な移動と帰還可能性」、Geologist「Site_B の深層岩石、site_C の山岳地帯を重視」等、引用は正確。
- `debrief/mission_debrief.txt` → 「クルーはワンコの状況も毎日報告することが重要である」が実在。元評価の引用通り。
- `agent.py:216-235` `_build_fire_section` → 位置・強度・半径・距離のみを数値で渡し、「dangerous」「evacuate」等の質的記述を意図的に排除しているコメントが残存。元評価の「数値データのみ」という評価は正確。
- `simulation.py:317-340` `get_fire_info_for_agent` → 「Model B: only agents within each fire's radius get that fire's data」とコメントされた知覚境界実装。引用と整合。
- `agent.py:111-134` `get_nearby_agents` → 同一place もしくは外側同士のみ通信可能。引用通り。
- `config.yaml:20-117` places/fires 宣言 → 行番号もほぼ正確。
- `agent.py:190-214` `update_stress` → place名直書きの固定値（habitat -0.25, site -0.4 等）。LLM解釈の介在なし、これも正確。

**スコア妥当性**
世界ルール（火災・通信半径・ストレス・capacity）は丁寧に定量化されている一方、プロンプトに役職別行動方針と「ストレス閾値で何をすべきか」が明示的に書き込まれており、自由度を狭めている事実は確認できた。両軸のバランス取りとして 5.0/10 は適切。

**見落とし・誇張**
- 見落としなし。fire を半径内にしか知らせない設計、外/内の通信遮断などはむしろ加点要素として正しく拾っている。
- 誇張なし。「LLMに丁寧にロールプレイさせるシミュレーションになっている」という総評は実コードと合致。

**検証結論**: スコア・記述ともに妥当。

---

### B. 世界設定 — 元スコア 7.0/10

**根拠の検証** (OK)
- README 引用「チームに犬（ワンコ、Pochi）を入れた場合と化学者（Joe）を入れた場合の…比較」は実README に存在。
- README 「Dogチームはストレスが低い傾向（d=0.57）」「小サンプル（n=3）では効果量を過大推定（d=1.41→0.57）」「固着パターンはLLMエージェントの普遍的特性」も実在し、引用は文面通り。
- `simulation.py:232-242` のクルー9名構成（Aki Commander, Ryo Pilot, Ken Mechanical Engineer, Mina Life Scientist, Sora Geologist, Taro EVA Specialist, Yui Communications Officer, Hana Medical Doctor, Pochi Mission Dog）→ 完全に一致。
- `config.yaml:56-84` の中距離 Site_A/B/C と「途中に崖あり（別格）」と注記された遠距離 Site_D の差別化 → 実在。コメントも正確。
- `config.yaml:99-117` 火災イベント（power_plant_fire@step20, fire_1@step35, fire_2@step70）→ 一致。
- `Analysis_submitted_to_Hackathon/` には PDF（火星探査におけるワンコ効果_説明資料.pdf）とデモ動画（.mp4）が実在。

**スコア妥当性**
「コンパニオンアニマル効果」という仮説検証型の比較研究は単なる火星デモを超えるオリジナリティを持つ。30日期限・複数探査サイト・役職分化・火災イベントというミッション運用的構造も伴う。7.0/10 は適正水準。

**見落とし・誇張**
- 軽微な見落とし: README に「化学者（Joe）」と書かれているが、提出物では Dog 条件のクルーのみがハードコードされており、比較実験は実行スクリプト書き換えで実施したと推測される。元評価はこの点に触れていないが、減点には至らない。
- 誇張なし。

**検証結論**: スコア・記述ともに妥当。

---

### C. 発展性 — 元スコア 5.5/10

**根拠の検証** (OK)
- `config.yaml:20-117` places/fires/LLM/visualization の宣言的外部化 → 実在。
- `agent.py:190-214` `update_stress` の place名ハードコード（habitat/site_A-D/lab/power_plant/rocket_repair_base/greenhouse の if-elif チェーン）→ 確認。新 place 追加時にストレス影響は自動連動しない、という指摘は妥当。
- `simulation.py:232-242` クルー名・役職のハードコード → 確認。YAML 外でリスト化されている。
- `agent.py:402-509` ミッションプラン・役職責任の長文ハードコード → 確認。Site_A〜D の科学的目的説明や役職別責任が agent.py 内に直接記述されている。
- `ollama_client.py:13-26` → `repeat_penalty/repeat_last_n/min_p` を受け取るが本体ではフィールドに保存も使用もされておらず、`check_connection` も `bool(os.environ.get("OPENAI_API_KEY"))` のみ。指摘通り。
- README に将来展望なし → 確認。「環境設定、結果、解析、考察などはAnalysis内のPDFファイルを参照ください」と PDF に委譲しているのみ。

**スコア妥当性**
モジュール分割は良いが、ドメインロジック・プロンプト・クルー定義がコード側に固定化されており拡張コストが高い。README にも将来展望の本文記述なし。コード拡張性は中程度、将来展望は明示ほぼなしという観点から 5.5/10 は妥当。やや辛めにも見えるが、ハッカソンとして発展性の本文記述が乏しいという事実があるため許容範囲。

**見落とし・誇張**
- 見落としなし。
- 誇張なし。`OllamaClient` 命名・未使用引数の指摘も完全に正確。

**検証結論**: スコア・記述ともに妥当。

---

### D. 技術実装 — 元スコア 6.0/10

**根拠の検証** (OK)
- `ollama_client.py:35-44` Chat Completions 呼び出し、例外時空文字返却 → 完全一致。
- `ollama_client.py:46-61` `return` 後の到達不能コード（旧 `responses.create` 呼び出しが残存）→ 実在を確認。最初の `return response.choices[0].message.content.strip()` の後に、ifブロックと2つ目の try/except (`self.client.responses.create(...)`) が dead code として残存。
- `agent.py:563-598` `_extract_json_from_text` の中括弧深度＋文字列内エスケープを考慮した堅牢パーサ → 実在を確認。`in_string` トラッキングと `escape_next` も実装されている。
- `agent.py:713-722` LLM `memory` フィールドの自己フィードバック → `memory_entry = f"{self.name} ({self.role}) Step {step}: {memory_content}"` を append し、`memory_limit` 超過時に `pop(0)`。記述通り。
- `agent.py:171-188` メモリ／メッセージのコンテキスト抽出（recent_memory = `self.memory[-self.memory_size:]`、recent_messages = `self.received_messages[-self.message_context_size:]`）→ 確認。
- `simulation.py:342-466` 2フェーズ（メッセージ→行動）の同期ステップ → 確認。
- `simulation.py:436-437` `outside_position = (0, -self.half_space_size)` の2行重複 → 実在を確認。完全に同一行が2回書かれている。
- `agent.py:349-350` `print("DEBRIEF LENGTH:", ...)` と `print(mission_debrief[:100])` → 実在を確認。
- `simulation.py:546-578` 同一CSV保存処理の2回記述 → 実在を確認。run メソッド内に最初の保存ブロック（latest + timestamp）と、`logger.info("Simulation completed")` 以降の dead な2つ目の保存ブロック（`agent_logs.csv`）が並存。
- `config.yaml:87-96` `repeat_penalty / repeat_last_n / min_p` 設定値 → 実在を確認。コードでは未使用。
- `gender = random.choice(["male", "female"])` が犬にも適用 → `simulation.py` で確認。Pochi にも性別がランダム割り当てられる。

**スコア妥当性**
- LLM3観点: 2段階プロンプト（メッセージ生成→位置情報付き行動決定）、JSON 抽出の堅牢化、フォールバック設計は中の上。`max_tokens=200` の短さ、リトライ・スキーマ検証なし、temperature 固定はマイナス。
- メモリ3観点: LLM 自身に次回記憶を生成させる自己要約型、`memory_limit`/`memory_size`/`message_history_limit`/`message_context_size` で別個に管理は中の上。
- コード品質: dead code（`return` 後）、CSV 二重保存、変数重複代入、デバッグ print、未使用引数、命名矛盾（`OllamaClient`→OpenAI）、シード未指定など複数の品質欠陥が確認できる。
- シミュレーション正確性: 2フェーズ同期、知覚境界、ログ整備は丁寧。レース条件の懸念なし。

総合して 6.0/10 は妥当。コード品質が突出してマイナスを引いており、メモリ・LLM・シム正確性の良さがそれを支えている形。

**見落とし・誇張**
- 見落としなし。むしろ非常に細かい欠陥（CSV 二重保存・dead code・変数重複代入・犬の性別ランダム）まで網羅的に拾っている。
- 誇張なし。すべての指摘がソースに対応している。

**検証結論**: スコア・記述ともに妥当。

---

## 総合所見

元評価 `mars-llm-simulation-wanko_eval.md` は、引用された全ての行番号・文面・コードパターンが実ファイルと一致しており、検証範囲では捏造・誤読・誇張は確認できなかった。特に評価者は以下のような細部まで一次資料から拾っており、レビューの信頼性は高い:

- `ollama_client.py` の `return` 後 dead code
- `simulation.py:436-437` の2行同一代入
- `simulation.py:546-578` の同一 CSV 保存ロジック二重化
- `OllamaClient` 命名と OpenAI 実装の矛盾
- `repeat_penalty` 等のconfig値がコード側で未使用
- `update_stress` の place名ハードコード
- mission_debrief.txt の「ワンコの状況も毎日報告」というメタ指示

スコア配分も妥当で、特に A（5.0）は「世界ルールの精度は高いがプロンプト側で行動指示が大量に与えられている」という観察に裏付けされており、創発設計の評価ロジックとして筋が通っている。B（7.0）の「ペット犬という社会科学的×ミッション運用的なオリジナリティ」、D（6.0）の「LLM・メモリ設計の丁寧さとコード衛生の甘さの綱引き」も適切なバランス。C（5.5）はやや辛めの印象を受けるが、README が PDF に発展展望を委譲しており本文上に技術的拡張案が乏しい事実は確認できるため減点理由は成立する。

**判定: 元評価合計 23.5/40 を支持する。スコア補正の必要なし。**

評価レポートとしての完成度は高く、改善点・提言の方向性（プロンプトから具体的行動指示を剝がして世界ルールに集約、ドメインロジックの YAML 化、乱数シード固定、コード衛生）も submission の弱点と正確に対応している。レビューワー側で追加すべき指摘は特に見当たらない。
