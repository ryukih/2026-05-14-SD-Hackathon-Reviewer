# 評価レポート: spring-park-simulation

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

原点中心の 2 次元フィールドに複数の「場所」（デフォルトでは左右にバーが 1 つずつ、独立した収容上限を持つ）を配置したマルチエージェント・シミュレーション。各エージェントは Ollama 経由の LLM で駆動され、毎ステップ Message / Memory / Action を生成して同期的に行動する。エージェントには占有率・火事の強度や距離などの生の数値データのみが与えられ、「危険」「快適」といった定性評価や行動指示は一切渡されない。各エージェントには male / female のペルソナがランダムに割り当てられ、通信半径内かつ同一領域（両者とも場所外、または同じ場所内）の相手とのみメッセージ交換ができる。シミュレーション途中で複数の火事イベントが発生し、知覚半径内のエージェントのみが火事情報を直接受け取り、半径外のエージェントは他者からのメッセージ経由でのみ間接的に知る設計となっている。テーマは、定量情報と局所通信のみから集団的な回避・情報伝達・滞在判断などの行動パターンがどのように創発するかを観察することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  3   |  3   |  -   |    3.0    |
| B. 世界設定  |  7   |  7   |  -   |    7.0    |
| C. 発展性    |  5   |  4   |  -   |    4.5    |
| D. 技術実装  |  6   |  6   |  -   |    6.0    |
| **合計**    |  21  |  20  |  -   |  **20.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 3.0/10

#### 評価
世界ルール側（占有率・通信領域制約・火事の知覚モデルB）は丁寧に設計されているが、行動の自由度が設計レベルで強烈に閉じられている。`config.yaml` のペルソナで「actionは "move" + "up" を選べ」「actionはstayのみ」と行動そのものを直接指定し、さらにコード側でも id=0 は強制 `action="stay"`、id=1 は人間語フィルター、id=2 は stay 用ペルソナを与え、LLM は実質的にテキスト整形器・詩生成器として動作する。`_purify_reasoning` での200行超の正規表現連鎖が出力分布を狭く縛り、本Configでは `fires: []` のため火事による緊張も発生しない。映像作品 TRINOIR への従属が強く、創発ポテンシャルは設計時点で意図的に最小化されている。

#### 根拠
- `config.yaml:42-46` — 犬ペルソナに「においがするかぎり、北（up）へ一歩あるけ。actionは "move" + "up" を選べ。」
- `config.yaml:64,68` — 観測者・いち子ペルソナに「actionはstayのみ」「reasoningは必ず ""」と明示
- `agent.py:892-904` — `parse_action_response` 内で id=0 は LLM 出力を無視して `{"action": "stay", "direction": None, ...}` を強制返却
- `agent.py:828-849` — `_filter_dog_message` で犬の人間語を抽出後カット
- `agent.py:275-327` — `_compute_sensory_data` が「ほとんど感じない」「強い」など定性ラベルを生成して渡す（READMEの「定量情報のみ」原則から逸脱）
- `simulation.py:681-704` — 火事の Model B 知覚は精密実装だが、`config.yaml:17` で `fires: []` のため未起動
- README.md — "エージェントには生の数値データ（占有率、火災の強度・距離等）のみを提供し、行動指示や定性的評価（「危険」「快適」等）は一切与えない" と謳うが、実コード/Configはこれと矛盾

---

### B. 世界設定 — 最終: 7.0/10

#### 評価
「春の公園、最古の桜、散る花びらの境界の時間」という設定に、不動の現象としての「いち子」・本能で北へ向かう「犬」・ベンチで観測する「観測者」のトライアドを配する構成は、標準的なクラウド/避難デモから完全に外れており、芸術的・詩的なオリジナリティが極めて高い。Yugen・Wabi-sabi 美学の組込み、TRINOIR Visual Compilation による30ステップの映像プロンプト化（起・承・転・結）への接続も野心的。一方、ビジネス/社会科学的な分析価値や定量的なドメイン課題への射程は薄く、研究シミュレーションというより「LLM参加型映像作品」としての色が強い。

#### 根拠
- `config.yaml:8-16` — 「ここは春の公園。一番古い桜の木がある。桜が散るか散らないかの、美しい境界の時間が流れている。」
- `config.yaml:30-68` — 三者ペルソナ（いち子＝現象、犬＝鼻、観測者＝詩を打つ存在）
- `simulation.py:462-566` — `_run_image_prompt_generation` が起承転結4枚の英文シネマプロンプトを生成（Yugen / Wabi-sabi 指定）
- `generate_video_prompts.py:26-173` — Wong Kar-wai 風スタイル、7シーン定義（"The Ghost Stirs"、"Language Dissolves"等）
- README.md — "デモ動画：火災近傍では、単純な退避にとどまらず、合流指示・物資確保・他者待機・監視などの創発的な判断が観察された。"

---

### C. 発展性 — 最終: 4.5/10

#### コード拡張性
ファイル分割（agent.py / simulation.py / ollama_client.py / visualization.py / utils.py）と YAML での places / fires / personas / late_spawns の外部化は良好で、場所や火事の複数対応も実装済み。一方、`agent.py` の `_purify_reasoning`（agent.py:531-697）と `_clean_memory`（agent.py:699-748）に Phase 15〜Phase 44 までの ad-hoc 正規表現が200行近く積層しており、新エージェント追加や別ペルソナ導入には `if self.id == 0/1/2` 分岐の修正が至るところで必要になる。`_compute_sensory_data`（agent.py:275-327）も id ベタ書き分岐で、構造的拡張性は低い。

#### 将来展望
README には places を bar/cafe/library に拡張する具体例があり最低限の方向性は示されている。しかし、Phase 44 までの試行錯誤が映像生成（TRINOIR）方向に集約され、シミュレーションそのものを社会科学・組織研究的に発展させる議論はほぼ示されない。テストや CI、シード管理など、研究的再現性を支える基盤に関する将来計画も無い。

#### 根拠
- `agent.py:531-697` — `_purify_reasoning` 内のPhase 15〜Phase 44 の正規表現連鎖（200行超、エージェントIDべた書き分岐含む）
- `agent.py:300-327` — `_compute_sensory_data` 内で `if self.id == 1` による犬専用の温もり/重さ計算
- `simulation.py:321-329` — `initialize_agents` 内で id=0 を sakura_tree 中心固定、id=1 を (0, -4) 固定とハードコード
- `config.yaml:69-81` — places は YAML 化済み、種別 (`type`) も拡張可能
- README.md — "将来の拡張例（カフェや図書館など）" の YAML 例は提示
- README.md — 研究的・ビジネス的な発展方向（情報拡散モデル、組織モデルへの応用等）に関する記述は見当たらない

---

### D. 技術実装 — 最終: 6.0/10

#### LLM利用
Ollama 互換クライアント（`ollama_client.py`）に温度・min_p・repeat_penalty・think フラグなど主要パラメータを集約し、観測者専用クライアント（`simulation.py:111-126`）まで切り分けている点は丁寧。JSON 抽出は brace-matching と文字列エスケープ追跡を行う実装（agent.py:750-787）で、qwen3 系の `<think>` タグ除去や fallback ラベル抽出、画像プロンプト品質チェックの最大3回リトライ（simulation.py:568-607）も備わる。一方、prompt 内に行動指示が混入する設計はプロンプトエンジニアリングとしては逆行している。

#### メモリ設計
`memory_limit` と `memory_size`、`message_history_limit` と `message_context_size` を分離し、保存と参照を別パラメータで制御している点は妥当（agent.py:71-74）。`Step N: ...` 形式でタイムスタンプ付与し、Kintsugi フィルター（simulation.py:148-198）で出力ログ自体も正規化している。ただし犬は memory を空欄固定（agent.py:706-707）、いち子は forced_gesture 注入（agent.py:902-903）など、メモリが「LLMの内的状態の蓄積」というより「演出用テキスト整形」に寄っており、創発研究としての memory 設計とは性質が異なる。

#### コード品質・シミュレーション正確性
4フェーズ同期実行（メッセージ決定→送信→行動決定→移動）が明確で、移動前の位置関係でメッセージを送る順序保証は適切（simulation.py:706-843）。late_spawn、複数 fires の active_names ベースの再活性化防止も実装。一方で agent.py は1060行・simulation.py は950行を超え、`_purify_reasoning` には Phase 15〜44 のコメント付き正規表現が雑居しテスト・型チェックは無い。`random.seed` 等の再現性指定もなく、初期位置は毎回ランダム。

#### 根拠
- `ollama_client.py:47-94` — `generate` の例外処理と timeout / num_predict / repeat_penalty / min_p 指定
- `simulation.py:111-126` — observer_llm_client の分離初期化
- `agent.py:750-787` — `_extract_json_from_text` の brace-matching 実装
- `simulation.py:568-607` — `_validate_image_prompts` + 3回リトライ + 違反フィードバック注入
- `agent.py:71-74` — memory / message の保存/参照サイズ分離
- `simulation.py:230-256` — `_log_memory_reasoning_batch` で JSONL バッチ出力
- `simulation.py:706-848` — 4-phase 同期ループ（メッセージ→送信→行動→移動）
- `simulation.py:258-300` — `_generate_initial_positions` がランダム位置を生成するが `random.seed` 指定が無く再現性が無い

---

## 総評

### 優れている点
- 「桜・いち子・犬・観測者」という極めて独創的かつ詩的な世界観をマルチエージェント基盤に乗せ、TRINOIR Visual Compilation（4枚の英文シネマプロンプト自動生成）まで一本のパイプラインに繋いだ統合度の高さ
- 占有率の場所別管理、通信の「同一領域」制約、火事の Model B 距離依存知覚など、世界ルール側のメカニクスは精密に設計され、複数場所/複数火事/late_spawn 等の構造的拡張も実装済み
- Ollama 連携で base_url / model / temperature / repeat_penalty / min_p / think を Config 化し、観測者用 LLM の分離、画像プロンプト品質チェック+リトライなど、LLM 実装の堅牢性は平均以上

### 改善点・提言
- 創発研究としての評価軸では、ペルソナとコード（id ベタ書き分岐 + 200行の `_purify_reasoning`）でエージェント行動を強くスクリプト化している点が決定的に弱い。世界ルールの精度を活かすには、行動指示文を排し、エージェントは raw データのみから自律的に判断する形へ戻す方が望ましい
- Phase 15〜44 のフィルター肥大化を、ペルソナ別 Strategy クラスやデータ駆動の禁止語リストに分解すれば、新エージェント・新シーン追加コストが劇的に下がる
- `random.seed` を Config 化し、観測ログ（messages.jsonl、memory_reasoning.jsonl）から創発指標（合流、退避、情報拡散）を抽出する分析スクリプトを添えると、研究シミュレーションとしての価値が大きく増す

### 一言コメント
「LLMマルチエージェントを使った映像詩」としては独創的で完成度が高い一方、ハッカソンが想定する「LLMの自律性から創発を観察する」基準では、世界設計の精度がペルソナとフィルター層の行動制御で大きく相殺されており、創発設計のスコアが伸びにくい構成になっている。
