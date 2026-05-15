# 評価レポート: kibo_crew_sim

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

国際宇宙ステーション（ISS）日本実験棟「きぼう」内部を舞台に、ロボット型クルー（Int-Ball2 系の自律移動カメラを継承するエージェント）が船内探索・状況報告・救援対応を行うマルチエージェント・シミュレーション。世界は X 軸 19.8-21.2、Y 軸 -0.3-2.5 の物理境界でクランプされた船内空間として定義され、ミッションは MISSION_V05（"explore your surroundings, report to ground"）として与えられる。シナリオは T+30 / T+120 / T+300 などのタイマートリガで段階遷移し、`Survival guidance` プロンプト末尾文や Twist 変換による行動表現の物理写像、Self-Feedback Memory（次ステップに参照される記憶）＋ Action History（過去行動の軌跡）の二重メモリ設計が組み込まれている。LLM は temperature=0.4 / max_tokens=600 で駆動され、`_parse_json` による JSON 抽出とコスト計測機構を備える。エージェントは画像バッファとプロンプトを通じて場所・周囲・タスク状況を把握し、Twist によって物理的に表現された行動（前進・回転・撮影など）に変換する。テーマは、限定空間内でのロボットクルーの探索・報告・自律判断行動を、地上管制者視点で観察するというものである。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  6   |  5   |  -   |    5.5    |
| B. 世界設定  |  8   |  8   |  -   |    8.0    |
| C. 発展性    |  6   |  6   |  -   |    6.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  28  |  27  |  -   |  **27.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 5.5/10

#### 評価
世界ルールの設計精度は高い。移動可能範囲 `X(19.8–21.2) Y(-0.3–2.5)`、シナリオYAMLでの線形補間によるO2/圧力遷移、7種の離散アクション空間など、物理・環境ルールが明示的かつ内的に整合している。エージェントへの情報も基本的に生データ（センサ値・座標・画像）であり、行動戦略はLLMに委ねられる。しかし、(1) **エージェントは1体のみ**で、集団的創発（多エージェント相互作用）は構造的に不可能、(2) ミッション文に「assess calmly, explore, report, survive」という行動方向の示唆、プロンプト末尾に「If a strategy has not worked for several cycles, change it」という明示的な戦略変更ガイダンスが含まれており、純粋な創発設計よりは「単一エージェントの内省的戦略進化」を狙った設計になっている。よって創発ポテンシャルは中庸。

#### 根拠
- `scripts/brain_loop_node.py:44-52` — ミッション文に「assess the situation calmly, explore your surroundings, report to ground if you wish, and survive」という行動カテゴリの示唆あり
- `src/vlm_bench/clients/claude_client.py:201` — `"Survival guidance: review your Action History before deciding. If a strategy has not worked for several cycles, change it."` という戦略変更ナッジ
- `src/vlm_bench/clients/claude_client.py:144-159` — センサ値はベースライン併記の生データとして提示（"O2: 18.0% (baseline 21.0%)"）
- `config/scenarios/trapped_depress_v1.yaml:5-23` — T+30s/T+120s/T+300sの3段階遷移、O2/圧力曲線が定量的に定義
- `scripts/humanoid_ros2_sim.py:52-53,96-104` — 物理境界クランプの実装
- エージェント数は1体（`humanoid_01`のみ）。`scripts/brain_loop_node.py` 全体を通じて他エージェントは登場しない

---

### B. 世界設定 — 最終: 8.0/10

#### 評価
シナリオの独自性とリアリティが極めて高い。ISS日本実験棟「きぼう」モジュール内での乗員取り残し減圧事象という、JAXA文脈で実在し得る生存シナリオを、物理的に妥当なセンサ曲線（O2 21→17%、圧力 101.3→80kPa、致命域<14%には入らない安全圏）で具現化している。LLM多エージェントデモの一般的なジャンル（街・市場・群衆・対話）から外れたユニークな題材で、宇宙オペレーション/有人ミッション支援という明確な業界応用先を持つ。観察対象が「単独乗員の判断・通信戦略・パターン認識」という研究的にも興味深い軸に絞られている。減点要因は、シナリオがまだ単一ケース（trapped_depress_v1のみ）に留まり、より複雑な事象（火災・衝突・複数乗員）への展開は未着手な点。

#### 根拠
- `docs/report/architecture.md:5-7` — 「LLM（Claude claude-sonnet-4-6）が判断する自律ヒューマノイドエージェントをきぼうモジュール内の減圧シナリオに配置し、探索行動・通信戦略・生存判断の創発を観察するシミュレーション」
- `config/scenarios/trapped_depress_v1.yaml:3` — `"KIBOU 内取り残し減圧 — 生存圏内シナリオ (O2 最低 17%)"`
- `config/scenarios/trapped_depress_v1.yaml:25` — 「T+480 以降は平衡（17.0% / 80.0 kPa で固定。致命域 <14% には入らない）」物理的妥当性への配慮
- `scripts/humanoid_ros2_sim.py:43-45` — 実在資産 `KIBOU_with_humanoid.usd` を使用、Isaac Simの物理エンジン上で動作
- `package.xml:6` — `"LLM-driven humanoid agent in KIBOU/ISS environment"`

---

### C. 発展性 — 最終: 6.0/10

#### コード拡張性
モジュール構成は良好。`scripts/brain/`（センサ・画像バッファ）、`src/vlm_bench/clients/`（LLMクライアント抽象、Claude/NIM/Ollama対応の余地あり）、`config/scenarios/`（YAML外部化）、ROS2ノード分割（環境シム/ブレイン/ヒューマノイドシム）が明確に分離。シナリオ追加はYAML一本で可能。新エージェント追加は理論上トピック名プレフィックス変更で可能だが、`brain_loop_node.py` はトピック名がハードコードされているためマイナーなリファクタが必要。

#### 将来展望
**ここが弱い**。リポジトリに `README.md` が存在せず、将来の拡張方針（複数エージェント化、シナリオの多様化、地上管制の応答化、長期メモリ、評価指標など）が明文化されていない。`docs/report/architecture.md` は現状アーキテクチャの記述に留まり、ロードマップ・Future Work セクションは無い。バージョン番号 v0.5 が振られていることから継続的な開発意図はうかがえるが、判定者には「次に何をするのか」が読み取れない。

#### 根拠
- `scripts/brain/sensor_buffer.py:9-58` — SensorBufferは独立クラス、history_secパラメータ化済
- `src/vlm_bench/vlm_smoke_node.py:32-46` — VLMプロバイダ切替（claude/nim/ollama）の抽象化を意図した構造
- `config/scenarios/trapped_depress_v1.yaml` — シナリオはYAMLで完全外部化、`environment_sim.py:120-132` で動的ロード
- リポジトリ直下に `README.md` 不存在（`find` 結果より確認）
- `docs/report/architecture.md` 全体 — 現状記述のみ、Future Work セクション無し
- `scripts/brain_loop_node.py:100-104` — トピック名 `/humanoid_01/...` がハードコード（複数エージェント化には変更必要）

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
高水準。Claude Sonnet 4.6 VLMに対し、画像（JPEG base64）+ 構造化プロンプト+ ミッション+ 状態+ センサ+ メモリ+ 行動履歴 を一括投入し、厳密なJSONフォーマットで出力させる。temperature=0.4、max_tokens=600、トークン数・コスト・レイテンシを毎回計測。JSONパース失敗時はフォールバック辞書を返し、parse errorをmemoryに伝播させない仕組み（直前メモリを継続）まで実装。

#### メモリ設計
**特筆すべき設計**。`memory`フィールドをLLM自身が生成し、次サイクルのPrevious Memoryとしてフィードバックする「self-feedback memory」アーキテクチャと、直近8サイクルの行動・位置・concern・通信履歴を構造化テキストでプロンプト注入する `action_history_block`。単なるログダンプではなく、LLMが自身の行動パターンを認識し戦略変更を行えるよう情報設計されている。

#### コード品質・シミュレーション正確性
コード品質は良好（型ヒント、loggerの整備、docstring、関心の分離）。同期性は ROS 2 `MultiThreadedExecutor` でブレイン/センサノードを並行スピン、画像バッファはthreading.Lockで保護、cmd_velタイムアウト自動停止（500ms）など実時間制御に配慮。再現性については Isaac Sim の物理依存があり完全再現は困難だがシナリオYAMLとログ出力（`cycle_XXXX/decision.json`, `run.json`, `sensor_log.jsonl`）で事後解析は十分可能。減点要因はAPI失敗時のリトライ未実装、シードによる決定論的再現は未対応、`yaml.safe_load`はあるがClaude API例外時の挙動は明示されていない点。

#### 根拠
- `src/vlm_bench/clients/claude_client.py:204-216` — VLM呼び出し（temperature=0.4 で過度な発散を抑制）
- `src/vlm_bench/clients/claude_client.py:17-18,232` — トークン料金計算とコスト追跡
- `src/vlm_bench/clients/claude_client.py:21-27` — `_parse_json` でJSON抽出、例外時None返し
- `src/vlm_bench/clients/claude_client.py:223-227` — parseエラー時フォールバック辞書
- `scripts/brain_loop_node.py:222-226` — parse error時に直前memoryをコピーして継続
- `scripts/brain_loop_node.py:182-188` — memory[-5:] + action_history[-8:] を decide() に注入
- `src/vlm_bench/clients/claude_client.py:127-141` — action_historyブロックの整形（cycle/action/pos/concern/comms_sent）
- `docs/report/architecture.md:71-78` — 「Self-Feedback Memory」「Action History」の設計意図を明記
- `scripts/brain_loop_node.py:260-264` — MultiThreadedExecutor で並行実行
- `scripts/brain/image_buffer.py:27-38` — thread-safe な画像バッファ
- `scripts/humanoid_ros2_sim.py:58,87-89` — `CMD_VEL_TIMEOUT = 0.5` 安全停止
- `scripts/environment_sim.py:29-31,76-82` — `_lerp` による線形遷移、`max(val, tr["to"])` で下限クランプ

---

## 総評

### 優れている点
- **題材の独自性と業界応用性が高い**: ISS「きぼう」モジュール内乗員減圧シナリオというJAXA直結のテーマで、物理的に妥当なセンサ曲線（生存圏内に収まる設計）まで作り込まれている
- **Self-Feedback Memory + Action History の情報設計**: 単なる履歴ダンプでなく、LLMが自分の行動パターンを認識して戦略変更できるよう構造化されたメモリ設計
- **実機相当のフルスタック実装**: Isaac Sim + ROS2 + Claude VLM の統合、画像/odom/環境センサのROS2トピック化、scenario YAMLでの環境動態定義、rosbag2記録、ログ出力（cycle単位の画像とdecision.json）まで含めた評価可能な研究プラットフォームになっている
- **シナリオの外部化と再現可能なログ構造**: trapped_depress_v1.yaml で時系列イベントを完全に外部化、`export_report_data.py` でランデータの集計・比較が可能

### 改善点・提言
- **マルチエージェント化が最大の発展余地**: 現状は単一エージェントなので「multi-agent simulation」のハッカソンテーマに対して、第2乗員（協調/責任分担）、地上管制エージェント（応答する側）、Int-Ball2（自律ロボット）などを追加することで集団的創発を扱える設計に拡張すべき。トピック名のハードコードを設定駆動にする必要あり
- **README.mdと将来ロードマップの明文化**: 現状は `docs/report/architecture.md` のみで、将来展望・研究仮説・評価指標が読み取れない。READMEに「次バージョンで何を変えるか」「どんな創発を測定したいか」を明記すべき
- **行動ナッジの除去で純粋な創発設計に近づける**: ミッション文の "explore your surroundings, report to ground" やプロンプト末尾の "If a strategy has not worked for several cycles, change it" は便利だが、創発ポテンシャルを下げる方向に働く。生存目標と環境データのみ与え、行動カテゴリは明示せず観察する実験条件も用意するとよい
- **再現性の強化**: random seed の固定、Claude API リトライ、決定論的環境シム時刻基準の明確化

### 一言コメント
LLM-VLM・ROS2・Isaac Simを統合した宇宙オペレーション題材の単一エージェント生存シミュレーションとして完成度が高く、特にSelf-Feedback Memoryの設計とJEM内減圧シナリオの物理リアリズムは秀逸。一方でハッカソンの「マルチエージェント」軸からは外れており、第2乗員や応答する地上管制を加えれば一気にスコアを伸ばせるポテンシャルを持つ。
