# 評価レポート: hackathon-singulab-inu

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

GOOD ECHO は、ISS のような閉鎖空間を題材に、ストレス・混雑・孤立・対人摩擦の下で「善性」が空間・物・ルール・会話の設計によってどう支えられるかを検証する LLM マルチエージェント・シミュレーション。20 人規模のエージェントが性格・関係性・役割を持ち、ISS の各部屋を移動しながら生活・会話・衝突・再接触を行う。ナッジオブジェクト（持ち寄り棚、OK サイン、リソース・スコアボード、投票パネル等）、ルール（短い確認、静穏時間、再開時刻）、修復／衝突イベントを介入要素として、A（ナッジなし）、B（標準ナッジ）、C（ナッジ撤去）、D（ルールのみ）、E（ナッジのみ）の 5 条件で比較する。指標として発話数・衝突・修復的 status・孤立・媒介物語彙などを記録し、修復イベントなし・喧嘩イベントなしの補助軸も設けている。テーマは「仲直りを増やす装置」ではなく、高ストレス下で関係修復のための足場（戻り口）を残す環境インフラの設計であり、ISS から災害避難所・病棟・介護施設等への転用可能な閉鎖環境設計フレームを志向する。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  7   |  7   |  -   |    7.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  9   |  9   |  -   |    9.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  33  |  33  |  -   |  **33.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 7.0/10

#### 評価
世界ルールは精密に設計されている（モジュール定員と占有率、半径ベースの通信ルール、同一場所内のみ会話可、火災は半径内のみ知覚可、6種の圧力フィールドに閾値とイベント発火条件）。エージェントへ渡す情報は数値データと場所説明にとどめ、行動は「move/stay + 自由記述のmemory/reasoning」と完全に自由化されている。`agent_observation.md` でも「命令ではなく観測」と明示。一方、`generate_habitat_conversations.py` の会話再生成プロンプトではtone許容集合や「ナッジ語彙を出してはならない」等の強い制約が課されており、ここでは創発の自由度がやや絞られる。コアエンジンは高水準だが、最終UIに反映される観測はポストプロセス側の制約を受けるため、フル満点には届かない。

#### 根拠
- `examples/spatial_demo/agent.py:371-400` — message プロンプトは数値占有率・近傍エージェント状況・記憶・受信メッセージのみを与え、JSONで `{message, reasoning}` を要求する設計（行動指示なし）。
- `examples/spatial_demo/agent.py:476-512` — action プロンプトも「move/stay + memory + reasoning」のみで戦略を示唆しない。
- `domain_packs/iss_benevolence/domain.yaml:131-156` — `auto_pressure_rules` が `resource_pressure>60` などの閾値で世界状態を遷移させる。直接行動を指示せず "緊張↑ 協力必要性↑" のような状況ラベルにとどめている。
- `domain_packs/iss_benevolence/prompts/agent_observation.md` — "各エージェントの内面と行動を、命令ではなく観測として記録する。"
- `scripts/agent_turn_runner.py:601-606` — neutral_v2 プロンプトに "全員を協力方向へ誘導しない / 不安・撤退・沈黙などの反応も正当な観測値" と明記。
- `scripts/generate_habitat_conversations.py:336-345, 444-456` — `run_condition_hint` ごとに語彙を制限し（"rule_only_no_nudge_objects の時は、ナッジ、善性オブジェクト、持ち寄り棚…という語を出さないでください"）、tone集合を会話タイプで絞る。これがコア自由度を一部相殺。
- `examples/spatial_demo/configs/config.iss.claude.run_b.yaml:52-84` — place description に "[Benevolence object: Memory Shelf] People may leave a small personal item and take one, sharing stories across cultures." 等の用途示唆が含まれ、軽度の行動ヒントとなりうる。

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
ISS閉鎖環境を題材にした原創性の高いシナリオで、20名の「世界が100人の村」型HCD設計ペルソナ、5条件（A/B/C/D/E）の対照実験、補助軸（修復イベントなし／喧嘩イベントなし）まで含む。仮説も「ナッジは仲直りを増やすのではなく、仲直りしなくても戻れる道を作る」という非自明な再フレーミングで、社会科学的・組織設計的価値が高い。災害避難所・病棟・寮・船舶への転用構造まで明示されており、現実の組織問題への接続が明瞭。

#### 根拠
- `README.md` — "GOOD ECHOは、善性を個人の性格だけに任せず、空間・物・ルール・会話の設計によって支えられるかを検証するシミュレーションプロジェクトです。"
- `README.md` — "ナッジは仲直りを作ったのではない。仲直りしなくても戻れる道を作った。"
- `docs/ISS/experiment_design_iss_20agents_100steps.md:38-94` — 20名のHCD設計ペルソナ（難民Amir、義足のAisha、識字率の低いTariq等）と人口分布チェック表。
- `docs/ISS/experiment_design_iss_20agents_100steps.md:289-307` — HCDレビュー解決状況テーブル（public/private二層ストレス、Crew Quarters追加、ファトワ原則）。
- `README.md` — "災害避難所、病棟、介護施設、寮、船舶、研究施設、高圧プロジェクトチームなどにも転用できます。"
- `domain_packs/iss_benevolence/data/objects_menu.tsv:2-12` — 10種類の善性オブジェクト（持ち寄り棚、聖域マーク、移動投票パネル等）の具体設計。

---

### C. 発展性 — 最終: 9.0/10

#### コード拡張性
`sim_core` がドメインパック汎用フレームワークとして実装され、`default_v1.yaml` から継承する仕組み、`${pack}` / `${root}` トークン置換、scenario YAMLによる差分上書き、validation report、hooks スロット（pre_run/pre_step/post_observation 等）まで揃っている。2つ目のドメインパック `agi_youth_japan` の存在が再利用性を実証している。`examples/spatial_demo` 側もconfig YAMLでplaces/personas/world_context/fire/economyを切り替え可能。

#### 将来展望
README に「差し替え可能な設計」セクションがあり、人物・空間・イベント・ナッジ・評価指標の各軸で具体的に何が差し替え可能かを列挙。災害避難所・病棟・寮など具体的な転用先を提示。さらに `conversation_threads.tsv` / `messages.jsonl` / `habitat_frames.jsonl` の共通スキーマでUIフレームを再利用できる点まで設計済み。学術寄りのcross-experiment paper draftも整備されており、研究価値の継続的拡張が見える。

#### 根拠
- `sim_core/domain_pack.py:57-67, 70-86, 89-94, 211-244` — deep_merge による継承、トークン置換、validation の汎用化。
- `sim_core/defaults/default_v1.yaml:27-35` — hooks スロット定義（pre_run/pre_step/...）。
- `domain_packs/agi_youth_japan/domain.yaml` — 2つ目のドメインパックが存在し、フレームワーク汎用性を実証。
- `domain_packs/iss_benevolence/domain.yaml:74-99` — 20以上のruntime profiles を登録（Claude/Codex/Cursor別、smoke/full別、20x100別）。
- `README.md` — "差し替えられるもの: 人物… 空間… イベント… ナッジ… 評価指標…"
- `README.md` — "[GOOD ECHO cross experiment paper draft](docs/ISS/reports/2026-05-06_good_echo_cross_experiment_paper_draft.md)" など研究文書群。

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
`LLMClientProtocol` で Ollama と CLI ベース（Claude/Codex/Cursor）を抽象化。CLI 用に `CommandLLMClient` がリトライ／タイムアウト／ANSI除去／stdout フィルタ／JSON フィールド抽出を担う。プロンプトは JSON 強制（`response_format`/`response_json_field`）で、ブレース対応の手書きJSON抽出器がフォールバックも兼ねる。並列度は `llm.parallelism` で制御。エラーハンドリングは exception 単位で堅牢。

#### メモリ設計
エージェント自身が「次ステップで覚えたいこと」をLLM出力で生成（`ActionDecision.memory`）し、`memory_limit`（FIFO上限）と `memory_size`（直近何件をプロンプトに含めるか）で制御。受信メッセージも同様にバッファ化。`memory_reasoning.jsonl` にバッチ出力されるため後解析可能。さらに `relationship_seed` / `baseline_stress` / `persona` の静的コンテキストもプロンプトに供給され、単なるログ羅列ではない構造化メモリ。

#### コード品質・シミュレーション正確性
4 フェーズの同期実行（message decisions → message送信 → action decisions → movement）でレース回避設計。`previous_fire_exposure` / `previous_place_occupancy` を step 開始時に固定し、step 内整合性を確保。型ヒント・TypedDict・Protocol を使った静的構造化。ファイル分割もきれい（agent / simulation / llm_backends / utils / visualization）。一方、合計約 4000 行と肥大化し、ISS実験では使わない economy/fire レイヤーが残っている。`random.seed` の明示が見当たらず、再現性は config に依存。

#### 根拠
- `examples/spatial_demo/llm_backends.py:152-273` — タイムアウト／リトライ／JSON 抽出／ANSI除去まで含む CLI クライアント。
- `examples/spatial_demo/llm_backends.py:22-41` — `LLMClientProtocol` による抽象化。
- `examples/spatial_demo/agent.py:515-550` — ブレース深度追跡で堅牢に JSON 抽出。
- `examples/spatial_demo/agent.py:676-692` — LLM 自身が生成した `memory` を FIFO で蓄積。
- `examples/spatial_demo/simulation.py:568-748` — 4 フェーズ同期実行（message decisions → 送信 → action decisions → movement）。
- `examples/spatial_demo/simulation.py:178-185` — `ThreadPoolExecutor` を `llm_parallelism` で制御。
- `examples/spatial_demo/simulation.py:386-406` — `memory_reasoning.jsonl` バッチ出力で I/O 効率化。
- `examples/spatial_demo/simulation.py:606-618` — step 開始時に previous_state を凍結してレース回避。
- `scripts/agent_turn_runner.py:625-720` — neutral_v2 プロンプトの構造化（観測項目を thought / private_talk / social_post の3層で分離）。

---

## 総評

### 優れている点
- ISS閉鎖環境×HCD設計ペルソナ×5条件比較という独創的かつ社会科学的価値の高いシナリオで、「ナッジは仲直りを増やすのではなく戻れる道を作る」という非自明な仮説提示を実験で支えている。
- `sim_core` がドメインパック汎用フレームワークとして完成度高く、2つ目のパック（agi_youth_japan）と20以上のプロファイル登録で再利用性が実証されている。
- LLM抽象化（Ollama / Claude CLI / Codex CLI / Cursor CLI）、リトライ・JSON抽出・並列実行・LLM自己生成メモリ・4フェーズ同期実行など、技術実装が成熟している。
- 詳細なレポート群（cross experiment paper draft、evidence dossier、integrated discussion）で実験結果と考察まで体系化されている。

### 改善点・提言
- `generate_habitat_conversations.py` での tone集合・語彙禁止リスト・conversation_type 制約が、コアエンジンが提供する自由度を再生成段階で絞り込んでいる。UI観測値そのものが半ば指示の影響を受けるため、ポストプロセスをオプション化するか「観測のみ」と「再生成」で記録を二系統に分けるとさらに堅牢になる。
- `random.seed` 等の明示的シード制御が見えにくく、CLI バックエンドの非決定性も加わって厳密な再現性確保には追加の仕組みが望ましい。
- コードベースが約 4000 行と肥大化し、economy / fire レイヤーは ISS 実験では未使用のままなので、ドメインパックごとに不要レイヤーを明示的に無効化する仕組みがあると保守性が高まる。

### 一言コメント
社会科学的洞察・実験設計・汎用フレームワーク・LLM統合のすべてに渡って完成度が高く、ハッカソン作品としては研究プロトタイプ水準。「戻り方」という再フレーミングが特に印象的で、閉鎖環境設計の議論に実質的な貢献を持ちうる。
