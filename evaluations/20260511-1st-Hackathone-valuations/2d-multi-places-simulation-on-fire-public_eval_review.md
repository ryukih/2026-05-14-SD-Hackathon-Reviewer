# 評価検証レポート: 2d-multi-places-simulation-on-fire-public

**検証日**: 2026-05-11
**対象**: `2d-multi-places-simulation-on-fire-public_eval.md`
**元評価合計**: 33.0/40

## 検証サマリー

| カテゴリ      | 元スコア | 検証結果 | 引用検証 | 妥当性 |
|-------------|---------|---------|---------|-------|
| A. 創発設計  | 9.0/10  | 妥当     | 全て一致 | 高     |
| B. 世界設定  | 7.0/10  | 妥当     | 全て一致 | 高     |
| C. 発展性    | 8.0/10  | 妥当     | 概ね一致 | 高     |
| D. 技術実装  | 9.0/10  | 妥当     | 概ね一致 | 高     |
| **合計**    | **33.0** | **支持**| -        | -      |

**総合判定**: 元評価のスコア・引用・分析は、いずれもソースコードと観察ファイル群で裏付けられる。引用された行番号は実ファイルと数行のズレが見られるが、対象とする関数・処理の範囲は正確であり、評価の本質的妥当性は揺らがない。冒頭の「派生作品である」という注記（原作 = 兵頭氏、追加 = Su 氏）は提出物の README とも整合し、評価の透明性を高めている。

---

## カテゴリ別検証

### A. 創発設計（元: 9.0/10）

**根拠の検証**:
- ✓ `agent.py:50-56` — `fire_entry` テンプレ。実ファイルでは行 50-56 がまさに `Position / Intensity / Radius / Distance` のみの英語テンプレに該当し、`ja` 版も行 119-125 で同一構造。「定性的形容詞ゼロ」は事実。
- ✓ `agent.py:43-48` — `place_status_template`。実ファイル行 43-48 が `Number of agents here / Capacity / Occupancy rate` のみ。一致。
- ✓ `agent.py:376-399` — `_build_fire_section` のコメント。実ファイル行 376-399 に「Only quantitative data is provided ... No qualitative descriptions ... are included.」と明記されており完全一致。
- ✓ `simulation.py:299-322` — `get_fire_info_for_agent` の Model B 実装。実ファイル行 299-322 にて `if distance <= fire['radius']` で配信を絞り、無ければ `None` を返す。「Implements Model B」のコメントも実在。
- ✓ `agent.py:286-309` — `get_nearby_agents` の通信ルール。実ファイル行 286-309 にて「両方とも場所外」「同一場所内」のみで `same_area` 成立、それ以外は不可、というルールがそのまま実装されている。
- ✓ README 引用「エージェントには生の数値データ（占有率、火災の強度・距離等）のみを提供し、行動指示や定性的評価（「危険」「快適」等）は一切与えない」は README L14 にそのまま存在。
- ✓ `SUBMISSION.md` の "manageable" 観察例は SUBMISSION.md L45 / L52 に存在し、「intensity=0.8 にも関わらず "manageable"」という解釈は実観察ログに基づく。

**スコア妥当性**: 9/10 は妥当。世界ルールが「数値のみ提示」を `agent.py` 内のテンプレ単位で徹底し、行動空間（move/stay × 4方向 + 任意メッセージ + 自由 memory）が完全に開かれ、通信制約が情報非対称性を構造的に生む設計は、ルーブリック「9-10 precise+raw data」に強く該当する。提出物に同梱された SUBMISSION_*.md 群が、設計意図通りの創発（"manageable" 再解釈、舞踏場・個室の集団共有、防災語彙の集団的補完）を複数モデル × 複数 run で実証している点も加点根拠として妥当。

**見落とし・誇張**:
- ほぼ無し。あえて指摘するなら、創発設計の「設計」は原作（兵頭氏）側に帰属するため、提出者 Su 氏の固有貢献としてのスコアではなく「成果物全体としての設計品質」のスコアであることが冒頭注記で明示されており、誇張ではない。

**検証結論**: 9.0/10 を支持する。

---

### B. 世界設定（元: 7.0/10）

**根拠の検証**:
- ✓ `config.yaml:21-33` — left_bar (capacity 12), right_bar (capacity 10)。実ファイル行 22-33 に該当（行 21 はコメント "Each place must have a type" の続き）。capacity 値・非対称性は正確。
- ✓ `config.yaml:48-61` — fires2件、fire_1 right_bar直上を覆う半径15、fire_2 反対側半径8。実ファイル行 49-63 にて fire_1: start_step 35, intensity 0.8, radius 15, center (15, 10) / fire_2: start_step 70, intensity 0.5, radius 8, center (-10, -10)。right_bar 中心 (15, 0) と fire_1 中心 (15, 10) の距離は10で、半径15内に right_bar 全体が収まる。「直上を覆う」も妥当。
- ✓ README 引用「LLMエージェントの創発性と自律性を発現させることを目的とした」は README L13 に完全一致。
- ✓ README 引用「フィールドに複数の場所が存在し、各場所は独立した収容上限を持つ」は README L15 に存在（"フィールドに複数の場所（デフォルトでは左側と右側にバーが1つずつ）が存在し、各場所は独立した収容上限（capacity）を持つ"）。
- ✓ `SUBMISSION_ja_d50_qwen25_7b.md` 引用「出口 634 / 消防 584 / 煙 477 / 温度 443」は SUBMISSION_ja_d50_qwen25_7b.md L167-170 にそのまま存在。
- ✓ 「場所のtypeは文字列ラベルのみで、type固有の機能差は実装されていない」は事実。実装上 `place['type']` は表示用 string であり、bar/cafe/library 間のルール差は無い。

**スコア妥当性**: 7/10 は妥当。「2D空間で複数のバー＋火災」というテーマ自体は災害シミュレーションの一典型で独創性は中庸（ルーブリック「5-6 generic」）。一方、(1) capacity による社会的圧、(2) 知覚モデルB による情報非対称、(3) ペルソナ（性別）の組み合わせが「定量情報のみからの集団規範創発」という社会科学的問いに直結している点で「7-8 interesting」帯に位置付けるのが正当。

**見落とし・誇張**:
- 元評価は SUBMISSION で観察された「存在しない舞踏場・個室の集団的共有」「実在しない消防設備の集団生成」を世界の補完現象として正しく評価している。
- 限界として「type 機能差なし」を率直に指摘しており、誇張は見られない。

**検証結論**: 7.0/10 を支持する。

---

### C. 発展性（元: 8.0/10）

**根拠の検証**:
- ✓ `simulation.py:75-94` — `fires_config` リスト処理、`center_x/y` 省略時はランダム。実ファイル行 75-93 にて `fires_config = self.config.get('fires', [])` から `if 'center_x' in fc and 'center_y' in fc:` の分岐実装。一致（範囲は1行短い 75-93）。
- ✓ `simulation.py:52-69` — places のスキーマバリデーション。実ファイル行 50-71 にて `required_fields = ['name', 'type', 'center_x', 'center_y', 'half_size', 'capacity']` と検査。範囲は若干広いが趣旨一致。
- ✓ `utils.py:17-24` — `PlaceConfig` TypedDict。実ファイル行 17-24 にて該当 TypedDict 定義あり。完全一致。
- ✓ `agent.py:32-169` — `PROMPT_TEMPLATES` 辞書。実ファイル行 32-169 にて en/ja の2言語テンプレ辞書が存在。完全一致。
- ✓ `agent.py:174-209` — sys.argv 経由の言語検出。実ファイル行 174-209 にて `_detect_prompt_language` が `--config` フラグを sys.argv から読み取り、yaml をパースして `prompt_language` キーを返す。完全一致。
- ✓ `PROMPT_LANGUAGE.md` 引用「既存 `config.yaml` は 1 バイトも変更しない」「`simulation.py`, `ollama_client.py`, `visualization.py` は触らない」は PROMPT_LANGUAGE.md L20-21 に完全一致。
- ✓ README 引用「将来の拡張例（カフェや図書館など）」は README L247 に存在し、YAML 例も L249-260 に同梱されている。
- ✓ `CLAUDE.md` の感度実験計画（communication_radius, temperature）は CLAUDE.md L194-198 に存在。
- ✓ モデル別 config 同梱（gemma3, qwen2.5/3, llama3.2, sarashina, swallow, elyza, phi4）は `ls submissions/.../config_*.yaml` で確認済み。事実。

**スコア妥当性**: 8/10 は妥当。モジュール分離（agent / simulation / ollama_client / visualization / utils）は明確、TypedDict による設定型安全、YAML 駆動の場所・火災追加、`simulation.py` 不変制約下での日本語化拡張の実証が揃っている。「ビジネス・社会科学マッピング不在」を減点として正しく指摘している。

**見落とし・誇張**:
- 元評価は「`PROMPT_TEMPLATES` + `sys.argv` 検出による後付け拡張」を「拡張容易性の実証」として高く評価している。妥当。
- やや甘い可能性: 言語切替は `agent.py` 内に大きな辞書を追加する形であり、純粋な「拡張ポイント」としては設計されていない（後付けハック的）。とはいえ、`simulation.py` を不変にできた点は本物の拡張性指標。総合 8/10 は妥当範囲。

**検証結論**: 8.0/10 を支持する。

---

### D. 技術実装（元: 9.0/10）

**根拠の検証**:
- ✓ `agent.py:600-651` — JSON 抽出と方向キーワードフォールバック。実ファイル行 600-634 が `_extract_json_from_text`（ネストブレース・文字列内ブレース無視・エスケープ対応）、行 637-650 が `_extract_direction_from_text`。元評価の範囲指定は若干広めだが対象は正確。
- ✓ `agent.py:415-490` / `492-598` — Phase 1 / Phase 3 のプロンプトビルダー分離。実ファイル行 415-490 が `create_message_prompt`（位置情報なし）、行 492-598 が `create_decision_prompt`（位置情報あり）。行範囲完全一致。
- ✓ `simulation.py:324-422` — 4-Phase 同期実行。実ファイル行 324-422 が `step_simulation`、コメントに「New order: 1. ... 2. ... 3. ... 4. ...」が明記。一致。
- ✓ `agent.py:733-762` — `decide_action` が `decision['memory']` を `Step N: {memory_content}` 形式で履歴に追加し、`memory_limit` で先頭削除。実ファイル行 733-762 にて該当処理あり。完全一致。
- ✓ `ollama_client.py:45-97` — `repeat_penalty / repeat_last_n / min_p` を options に含め、`RequestException` を捕捉。実ファイル行 45-97 にて該当実装あり。完全一致。
- ✓ `simulation.py:130-176` — `_log_message` と `_log_memory_reasoning_batch`。実ファイル行 130-176 に該当（batch I/O 化のコメントも実在）。一致。
- ✓ `config.yaml:36-45` — LLM サンプリングパラメータ。実ファイル行 36-46 に該当。`temperature: 0.2`, `repeat_penalty: 1.1`, `repeat_last_n: 128`, `min_p: 0.05` は事実。
- ✓ `agent.py:725-731`, `760-762` — try/except でフォールバック返答。実ファイル行 725-731 が `decide_message` の例外処理、行 760-762 が `decide_action` の例外処理。一致。
- ✓ `agent.py:364-374` の `_build_messages_context` で `from Agent X:` 形式整形。実ファイル行 363-374 に該当。一致（1行差）。
- ✓ `agent.py:777-793` の `receive_message` でメッセージ保存と上限管理。実ファイル行 777-794 に該当。一致。
- ✓ 減点根拠「乱数シードなし」「テストなし」: `grep seed config.yaml` でヒット無し、`tests/` ディレクトリ無し、`*.py` 内に `random.seed` 呼び出し無し。事実。

**スコア妥当性**: 9/10 は妥当。LLM(3) は2フェーズプロンプト分割・JSON 強堅パーサ・フォールバック・サンプリング調整で満点に近い水準。Memory(3) は LLM 自己生成 `memory` の自己フィードバック + `memory_limit/memory_size` 分離 + `from Agent X:` 形式整形で高水準。CodeQ(2) は TypedDict / 定数化 / docstring / 命名が良好。SimCorrect(2) は 4-Phase 同期実行 + 構造化ロギングで高水準だが、シード未固定とテスト不在で減点 -1 が妥当。合計 9/10 は支持される。

**見落とし・誇張**:
- 元評価は「Phase 1 で位置情報なし → メッセージで先回り計算結果を漏らさない」という設計意図を正しく評価しており、これは `simulation.py:324-422` のコメント「New order」と整合する。
- 減点 1点（シード・テスト不在）は LLM ベース確率的シミュレーションとしては妥当。シード固定しても LLM 内部のサンプリングは制御不能だが、`random` モジュールの初期配置とランダム火災位置は固定可能。指摘は適切。
- やや甘い可能性: `received_messages` の保存に step 番号が付くが、`_build_messages_context` ではメッセージ内容を `from Agent X:` で並べるだけで step 順や時系列の重みづけは無い。これは設計判断であり明確な欠陥ではないが、技術実装の精密性をやや穏やかに表現する余地はある。

**検証結論**: 9.0/10 を支持する。

---

## 総合所見

### 元評価の品質
- **引用精度**: 行範囲は最大 4 行程度のズレがあるが、対象関数・対象テンプレ・対象コメントはすべて実ファイルに存在し、引用内容は正確。
- **README 引用**: 3箇所いずれも逐語的に存在を確認できる。
- **観察ログ引用**: `SUBMISSION.md` の "manageable"、`SUBMISSION_ja_d50_qwen25_7b.md` の語彙統計（634/584/477/443）と発話量（1390→756→996）はすべて該当ファイルに完全一致で存在。
- **冒頭注記の透明性**: 「原作 = 兵頭氏、追加 = Su 氏」という責任分界の明示は、ハッカソン提出物の特殊な性質（派生作品）を評価する上で重要な透明性を確保している。
- **総合判定**: ルーブリック準拠かつ引用裏付けが堅実な、信頼できる評価。

### 主要な指摘点
- A・D が 9 点と高水準だが、これは「原作の設計品質 + 追加された日本語化／多モデル方法論／観察ログ」が合わさった成果物全体への評価としては妥当な水準。
- B は世界テーマの独創性が中庸である点を率直に減点しており適切。
- C は「拡張容易性の実証としての PROMPT_TEMPLATES 後付け」を加点根拠としているが、これは「拡張点として設計された」ものではなく「制約下での後付けハック」であることに留意。それでも `simulation.py` 不変で実現できた事実は拡張性の十分な間接証拠。

### 推奨アクション
- そのまま採用可能。スコア修正は不要。
- もし精密性をさらに上げるなら、行番号を「±2行精度」で再確認するか、`L20-L33` のような正確な範囲を再記載すると良い。ただし現状の引用でも対象が一意に特定できるため、運用上の支障はない。

---

## 100語サマリー

The evaluation (33.0/40) is well-supported by the source code. Every cited line range maps to the correct function or template within a few lines of tolerance, and every README/SUBMISSION quote exists verbatim in the corresponding file. The "manageable" reinterpretation, the 634/584/477/443 vocabulary counts, and the 1390-756-996 message-volume variance are all genuine observations. The transparent attribution (original = Hyodo, additions = Su) strengthens credibility. Scores of 9/7/8/9 are each justifiable under the rubric, with appropriate deductions for missing seed control, missing tests, no place-type differentiation, and absent business/social-science mapping. No corrections needed.
