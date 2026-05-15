# 評価レポート: good_echo_iss_sim_cursor_100s_w_accident

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

ISS（国際宇宙ステーション）を模した閉鎖空間を舞台に、複数のクルー（既定では 10 名、1 ステップ = 1 日）がどのようにストレスを受け、会話し、協力し、衝突し、修復するかを観測する LLM マルチエージェント・シミュレーション。本サブミッションは 100 ステップ・事故イベントありの構成で、Day 50 に宇宙デブリ衝突による HAB 損傷、Day 51-55 の地上指示による応急修復と HAB 封鎖、Day 56 からの HAB 暫定再開、Day 60 前後の HAB 酸素漏れ判明、Day 65 以降の LAB 酸素低下による生命維持危機までの外乱が `events_run_*.tsv` に組み込まれている。Run A（ナッジなし対照群）と Run B（共同食記録・静穏時間の合図・個室公平利用・感謝可視化など善性ナッジオブジェクトあり介入群）の A/B 比較を通じて、衝突後の修復会話の発生、会話の偏り解消、孤立の減少、高ストレス時のチーム維持などを測定する。`sim_core/hooks.py` の 9 段階フック構造、`domain.yaml` の 12 プロファイル（claude × codex × cursor × smoke/full × A/B）、独自 JSON ブレース深度パーサ、ANSI クリーンアップ付き CLI バックエンドなど、再現性と移植性に配慮した構成になっている。テーマは「善性を育む環境設計」の検証で、ISS 以外の災害避難所・病棟・介護施設・寮・船舶など、他の閉鎖環境への転用可能なドメインパック化フレームを志向する。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  8   |  -   |    8.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  9   |  9   |  -   |    9.0    |
| D. 技術実装  |  8   |  7   |  -   |    7.5    |
| **合計**    |  34  |  33  |  -   |  **33.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 8.0/10

#### 評価
世界ルールの設計精度・行動の自由度の両軸が高い水準でバランスしている。場所（capacity/half_size/center）、通信半径、in-place isolation（中にいる者と外の者は通信不可）といった物理的制約が明確に定義され、Day50の宇宙ゴミ衝突から Day65以降のLAB酸素低下までイベントの強度と影響対象が `events_run_*.tsv` で精密に外部化されている。エージェントには占有率・容量・距離・近隣エージェントの位置・所属モジュールなど **生の数値と事実のみ** が提示され、`"あなたは助けるべき" "警告せよ"` のような命令的指示は含まれない。Run B のナッジオブジェクトも `[Benevolence object: Talk-OK seat]` のように **場所説明としての存在** が示されるだけで、それを使えという指示にはなっていない。命令ではなく観測条件として渡すという `agent_observation.md` の方針が一貫している。減点要因は、place description に挙動を示唆する英語表現（"reduce isolation" "encourages moving" など）が場面によって入っており、完全な raw data 提示ではない点。

#### 根拠
- `examples/spatial_demo/agent.py:333-369` — 行動プロンプトは Position / In place / occupancy_rate / capacity / 近隣エージェント情報＋"stay/move(up/down/left/right)"の選択肢のみで構成され、行動方針の指示はない
- `examples/spatial_demo/agent.py:107-128` — `get_nearby_agents` で同一場所内 or 共に外、かつ通信半径以内のみ会話可能という物理ルールがコードに埋め込まれている
- `examples/spatial_demo/configs/config.iss.cursor.run_b.yaml` — place description に `[Benevolence object: Memory Shelf] People may leave a small personal item...` のようなナッジ説明が含まれるが使用指示はしない
- `domain_packs/iss_benevolence/data/events_run_a.tsv` — `DEBR01..DEBR05` で Day50衝突〜Day65酸素低下までを構造化したイベント系列
- `domain_packs/iss_benevolence/prompts/agent_observation.md` — "命令ではなく観測として記録する"

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
ISS閉鎖空間 × 一般市民10名 × 100日 × 中盤の宇宙デブリ事故〜末期の酸素低下生命維持危機、という極めてオリジナルかつ社会科学的に価値の高いシナリオ。10名のペルソナ（シリア難民Amir、ケニア義足パラアスリートAisha、コロンビア人アーティストSofia、フランス元外交官Henri等）は文化・年齢・宗教・経済層・心理的脆弱性が多様に分布し、`baseline_stress`, `self_efficacy`, `institutional_trust`, `social_anchor`, `vulnerability_note` まで構造化されている。Run A/B 比較設計（ナッジオブジェクト有無）と100日にわたる関係崩壊・修復・危機協働を同時に観察できる構造はビジネス・研究の両面で意味を持ち、README が避難所・病棟・寮など他閉鎖空間への一般化も明示している。

#### 根拠
- `README.md` — "高ストレス環境を強制構築して、心理と関係の崩壊を環境設計でどう防げるかを試すプロダクト"
- `domain_packs/iss_benevolence/data/agents.tsv` — 10名の構造化ペルソナ（age, region, religion, baseline_stress, vulnerability_note 等）
- `domain_packs/iss_benevolence/data/events_run_b.tsv` — CONF/REPB/OBJ/DEBR が連鎖する100日タイムライン
- `README.md` — 「災害避難所、病棟、介護施設、寮、船舶、研究施設、高圧プロジェクトチームなどにも転用できる」
- `docs/ISS/personas_iss_10agents.md`, `docs/ISS/places_iss_design.md`, `docs/ISS/iss_objects_menu.md` — 設計ドキュメントが充実

---

### C. 発展性 — 最終: 9.0/10

#### コード拡張性
`sim_core/`（domain pack 解決・バリデーション・hooks）と `examples/spatial_demo/`（実行エンジン）と `domain_packs/iss_benevolence/`（データ・プロンプト・シナリオ）が明確に分離されたモジュール構成。YAML スキーマ（`schema_version: domain_pack_v0.1`）, データは TSV、`column_aliases` で列名差し替え可能。LLM プロバイダーは Ollama / Command (Claude/Codex/Cursor) 切り替え可能。 `runtime.profiles` で 12 種類のプロファイル登録済み。`hooks.py` で `pre_run/pre_step/pre_observation/.../export_viewer/audit` の拡張点を定義し、プラグイン拡張の足場を用意している。

#### 将来展望
README で「終盤イベントだけを差し替える」「新ドメインへの差し替え」を具体例（events_run_*.tsv の編集、`module_id`→`place_id`/`zone_id` 化、災害避難所/病棟への展開）まで提示しており、新規ドメインのテンプレ手順が明確。`source_docs` に設計ドキュメントを参照させる仕組みもある。減点要素は、Cursor CLI 特化のシェルスクリプトが残っており、将来の他バックエンドでは再実装の余地がある点。

#### 根拠
- `sim_core/domain_pack.py:1-308` — domain pack 解決と検証
- `sim_core/hooks.py:9-17` — 9 段階のフック拡張ポイント
- `domain_packs/iss_benevolence/domain.yaml` — 12 ランタイムプロファイル登録
- `README.md` — "差し替えられるもの" 章、"新しいドメインへの差し替え" 章、"終盤イベントだけを差し替える" 章
- `domain_packs/iss_benevolence/README.md` — 「Run A/Bを同じ枠組みで比較できる」「実行時に必要なデータは iss_benevolence/data/ 内で完結」

---

### D. 技術実装 — 最終: 7.5/10

#### LLM利用
ステップごとに2フェーズの LLM 呼び出し（位置情報なしでメッセージ決定 → メッセージ送信後に位置情報込みで行動決定）という洗練された設計。JSON 抽出は素朴な split ではなく、文字列内エスケープ考慮の brace-matching パーサで実装されている。CLI バックエンドはタイムアウト、リトライ、ANSI escape 除去、stdout フィルタ、複数プロセス排他ロック（`mkdir` atomic）を備える。`ThreadPoolExecutor` による並列化（`parallelism: 2`）にも対応。一方、CLI バックエンドでは temperature/max_tokens が無視される（`del temperature; del max_tokens`）、prompt は英語固定で `world_context` を埋め込む構造のため日本語パックとの整合は world_context 経由依存。

#### メモリ設計
LLM が `memory` フィールドに自分で記憶を生成 → FIFO（`memory_limit=15`, prompt 投入は `memory_size=4` 直近のみ）。`received_messages` も `message_history_limit=8` / `message_context_size=3` で別管理。`memory_reasoning.jsonl` にバッチ書き出し。単なるログダンプではなく LLM 自身が "what to remember" を選ぶ点でセマンティックメモリに寄っている。長期記憶/要約/想起の階層は無いがハッカソンレベルでは十分。

#### コード品質・シミュレーション正確性
ステップは 1) 全エージェントのメッセージ決定 → 2) メッセージ配信 → 3) 全エージェントの行動決定 → 4) 移動実行 → 5) 状態更新、と同期的に分離されており、移動前の位置で会話判定が確定するためレースは無い。`TypedDict` を活用した型注釈、`dataclass(slots=True)`、責務ごとのファイル分離（agent / simulation / llm_backends / visualization）、`logger` 利用が一貫している。減点：`agent_turn_runner.py` が 1247 行と肥大化、`simulation.py` も 955 行ある。`random.choice` 等で `seed` 設定が無く再現性は確保されていない（fires のランダム位置、初期位置のランダム生成）。

#### 根拠
- `examples/spatial_demo/simulation.py:626-680` — 4 フェーズ同期実行コメント
- `examples/spatial_demo/agent.py:540-577` — `_extract_json_from_text` の brace-depth マッチング
- `examples/spatial_demo/llm_backends.py:140-243` — `CommandLLMClient.generate` のリトライ・タイムアウト・stderr 取り扱い
- `scripts/run_cursor_prompt.sh` — atomic mkdir ロック、EPROTO リトライ、Pythonバックオフ
- `examples/spatial_demo/agent.py:683-715` — LLM の `memory` 出力を `Step N: ...` 形式で FIFO に追加、`memory_limit` 超過で pop
- `examples/spatial_demo/simulation.py:480-499` — 初期位置生成に `random.randint` を直接使用、seed 設定なし

---

## 総評

### 優れている点
- ISS閉鎖空間 × 多文化10名 × 100日 × 中盤の宇宙デブリ衝突という極めてオリジナル性が高く、社会科学的にも現実応用にも結びつく題材
- 世界ルール（capacity / communication radius / place isolation / event timeline）と行動自由度（生データのみ提示、JSON 行動選択は完全自由）のバランスが取れた創発設計
- domain pack（YAML + TSV + prompts）と実行エンジンの分離、`sim_core/hooks.py` による9段階の拡張点、12種のランタイムプロファイル、Run A/B 差し替えがすべて構造化されており、他閉鎖空間ドメインへの展開可能性が極めて高い
- 二フェーズ LLM 呼び出し、brace-matching JSON parser、CLI バックエンドの排他ロック・リトライ、JSONL/TSV による下流 UI 連携など、エンジニアリングの完成度が高い

### 改善点・提言
- ランダム要素（初期位置・fires位置・job サンプリング）に seed 設定が無く、A/B 比較の再現性確保のためには `random.seed`/`np.random.seed` を config で受ける改修が望ましい
- `agent_observation.md` が 3 行と簡素で、`agent_turn_runner` 用の長い JA プロンプトとの役割分担が不明瞭。spatial_demo 側プロンプト（英語、agent.py 内）と pack 側プロンプト（日本語、agent_turn_runner.py 内）を統一するか、用途を明示するとよい
- place description に "encourages moving" "reduce isolation" のような効果示唆英文が含まれる箇所があり、より純粋な raw data 提示に整理すれば創発スコアがさらに上がる
- CLI バックエンドで temperature/max_tokens が `del` されているため、Cursor/Codex 経由でも温度等を CLI 引数として渡せると LLM の振る舞いをコントロールしやすい

### 一言コメント
ISS閉鎖空間ストレスを題材に、世界ルールの精密さとエージェントの行動自由度を高い水準で両立した完成度の高いマルチエージェント・シミュレーション。domain pack 構造と Run A/B 比較設計により、ナッジオブジェクトの効果検証フレームワークとして他ドメインへの展開可能性が非常に大きい。
