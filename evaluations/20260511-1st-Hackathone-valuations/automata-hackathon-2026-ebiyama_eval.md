# 評価レポート: automata-hackathon-2026-ebiyama

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

品川駅周辺を舞台とした「企業協力型 学外教育プログラム」を題材に、教育・まち・企業のカケザンで生まれる学習環境を LLM マルチエージェントで再現するシミュレーション。10 人の生徒エージェント（高校生／社会人／小学生の 3 バリアント、外国人ルーツ・車いす利用・学校不適応など多様なペルソナを含む）と、品川駅周辺 12 社の企業担当者ホストエージェントが登場する。全体は 3 フェーズ構成で、Phase A（教室 AB 座学・30 step）、Phase B（品川駅前 160m×160m の屋外フィールドワーク・50 step）、Phase C（観察者 LLM 経由のアンケート）の連続した学習プロセスとして実装される。LLM が「どこに行きたいか」「誰に話しかけるか」「何を学んだか」を自律決定し、各フェーズの memory・発話・行動ログが次フェーズへ引き継がれる。派生元（兵頭氏の 2D 火事避難シミュ）のコア（定量情報のみプロンプト提供、双方向同時発話排除、Jaccard 類似フィルタ、memory ローリングバッファ）を継承しつつ、3 層行動モデル（transit / observe / interact）、awareness 伝播、constrained mode、time-of-day guardrail などを追加。LLM バックエンドは Gemini 2.5 flash-lite に切替えられ、3D ビューア・統合 HTML/PDF レポートビルダも同梱。テーマは、まち全体を学び場とした際の子どもたちの視野・問い・主体感の変化、多様な参加スタイルの成立、企業・地域連携に必要な設計要素の検証である。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  6   |  6   |  -   |    6.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  8   |  8   |  -   |    8.0    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    |  32  |  32  |  -   |  **32.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 6.0/10

#### 評価
世界ルールの設計精度は非常に高い。160m × 160m の品川駅周辺フィールドを 111 places、scene_3d (roads/decks/stairs)、navigator による walkable mask、perceive_pass / perceive_enter による現地知覚情報注入、3 経路 awareness 伝播 (direct / conversation / notification)、internal_state (energy / hunger / social_fatigue)、relationship × proximity × emergency_boost による should_speak 確率ゲートなど、厳密な仕組みで構築されている。プロンプトは「strictly quantitative data ... No qualitative labels」を明示し、定量データのみ提供する原則を継承している (Phase 0 由来)。一方で、本案 (Phase B FW シミュ) では行動への誘導が強い: WORKING STATE ブロックに「必須2社以上 / 残りN社 / 未訪問の企業担当者リスト」を毎step注入し、current_goal で「序盤 (前半30step) は積極的に動いて広く歩き回ること」「同じ場所を何度も再訪することは避け」と直接指示し、school_fit='不適応' persona には「自分からは話しかけない」と対人行動を直に指定している。さらに、host (企業担当者) は LLM call をスキップして action_type='stay' に強制 (simulation.py:1952)、終盤残り5step以下では rule-based で walk_toward 東西自由通路 に上書き (agent.py:2322) するなど、創発させたい場面でも構造的な制約・スクリプト介入が混じる。結果として、世界ルール 8/10 × 行動自由度 4-5/10 のバランスで中位 (6) と判断した。

#### 根拠
- `agent.py:922-989` — `_build_working_state_block` が WORKING STATE として「必須N社 / 未訪問の企業担当者リスト (max 6件) / まだ確かめていない問い (max 3件) / 終盤戻りリマインダ」を毎step user_prompt に注入。
- `agent.py:2317-2332` — 残り5step以下では LLM 出力を無視して `decision['action_type']='walk_toward'`, `target_place='東西自由通路'` に rule-based 強制上書き。
- `simulation.py:1952-1974` — host persona は Phase 3 (decide_action) を完全スキップし `action_type='stay'` 固定。LLM call なし。
- `agent.py:1079-1091` — `school_fit='不適応'` の persona には「自分からは話しかけない (挨拶も自発的にはしない)」「基本単独行動を好み」と対人行動を system_prompt に固定挿入。
- `agent.py:1393-1394` — "strictly quantitative data: coordinates, distances, occupancy counts, capacities, occupancy rates, fire intensities, fire radii. No qualitative labels ... are provided." (定量データのみ提供の原則は明示)
- `simulation.py:984-1056` — awareness 伝播の3経路 (direct / conversation / notification) は creative な世界ルール設計。phone_check_rate で persona ごとに通知到達率を変える設計も解釈余地を残す。
- `place_types.py:20-140` — place type ごとに social_likelihood, atmosphere, typical_stay_duration を定義。
- `README.md` — 「LLM が『自分はどこに行きたいか』『誰に話しかけるか』『何を学んだか』をすべて自律決定し」と謳う一方、本コードでは終盤帰路強制・host不動・不適応persona発話抑制など rule-based 介入が併用される設計。

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
独創性とテーマ性が極めて高い。題材は「教育 × まち × 企業」のカケザンで生まれる学外教育プログラム — 品川駅周辺 160m × 160m の実在街区を舞台に、高校生10人 (社会人/小学生 variant もあり) と企業担当者12社のホストエージェント、座学 (Phase A 30step) → フィールドワーク (Phase B 50step) → アンケート (Phase C) という3フェーズの一連の学習プロセスを再現する。多様な子ども (school_fit='不適応', mobility='wheelchair', nationality='western/asian', 外国籍3-4名) を必ず含めることで、「参加スタイルの差はどう生じるか」「企業対話は問いをどう深めるか」といった社会科学的問いを直接モデル化している。実在の品川 (旧東海道宿場、JR/京急/新幹線結節、リニア中央新幹線2027開業、TAKANAWA GATEWAY CITY 再開発、御殿山緑地、食肉市場など) の歴史・地形・現在進行中の再開発を具体的に context として注入し、現地で見える光景・動線・雰囲気を perceive_pass / perceive_enter として place ごとに記述している。教育研究 / まちづくり研究 / マルチエージェント・シミュレーション研究 のいずれにも直接接続する深いシナリオで、独創性 + 実問題への接続性ともに最高水準。

#### 根拠
- `README.md` — 「**作品テーマ**: 「**教育 × まち × 企業**」のカケザンで生まれる、新しい学びの形の検証・シミュレーション」「題材は **品川駅周辺での「企業協力型 学外教育プログラム」**。10人の生徒... と、品川駅周辺 12 社の企業担当者を LLM エージェント化」
- `README.md` — 「Phase A — 教室での座学 / Phase B — 品川駅前フィールドワーク / Phase C — シミュ後アンケート」の3フェーズ構造。
- `README.md` — 「**多様な子どもたちの参加スタイルが成立する学習環境になっているか** — 学校適応度・国籍・身体特性・関心領域などの差を持つ生徒が...」と研究上の3つの問いを明示。
- `configs/config_shinagawa_field_elementary_v2_100step.yaml:1030-1250` — 各 place に perceive_pass (横断歩道脇から見えること) / perceive_enter (中に入って気づくこと) を事実ベースで具体記述。実在地点の knowledge を再現。
- `docs/shinagawa_context.md` — 品川駅前ガイド (港南口 vs 高輪口 の性格差、JR/京急改札前、東西自由通路、北口広場(工事中)、御殿山緑地、食肉市場など) を fact-based に整備。
- `agent.py:1140-1156` — FW 前提コンテキスト (「13時集合・昼食後・午後 FW・東西自由通路スタート、品川の二面性 (歴史と現代・港南と高輪)」) を全 agent 共通で system_prompt に注入。
- `tools/run_survey.py:1-300` — Phase C アンケートで「観察者LLM経由で agent の memory を読ませて『この人物がアンケートにどう答えるか』を推測」する設計 (本人忖度バイアス回避)。
- `simulation.py:225-263` — 一般化された events (transit_disruption / last_train) を 3 経路で agent に伝播させる仕組みは、社会的情報拡散の研究的価値も付随する。

---

### C. 発展性 — 最終: 8.0/10

#### コード拡張性
モジュール分離は良好。`agent.py` (エージェント挙動 + プロンプト) / `simulation.py` (時間進行 + フェーズ制御 + 統計) / `navigation.py` (歩行可能マスク + 経路) / `place_types.py` (場所種別 registry) / `llm_client_factory.py` (Claude / OpenAI / Gemini 切替) / `utils.py` (persona 生成) / `reporter.py` / `visualization.py` がそれぞれ独立。設定は完全に YAML 外部化されており、12個以上の config (classroom_ab_a/b、shinagawa_field_high/elementary、jr_disruption、last_shinkansen、smoke 等) が並存。`tools/` 配下に 20 個以上のユーティリティ (config builder、persona generator、report renderer、3D viewer bundler、scene exporter、survey runner 等) があり、新しい variant (社会人 / 小学生 / 別シナリオ) を追加するのが容易。`tools/build_classroom_ab_config.py` で persona の variant {high|adult|elementary} を自動生成可能。place_type を追加するには `place_types.py` の dict に1エントリ追加するだけ。LLM プロバイダの差し替えも `llm_client_factory.create_llm_client()` 経由で透過的。

#### 将来展望
README に「Phase A 固定 → B, C 独立」のアーキテクチャ方針が明示されており、Phase A 出力を YAML として凍結することで Phase B/C を独立に試行錯誤できる設計判断が書かれている。3 variant (高校生 / 社会人 / 小学生) の併走が既に動作。一方で、長期的な研究ロードマップ (例: より多くの街への展開、リアルなデータと突き合わせた検証、エージェント拡張の方向性) は本案 (run #157) の成果物説明が中心で、抽象化レベルでの拡張ビジョンは README 上はやや薄い (5/10 程度)。コード拡張性 (4/5) + 将来展望 (4/5) の総和で 8。

#### 根拠
- ファイル分割の明確さ: `agent.py` (2609行) / `simulation.py` (2245行) / `navigation.py` (442行) / `place_types.py` (154行) / `llm_client_factory.py` (439行) / `utils.py` (154行)。
- `llm_client_factory.py:405-439` — `create_llm_client(llm_config)` で provider (anthropic / openai / google) を YAML から透過的に切替。
- `configs/` 配下 12+ YAML — classroom_ab_a/b/adult_a/b/elementary_a/b、shinagawa_field_elementary、jr_disruption、last_shinkansen、smoke 等の variant が同居。
- `tools/build_classroom_ab_config.py:67-87` — variant 指定で persona 生成。`tools/build_shinagawa_field_config.py:215, 374-399` で FW config を Phase A 出力から自動構築。
- `place_types.py:20-140` — place_type は registry pattern。新規 type 追加は dict 1 行。
- `README.md` — 「**Phase A 固定 → B, C 独立** のアーキテクチャ方針: Phase A 出力を YAML として凍結することで、Phase B/C を独立に試行錯誤できるよう設計」
- `simulation.py:225-263` — events (transit_disruption / last_train) は config の `events:` key で1エントリ追加するだけで活性化、`_SUPPORTED_EVENT_TYPES` の判定で型を拡張可能。
- 長期ロードマップ (別の街、別の世代、研究検証との接続) の具体プランは README 上では薄め。

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
非常に洗練されている。Gemini 2.5 flash-lite を主要プロバイダに採用し、`GeminiClient` は (a) hash ベースの CachedContent 再利用 (sha256(system_prompt) を key にした double-checked locking)、(b) `response_mime_type='application/json'` + `response_schema` による構造化出力強制 (`_pick_gemini_schema` で system_prompt を見て message / message_with_memory / action のいずれかを自動選択)、(c) rate-limit 時の指数バックオフ (`_retry_backoff` 1→2→4→8s)、(d) chat session thread モード (moltbook 風) の実験経路、(e) usage_metadata の input / cache_read / output ロギング を備える。プロンプトは system_prompt にキャッシュ可能な静的部分を集約し、user_prompt に動的部分を分離する設計。truncation 検知 (`_is_truncated_response`) で `MAX_RETRY_TOKENS=1500` での再試行も実装。

#### メモリ設計
3 層メモリ構造: (1) `initial_memory` (Phase A handoff: 未来像 + FW 意図 + 確かめたい問い) を起動時に prepend、(2) rolling raw memory (memory_size=5、memory_limit=20)、(3) `archived_summaries` (5stepごとに直近5件を LLM で1文要約して push、固定保管で消えない)。`_build_memory_context` で「訪問履歴 + 圧縮記憶 + 直近raw」を順に組み立てる。さらに `working_state` (現在進捗・未解決問い) を別経路で system_prompt 末尾に固定挿入し、rolling buffer の押し出しに依存しないようにしている。message ループ抑制は (a) per-partner 直近3発話履歴の可視化 + ⚠ 警告、(b) Jaccard 4-gram 後フィルタによる強制 silent 化、の二段構え。awareness 伝播は別チャネル (`known_events` set, `awareness_source` dict)。

#### コード品質・シミュレーション正確性
コード規模は大きい (Python だけで約 7800 行 + tools 8770 行)。`agent.py` は 2609 行とやや monolithic だが、内部は private helper method (`_build_*_section`, `_create_*_prompts_minimal`) に丁寧に分解されており、日本語コメントで設計判断 (例: 「Policy D」「smoke14: 後フィルタで…」) が随所に残されている。シミュレーション正確性: (a) Phase 1 (message) → Phase 2 (broadcast) → Phase 3 (action) → Phase 4 (move) の同期実行を `step_simulation()` で実装、(b) JSONL writer は per-sink threading.Lock で保護、(c) `random.seed(seed)` + `np.random.seed(seed)` で再現性確保、(d) double-bidirectional speech は `partner_map` 一括収集 → A→B かつ B→A の場合 id 大きい方を silent 化、(e) 並列 LLM call は `ThreadPoolExecutor` で `parallel_workers=16`、(f) place 境界横断時は `current_intent=None` で transit cache を invalidate。ロギングは messages.jsonl / memory_reasoning.jsonl / actions.jsonl / should_speak_log.jsonl / awareness_propagation_log.jsonl / event_awareness_log.jsonl / relationships_timeline.jsonl の7系統を分離出力し、後段の分析 (`render_v3_report.py`) で活用。

#### 根拠
- `llm_client_factory.py:210-241` — Gemini CachedContent の hash-based double-checked locking 実装。
- `llm_client_factory.py:30-72, 261-264` — `_pick_gemini_schema` で system_prompt の "action_type" 有無を見て structured-output schema を自動選択。
- `llm_client_factory.py:25-27` — `_retry_backoff` 指数バックオフ。`MAX_RATE_LIMIT_RETRIES=4`。
- `agent.py:228-240` — 3層メモリ: `initial_memory` prepend + rolling `self.memory` + `archived_summaries`。
- `agent.py:1004-1034` — `maybe_compress_memory(step)` で 5step ごとに直近 5 件を Gemini で 1 文要約して `archived_summaries` に push。
- `agent.py:415-458` — `_build_memory_context` が「訪問履歴 + 要約#N + 直近raw」を組み立て。
- `agent.py:477-509` — `filter_loop_message` で Jaccard 4-gram >= 0.50 のループ発話を強制 silent 化。
- `agent.py:540-575` — `_build_per_partner_history` でループ検知された相手には ⚠ 警告 + 同テーマ送信禁止指示。
- `simulation.py:56-62, 1700-1830` — 4-Phase step 同期実行 + `ThreadPoolExecutor` 並列 LLM call + per-sink lock で JSONL 書き込み保護。
- `simulation.py:65-73` — seed 設定で `random.seed` + `np.random.seed` 両方を初期化。
- `simulation.py:1790-1797` — 双方向同時発話排除 (A→B かつ B→A は id 大きい方を silent)。
- `agent.py:2229-2249` — `determine_behavior_layer` で transit / dwelling / interacting を判定、transit は cache_intent を最大5step再利用して LLM call 節約。
- `agent.py:696-710` — `_is_truncated_response` でブレース不整合を検出し `MAX_RETRY_TOKENS=1500` で再試行。

---

## 総評

### 優れている点
- 「教育 × まち × 企業」の3フェーズ学習プロセス (座学 → FW → アンケート) を実在街区 (品川駅前) で再現するシナリオの独創性・社会的価値が極めて高い。多様な背景の生徒 (車いす利用、外国籍、学校不適応) を必ず含める設計で、教育研究・まちづくり研究の双方に直結する問いを扱っている。
- LLM 統合の作り込みが本格的: Gemini context caching (hash + double-checked locking)、structured-output schema 自動選択、3層メモリ (initial + rolling + LLM圧縮 archive)、Jaccard 4-gram ループ抑制、awareness 3経路伝播、behavior_layer cache、双方向同時発話排除、truncation 再試行など、コスト削減と品質維持の両立が随所に効いている。
- モジュール分離と config 駆動の徹底度が高い。LLM provider 切替、3 variant (高校生/社会人/小学生)、Phase A/B/C 独立イテレーション、tools 群 (20+ scripts) で新シナリオ追加が容易。
- 7系統の JSONL ログ + 3D ビューア + 統合HTMLレポート (PDF化) の事後分析パイプラインまで含めて成果物の整え方が成熟している。

### 改善点・提言
- 創発設計の観点では、Phase B FW シミュにおいて WORKING STATE ブロック (必須2社 / 未訪問リスト / 序盤30step積極移動 / 同じ場所再訪避ける / 終盤戻り) が毎step prompt に注入され、さらに host=不動の rule-based 固定、残り5step以下の walk_toward 強制上書き、school_fit='不適応' persona の発話抑制ルールなど、行動への scaffolding がやや強い。創発をより純粋に観察したいなら、これらを「世界ルール側の制約」(例: host は受付カウンタという物理的場所に拘束されるので動けない、終了時刻になったら帰る判断は agent 側に委ねる) として表現し直すと評価軸 A のスコアが伸びる。
- school_fit / mobility / nationality 等の persona 属性をプロンプトで明示する手法は安全だが、「自分からは話しかけない」のような対人行動指示は LLM が解釈で振る舞いを再生産する形に寄せられるとなお良い (例: 過去の対人成功体験の少なさを memory として与え、talkativeness 値だけを下げる)。
- README は本案 (run #157) の成果物説明が中心で素晴らしいが、研究としての長期ロードマップ (別の街・別の世代・実データ突合・抽象シナリオ化など) を1セクション足すと、発展性 (C) の将来展望側で伸びる。
- agent.py が 2609 行と単一ファイルとしては大きい。プロンプト構築部 (`_build_*` / `_create_*_prompts_*`) を別モジュール `prompts/` に切り出すと可読性・再利用性が上がる。

### 一言コメント
派生元 Phase 0 の 2D 火事避難シミュを起点に、教育プログラム検証という独自テーマへ大幅拡張した完成度の高いハッカソン提出物。シナリオの独創性・社会的意義・実装の作り込み (キャッシング、メモリ階層、awareness 伝播、Jaccard ループ抑制) は突出しており、創発設計面での scaffolding をやや緩めるとさらに本質的な観察対象になる。
