# 評価レポート: singulab

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

「ルールでは書けない創発」を観察することを目的とし、LLM エージェントが住む小さな社会を 3 階層フレームワーク（1F 物：ペルソナ／場所／所持物、2F 環境：家族・趣味・気分などの personal_context、3F 物理：通信半径・認知上限・距離依存知覚）で構築するマルチエージェント・シミュレーション。エージェントは 9 ヵ国の国籍・MBTI 16 タイプ・年齢 20-60 のランダムサンプリングから生成され、`config/base.yaml` をベースに各シナリオ YAML（alien / zero_gravity / bar_fire）が extends + deep_merge で重ね合わされる。Qwen3 4B abliterated（Ollama）が `think:false` 設定で駆動し、二重メモリ（個人記憶＋LRU 社会記憶）、`personal_contexts` 状態記述、events 抽象基底による複数シナリオ実装が組み合わさる。主要メカニズムとして、capacity 違反時の `place_entry_denied`、距離依存知覚（near/mid バンド＋圏外）、`message_evicted / re_encountered` といったエコー対策が実装され、Gini 係数・silent rate・pair_stability・event_counts などの指標が `runlog/metrics.py` で集計される。実験は集団規模を独立変数として 5 体ラン vs 100 体ランの並列比較を中心に行われ、シナリオは UFO 飛来・無重力・バー火災の 3 種が用意されている。テーマは、3 階層的に最小定義された世界で、役割の自然発生・ハブの自然発生・社会構造の量的分化など、明示的指示なしに創発する社会パターンを観察することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  8   |  -   |    8.0    |
| B. 世界設定  |  8   |  7   |  -   |    7.5    |
| C. 発展性    |  8   |  8   |  -   |    8.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  32  |  31  |  -   |  **31.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 8.0/10

#### 評価
世界ルールの設計精度は非常に高い。3 階層フレームワーク(1 階:物/Persona、2 階:環境/景気、3 階:物理/通信半径・認知限界・メッセージ遅延)が明確に分離され、入れ子 place の半径最小優先・capacity 強制・UFO の距離依存知覚帯(near/mid/far)・ダンバー数 LRU による memory_evicted/re_encountered イベントなど、エージェントが意味のある差を出せる構造的足場が揃っている。エージェントへは raw data(占有率・UFO までの距離・知覚帯)を渡し、`role`/`speaking_style` を明示的に排除した `personal_contexts`(家族・趣味・気分など生活文脈)に切り替えた点は、行動の自由度を高める明確な設計判断である。一方でメッセージ生成プロンプト側に「質問・確認・提案を含めてください」「抽象語だけで終わらせないでください」「EVENT セクションを読んでください」などの誘導が残っており、フル 9-10 までは届かない。それでも全体としては「ルール × 自由度」のバランスが良く、創発ポテンシャルは高い。

#### 根拠
- `src/physics/cognition.py:60-110` — ダンバー数 LRU + `memory_evicted` / `memory_re_encountered` イベント発火。記憶量を物理法則として制約。
- `src/events/alien.py:103-150` — UFO の距離依存知覚(near/mid/far の 3 帯、far は会話経由でのみ伝播)。世界ルールが情報伝達非対称性を生む。
- `src/world/world.py:80-140` — `attempt_move` が capacity 違反を `place_entry_denied` イベントで返す中央管理。
- `src/agent/agent.py:139-160` — `_build_messages_context` が「直前 1 件のみ + 引用枠 + 自分の言葉で返す」形式に変更されコピペ抑制(エコー対策)。
- `src/simulation.py:434-470` — `personal_contexts` は「役割を演じてください」ではなく状態記述として与える設計コメント。役割直書きを排除。
- `src/agent/agent.py:286-320` — ただし `task_section` で「相手に向けた具体的な一言。状況確認・質問・提案のいずれかを含める」と行動形態を限定する誘導あり。
- `README.md` — "LLMエージェントが住む小さな社会を構築し、**ルールでは書けない創発**が起きるかを観察する。"

---

### B. 世界設定 — 最終: 7.5/10

#### 評価
シナリオは「日本のオフィス(日本人 90% + 外国人 10%、MBTI 16 タイプ、年齢 20-60)に突発イベント(UFO 接触/無重力/火事)を投じ、人数規模を独立変数として役割・ハブ・階層が自然発生するかを観察する」というもので、単なる避難デモを超えた社会学的観察軸を持つ。`personal_contexts`(家族構成・趣味・気分・最近の悩み)で個性のばらつきを与え、エコーループの抑制と多様反応の両立を狙うのは、社会シミュレーションとして実務的・学術的に意味がある設定。提出資料には 5 体 vs 100 体の比較で「役割の自然発生・ハブの自然発生・3 層構造の量的分化」が 100 体側でのみ観察されたと記録されており、設定に対する成果回収もできている。ただし舞台は「オフィスフロア 1 つ」と限定的で、市場・組織間競争・資源配分などへの広がりはまだ。

#### 根拠
- `README.md` — "3階層フレームワーク に沿って世界を設計" / 1 階(ペルソナ/場/所持物)、2 階(personal_context)、3 階(通信半径・認知上限・距離依存知覚)の明示。
- `config/base.yaml` — 国籍プール 9 ヵ国 + 重み 90/1.25×8 で「日本人 90%・外国人 10%」、MBTI 16 タイプ、年齢 20-60 のペルソナ設計。
- `config/scenario_alien_5.yaml` / `config/scenario_zero_gravity_5.yaml` / `config/scenario_alien_100.yaml` — 5 体/100 体 × UFO/無重力/火事の比較設計。
- `docs/20_報告資料/提出物サマリ.md` — "5 体ランでは出現せず 100 体ランで初めて出現した現象として、**役割の自然発生(専門家モード)・ハブの自然発生・3 層構造の量的分化** を観察できた"
- `src/world/environment.py:17-23` — `economy: good/bad/neutral` を状態記述として挿入(命令文を含まない)。

---

### C. 発展性 — 最終: 8.0/10

#### コード拡張性
モジュール分離が徹底されている。`src/world/`(物)・`src/physics/`(物理)・`src/events/`(イベント)・`src/agent/`(エージェント)・`src/llm/`(LLM クライアント)・`src/runlog/`(ログ)・`src/viz/`(可視化)が明確に分かれ、`src/simulation.py` が層を跨ぐ責務を引き受ける。新イベントは `Event` 抽象基底を継承して `maybe_activate`/`perceived_info`/`state` を実装するだけで追加でき、`scenario_*.yaml` の `events:` セクションに type を増やせば動く(fire/alien/zero_gravity の 3 つで既に拡張性が実証)。設定は `extends:` で YAML 継承する仕組みがあり、シナリオ側は差分のみ書ける。`PersonaFactory` と `personal_contexts_pool` で 100 体規模にも対応。

#### 将来展望
README・CLAUDE.md・docs/03_ToDo/・docs/20_報告資料/提出物サマリ.md にロードマップが整理され、提出物サマリでは「複数 seed による分布取得」「34B モデルでの再現」「組織構造観察の深堀り」などが残課題として明示されている。`message_delay` は値保持のみで配送キュー未実装などの拡張余地も明記。ただし将来の研究的方向は「スケールアップ」「より大きいモデル」が中心で、概念的に新しい方向(複数組織間の相互作用、外的市場、エージェント階層の階層化など)への具体的展望はやや薄い。

#### 根拠
- `src/events/base.py` — `Event` 抽象基底クラス。`maybe_activate`/`perceived_info` のみ要求しプラグイン可能。
- `src/events/alien.py` / `src/events/fire.py` / `src/events/zero_gravity.py` — 3 種イベントが同パターンで実装され拡張性を実証。
- `src/config_loader.py` — `extends:` 継承 + deep_merge。
- `src/agent/persona.py:97-126` — `PersonaFactory` + legacy スキーマ拒否(`_is_legacy_config`)で旧形式の混入を防ぎ移行を強制。
- `docs/20_報告資料/提出物サマリ.md` — "未解決課題: §7.3 複数 seed による分布取得は未実施"
- `CLAUDE.md` — "Phase 2 で `src/llm/base.py` の抽象化 + httpx async に置き換え予定"(LLM クライアント抽象化計画あり)

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
Ollama 経由で Qwen3 4B abliterated を運用。`think:false` パラメータ対応で thinking 抑制(検証ログで eval_count 約 17 倍効率化を確認)。M-08/M-09 の検証に基づき `temperature=0.8` / `max_tokens=100` / `min_p=0.05` / `repeat_penalty=1.1` まで実験的に絞り込み、エコー対策として根拠を持つチューニング。プロンプトは構造化された JSON 出力を要求し、`Agent._extract_json_from_text` がブレース対応で堅牢にパース、失敗時は方向抽出フォールバック。日本語強制セクションを `=== LANGUAGE REQUIREMENT (CRITICAL) ===` で明示挿入。エラーハンドリングは LLM 失敗時に空メッセージ/`stay` を返すのみで、リトライや指数バックオフは無い点はやや弱い。同期 `requests` で 100 体並列ではなく逐次。

#### メモリ設計
2 種のメモリを併用。(1) `Agent.memory`(自己反省ログ、`memory_limit=20`)、(2) `AgentMemory`(認知限界 LRU、`cognitive_limit=10/30`)。後者は他者ごとに `last_interaction_step` を持ち、超過時に LRU 追い出し + `memory_evicted` イベント、再接触時に `memory_re_encountered` + `gap_steps` を発火するという「忘却 → 再会」のセマンティクスを持つ設計。受信メッセージは history_limit でリングバッファ化、プロンプト側は直前 1 件のみ引用枠で挿入(エコー抑制)。単なるログダンプではなく、社会記憶モデルとして意味のある実装。

#### コード品質・シミュレーション正確性
責務分離が明確、関数・クラスのコメントが充実、Python 型ヒント + TypedDict あり。`step_simulation` は (1)Event 発火 → (2)update_state → (3)decide_message → (4)receive_message → (5)decide_action → (6)attempt_move → (7)update_state という固定順で、メッセージと行動を同 step で順次処理する明示的設計。`random_seed` で `random`/`numpy` 両方を固定し、`run_metadata.json` にシナリオ・seed・ペルソナ・LLM 設定を記録(再現性を担保)。テストは 6 ファイル(events/persona/metrics/environment/physics/world)。`MetricsCalculator` で発言 Gini・沈黙率・トップ発話者シェア・ペア安定度・events 集計を計算。並列ステップ更新でなく逐次のため race condition は無いが、メッセージ送信中に位置が変化する race は構造上回避済み。

#### 根拠
- `src/llm/ollama.py:60-95` — Ollama 呼び出し + think パラメータ + try/except のフォールバック。
- `src/agent/agent.py:381-420` — JSON 抽出のブレース対応スキャナ + 方向フォールバック抽出。
- `src/physics/cognition.py:48-100` — `AgentMemory.record_interaction` が LRU eviction + `memory_re_encountered` を返す設計。
- `src/simulation.py:60-65` — `random.seed` + `np.random.seed` 同時固定で再現性。
- `src/runlog/metrics.py:79-200` — Gini / silent_agent_rate / pair_stability / event_counts の集計。
- `tests/test_phase5_events.py` — start_step 前後・距離帯別の知覚をテスト。
- `config/base.yaml` — "max_tokens 512 → 100" / "temperature 0.5 → 0.8" のチューニング根拠が config コメントに記録。
- `src/agent/agent.py:557-580` — LLM 失敗時のフォールバック(空メッセージ・`stay`)。リトライ無しは改善余地。

---

## 総評

### 優れている点
- 3 階層フレームワーク(物/環境/物理)が概念だけでなくモジュール構造として実装されており、Event 抽象基底・PersonaFactory・WorldLaws・CommunicationPhysics・AgentMemory(LRU)など値オブジェクト・物理層・認知層が分離され、拡張ポイントが明確。
- 「役割を演じさせる」設計を意識的に排除し(legacy `conversation_profiles` → `personal_contexts` への移行)、生活の文脈ベースに切り替えることで創発の足場を作る、というスタンスがコード・コメント・議事録レベルで一貫している。
- 5 体 vs 100 体の比較を実走し、100 体側でのみ「役割の自然発生・ハブ形成・3 層分化」が観察された、という結果まで回収できている(提出物サマリで明示)。
- 検証ログ(M-XX)・問題点リスト・要件定義の整備が手厚く、`temperature` や `max_tokens` のチューニング、エコー対策の効き、距離依存知覚への切替(段 3)などすべてエビデンス付きで進めている。
- 創発指標(Gini・沈黙率・トップ発話者シェア・ペア安定度・events 集計)が tools 経由で算出可能で、観察を定量化する基盤がある。

### 改善点・提言
- メッセージ生成プロンプトに残る「質問・確認・提案を含めてください」「抽象語だけで終わらせないでください」「EVENT セクションを読んでください」などの誘導をさらに薄くし、純粋な状態提示のみに近づけると A の創発設計はもう一段伸びる。
- LLM クライアントが同期 `requests`(逐次)+ 失敗時に静かに空応答を返す実装のため、リトライ・タイムアウト分離・部分並列化(あるいは httpx async 化、CLAUDE.md にも計画記載あり)で堅牢性と速度の両方を改善できる。
- 世界設定が「オフィスフロア 1 つ + 突発イベント」に集中しているため、複数組織間の相互作用や外部市場・予算・タスク量のような 2 階(環境)の動的化が入ると、より広範なビジネス/社会科学の議論に繋がる。
- `message_delay` は値保持のみで配送キュー未実装の旨が明記されているので、3 階の遅延配送を実装すると情報非対称性をさらに観察可能になる。
- 全ラン `seed=42` 単発のため、複数 seed による分布取得(提出物サマリ §7.3 にも未解決と明記)を行うと、観察結果の頑健性が示せる。

### 一言コメント
3 階層フレームワークを忠実にコード化し、LLM プロンプト設計と物理ルール設計の両側からエコー抑制と創発誘発に取り組んだ、設計・実装・観察のループが回っている良質な提出物。5 体と 100 体の比較を独立変数として活用し、定量指標と定性観察の両方を残した点は特に評価できる。
