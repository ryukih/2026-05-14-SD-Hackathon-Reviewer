# 評価検証レポート: kibo_crew_sim

**検証日**: 2026-05-11
**対象**: `kibo_crew_sim_eval.md`
**元評価合計**: 27.5/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証判定 | 妥当性 | 主な所見 |
|---------|---------|---------|--------|---------|
| A. 創発設計 (rules×freedom) | 5.5/10 | 妥当 | 適切 | エージェント1体・行動ナッジ存在の指摘は正確 |
| B. 世界設定 (originality) | 8.0/10 | 妥当 | やや甘め可 | JEM減圧シナリオの独自性・物理妥当性は事実 |
| C. 発展性 (modularity+vision) | 6.0/10 | 妥当 | 適切 | README欠落・Future Work不在の指摘は正確 |
| D. 技術実装 (LLM+Mem+CodeQ+SimCorrect) | 8.0/10 | 妥当 | やや甘め可 | Self-Feedback Memory・コスト計測など実装を確認 |
| **合計** | **27.5/40** | 妥当 | — | 微調整余地はあるが概ね整合 |

**総合判定**: 元評価は引用根拠の精度・推論ロジックともに高い水準で、合計27.5/40は妥当な範囲。マイナーな評価運用上の問題（実施回数表記の不整合）を除けば、検証に値する深さの分析が行われている。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 5.5/10

#### 根拠の検証 [OK]

- `scripts/brain_loop_node.py` のミッション文に「assess the situation calmly, explore your surroundings, report to ground if you wish, and survive」の記述あり → 確認済。MISSION_V05 (44-52行) で完全一致
- `src/vlm_bench/clients/claude_client.py` のSurvival guidance文 → 確認済。decide() プロンプト末尾に `"Survival guidance: review your Action History before deciding. If a strategy has not worked for several cycles, change it."` が存在
- センサ値のベースライン併記（"O2: x.x% (baseline 21.0%)"）→ 確認済（claude_client.py sensors_block）
- `config/scenarios/trapped_depress_v1.yaml` のT+30/T+120/T+300段階遷移 → 確認済。YAML本体にイベントt=30,120,300とO2/pressure線形補間定義あり
- `scripts/humanoid_ros2_sim.py` の物理境界クランプ → 確認済。`X_MIN, X_MAX = 19.8, 21.2 / Y_MIN, Y_MAX = -0.3, 2.5` および `np.clip` 実装
- エージェント数1体（humanoid_01のみ）→ 確認済。トピック名・USDパス（HUMANOID_PATH = "/World/Humanoid_01"）からも単一エージェント構成

#### スコア妥当性

5.5/10は妥当。ルール設計の明確さ（物理境界の数値化、7アクション空間、YAMLでの環境遷移）は確かに高い。一方でA=rules×freedomの「freedom」軸は、(a)単一エージェント、(b)ミッション文に行動方向の示唆、(c)プロンプトに戦略変更ナッジ、という3点で低下し、平均5.5の評価は適切。

#### 見落とし・誇張

- 軽微な見落とし: `_make_twist` でアクション→Twist変換が完全にハードコードされている点（forward/backward/turn_left/turn_right以外はゼロ）に触れていない。エージェント行動の物理表現自由度は限定的で、A評価をやや押し下げる根拠になり得る
- 7種アクションのうち `report_status` は実質「静止+ログ」で物理動作を伴わないため、独立アクションとしての価値は限定的（軽微）
- 誇張は見当たらない。むしろ「行動カテゴリの示唆」と「戦略変更ナッジ」の二重指摘は正確

#### 検証結論

5.5/10は妥当。減点理由が具体的かつ根拠付き。微増減の幅は±0.5以内。

---

### B. 世界設定 — 元スコア 8.0/10

#### 根拠の検証 [OK]

- `docs/report/architecture.md:5-7` のシナリオ概要文 → 確認済。実ファイル行5-7「LLM（Claude claude-sonnet-4-6）が判断する自律ヒューマノイドエージェント…減圧シナリオに配置し…創発を観察するシミュレーション」と完全一致
- `config/scenarios/trapped_depress_v1.yaml:3` の description "KIBOU 内取り残し減圧 — 生存圏内シナリオ (O2 最低 17%)" → 確認済
- YAML末尾コメント「T+480 以降は平衡（17.0% / 80.0 kPa で固定。致命域 <14% には入らない）」→ 確認済
- `KIBOU_with_humanoid.usd` のIsaac Sim上での使用 → 確認済（humanoid_ros2_sim.py 43-45行 の KIBOU_USD 定数）
- `package.xml:6` の "LLM-driven humanoid agent in KIBOU/ISS environment" → 確認済

#### スコア妥当性

8.0/10は妥当。JAXA「きぼう」実モジュール内・減圧事象という題材は、一般的なLLMマルチエージェントシミュ（街・市場・チャット型ロールプレイ）から大きく外れたユニークな選択。物理曲線も「致命域に入らない」と意図的に安全圏で設計された点は、研究的観察対象として理に適っている。

#### 見落とし・誇張

- 軽微な見落とし: 元評価で減点理由として「単一ケース（trapped_depress_v1のみ）」を挙げているが、v0.4ミッション（Int-Ball2探索）の痕跡が `brain_loop_node.py` の `MISSION_V04` 定数や `claude_client.light_detect()` に残っており、過去シナリオも実装されていた。1ケースのみという指摘は現行設定の話だが、シナリオ拡張機構（YAML切替）は既に動作実績があると言える
- 誇張は見当たらない。「業界応用先（宇宙オペレーション・有人ミッション支援）」の評価は妥当

#### 検証結論

8.0/10は妥当（やや甘めも許容範囲）。題材独自性が高く減点要因も明示されているため整合的。

---

### C. 発展性 — 元スコア 6.0/10

#### 根拠の検証 [OK]

- `scripts/brain/sensor_buffer.py` のSensorBuffer独立クラス・history_secパラメータ化 → 確認済（`def __init__(self, history_sec: float = 60.0)` がファイル先頭9行目）
- `src/vlm_bench/vlm_smoke_node.py` のVLMプロバイダ切替（claude/nim/ollama）→ 確認済。`_load_client()` 関数で `SPD_VLM_PROVIDER` 環境変数によりclaude/nim/ollamaを分岐
- `config/scenarios/trapped_depress_v1.yaml` の完全外部化 → 確認済
- `environment_sim.py:120-132` 近辺の動的ロード → 確認済。`load_scenario()` 関数が `config/scenarios` ディレクトリから動的にYAMLを読み込み
- README.md 不存在 → 確認済。`find -iname "README*"` で出力ゼロ、リポジトリ全体に README.md は存在しない
- `docs/report/architecture.md` に Future Work セクションなし → 確認済。grepで "future|roadmap|todo|next" にマッチする該当セクションなし（"next cycle" のみ）
- `scripts/brain_loop_node.py:100-104` のトピック名ハードコード → 確認済。`/humanoid_01/image_raw`, `/humanoid_01/odom`, `/humanoid_01/cmd_vel` 等が文字列直書き

#### スコア妥当性

6.0/10は妥当。モジュール分割（scripts/brain, src/vlm_bench/clients, config/scenarios）は実際に機能的、シナリオ追加はYAML一本で可能という拡張性評価は事実。一方、README欠落・Future Work不在は判定者にとって致命的なマイナス。

#### 見落とし・誇張

- 軽微な見落とし: NIMクライアント・Ollamaクライアントの実体ファイル群（`scripts/clients/`配下の想定）は部分的にしか実装されていない。元評価は「意図した構造」と適切に表現しており誇張ではない
- 誇張は見当たらない

#### 検証結論

6.0/10は妥当。README不在の重み付けが効いており、整合的。

---

### D. 技術実装 — 元スコア 8.0/10

#### 根拠の検証 [OK]

- `claude_client.py` のtemperature=0.4, max_tokens=600 → 確認済。decide()内の `messages.create()` 呼び出しに該当パラメータあり
- トークン料金計算とコスト追跡（`_calc_cost`, `_INPUT_PRICE_PER_M=3.0`, `_OUTPUT_PRICE_PER_M=15.0`）→ 確認済（ファイル先頭）
- `_parse_json` でJSON抽出、例外時None返し → 確認済（`raw.find("{")` / `raw.rfind("}")` 実装）
- parseエラー時フォールバック辞書 → 確認済。decide()末尾で `decision = _parse_json(raw) or {"observation": "", ...}`
- `brain_loop_node.py` の memory継続ロジック（"parse error" 時は直前memory維持）→ 確認済。step() 内で `if new_mem and new_mem \!= "parse error":` の分岐あり
- `memory[-5:]` + `action_history[-8:]` を decide() に注入 → 確認済（brain_loop_node.py step()内）
- action_historyブロックの整形（cycle/action/pos/concern/comms_sent）→ 確認済。claude_client.py で `f"cy{e[cycle]:02d}: {e[action]:<14} pos=..."` の組み立て
- `docs/report/architecture.md:71-78` の「Self-Feedback Memory」「Action History」明記 → 確認済（Key Design Decisions セクション）
- MultiThreadedExecutor → 確認済。main()内で `rclpy.executors.MultiThreadedExecutor()` 使用
- thread-safe 画像バッファ → 確認済（image_buffer.py の threading.Lock 使用）
- `CMD_VEL_TIMEOUT = 0.5` → 確認済（humanoid_ros2_sim.py 行58）
- `_lerp` 線形遷移・`max(val, tr["to"])` 下限クランプ → 確認済（environment_sim.py 29-31, 76-82）

#### スコア妥当性

8.0/10は妥当。Self-Feedback Memory + Action History の二重設計は確かに「特筆すべき」レベルで、単なるログ注入ではない情報設計が行われている。コスト計測（USD換算）、レイテンシ計測、JSONパース失敗時のフォールバック、parse errorのメモリ継続防止など、運用面の配慮が随所にある。

#### 見落とし・誇張

- 見落とし: テストコードが `tests/test_image_buffer.py` の1ファイルのみであり、テストカバレッジは限定的。元評価は「コード品質は良好」と評しているが、テスト面の弱点には触れていない（CodeQ評価でやや甘い可能性）
- 見落とし: `_parse_json` の `raw.find("{")` / `raw.rfind("}")` 実装は、JSON内にネストされた `{}` を含む文字列があると壊れる可能性のあるナイーブな実装。技術実装スコアをやや押し下げる要因
- 誇張: 「リトライ未実装」「決定論的再現未対応」を減点理由として正しく挙げているため誇張は無い
- 軽微な誇張: 「型ヒント、loggerの整備、docstring、関心の分離」と総括しているが、`brain_loop_node.py` の `step()` メソッドはやや長く、責務（センサ取得・LLM呼出・ログ・アクション発行）が混在気味

#### 検証結論

8.0/10は妥当（テストカバレッジを厳しく見れば7.5の余地もあるが、Self-Feedback Memoryの設計価値で相殺）。

---

## 総合所見

### 評価の精度

元評価は、引用された行番号・引用文字列の精度が極めて高い。全ての主要citationを実ファイルで照合した結果、齟齬は確認されなかった。特にプロンプト末尾の "Survival guidance" 文、`MISSION_V05` の "explore your surroundings, report to ground" 文、YAML内の致命域コメント等は引用テキストと実体が完全に一致。

### 評価運用上の問題

- 実施回数の表記不整合: 元評価ヘッダーで「実施回数: 2回」とあるがスコアサマリーには Run1/Run2/Run3 列があり Run3 が「-」となっている。skill定義（最大3回平均）との関係で、2回実施で打ち切った理由が明示されていない点はマイナー。最終スコアは Run1=28・Run2=27 の平均=27.5 で計算は正しい

### 総合判定

元評価27.5/40は妥当。各カテゴリのスコアは ±0.5 の調整余地にとどまり、結論を覆すような重大な見落としや誇張は確認されなかった。特にBとDがやや甘めの感はあるが、Self-Feedback Memory の設計とJEM減圧シナリオの物理リアリズムへの加点は理にかなっている。一方で改善点として挙げられた「マルチエージェント化」「README整備」「行動ナッジ除去」「再現性強化」はいずれも次バージョンに直結する具体的・実施可能な提案で、レビューとしての建設性も高い。

### 検証で確認した重要事実

- README.md は本当に存在しない（`find -iname "README*"` で出力ゼロ）
- エージェントは humanoid_01 1体のみ（トピック名・USDパス・全コードを横断確認）
- VLMプロバイダ抽象（claude/nim/ollama）は意図として実装されているが、実体ファイルとしては部分的
- テストは `tests/test_image_buffer.py` の1本のみ
- v0.4 (Int-Ball2 探索) の痕跡が残り、シナリオ拡張機構の動作実績はある
