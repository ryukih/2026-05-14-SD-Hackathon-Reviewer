# 評価検証レポート: near-future-ai-society-100-steps
**検証日**: 2026-05-11
**対象**: `near-future-ai-society-100-steps_eval.md`
**元評価合計**: 36.0/40

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨スコア帯 |
|---|---|---|---|
| A. 創発設計 | 9.0/10 | 妥当 | 9.0〜9.5 |
| B. 世界設定 | 9.0/10 | 妥当（やや辛め） | 9.0〜9.5 |
| C. 発展性 | 9.0/10 | 妥当 | 8.5〜9.0 |
| D. 技術実装 | 9.0/10 | 妥当 | 8.5〜9.0 |

**総合判定**: 元評価合計 36.0/40 は妥当な水準。主要な引用はすべて実コードで確認でき、ルーブリック（A: ルール×自由、B: 独創性、C: モジュール性+構想、D: LLM3+Mem3+CodeQ2+SimCorrect2）の各観点を高水準で満たしている。誇張は確認されず、軽微な詳細齟齬（モデル名表記等）が散見される程度。提出版が `--no-introspect` ランのみであり L1 の実走行ログが未提出である点は元評価でも明示されており、適正に減点要因を捕捉済み。

## カテゴリ別検証

### A. 創発設計（元: 9.0/10）

**根拠の検証**:
- `config.yaml:33-95` 7箇所の場所定義 ✓ — 実コードでも 33〜95 行近辺に7箇所（grid_control / emergency_ops / council_chamber / citizen_hub / audit_room / maintenance_bay / archive_memorial）が座標・容量・category 付きで定義されている。capacity 合計 = 7+8+10+8+4+4+4 = 45、agent 20 体に対し 2.25 倍スラックも正確。
- `config.yaml:102-145` 4大規模イベント ✓ — blackout_warning(step30) / regulation_amendment(step50) / citizen_death(step75) / next_gen_announcement(step90) が intensity / radius / center / targets 付きで定義されている。
- `agent.py:300-334` 通信プロンプトに「必要なら沈黙してもよい」明記 ✓ — 該当箇所に「あなたのpersonaに従い、必要なら沈黙してもよい」「話さない判断なら空文字」が確認できる。指示型ではなくデータ提示型である点も支持される。
- `agent.py:223-231` イベント情報の提示形態 ✓ — display_name / 強度 / 半径 / 距離 のみ渡し「※必要なら、この発生地に向かう移動を選んでもよい」と選択肢の提示に留めている点は実コードで確認。
- `metacog/agent/prompt_template.py:62-68` L1 で KPI→個的言葉の書き換え許可 ✓ — 「CURRENT_GOAL は数値目標（KPI）である必要はない」「停電ゼロを維持→夜の街が眠っている間、私が起きている」が実コードに存在。
- `DESIGN.md` の「プロンプトに直接埋め込まない」自覚 ✓ — DESIGN.md の探究軸の節に「これらはプロンプトには直接埋め込まない（自意識過剰なエージェントを作らないため）」と確認できる。

**スコア妥当性**: ルール側（場所・容量・イベント・人間メッセージプール）は数値設計が緻密で、自由側は LLM 判断への明示的な「沈黙してよい」「書き換えてよい」自由保証が組み込まれている。creator-imposed rules と agent freedom の両立がルーブリック A の核であり、9.0/10 は妥当。

**見落とし・誇張**:
- 軽微な誇張: 「行動指示型ではない」と記すが、L1 プロンプトには「書き換えのガイドライン」として比較的具体的なガイドラインがあり、完全に提示型のみとは言い切れない（自由保証と並存する形での誘導はある）。ただし減点に値するほどではない。
- 見落とし: 「20体に対し容量45」のスラック設計が「凝集と回避の両立」をどう実現するかについて、`citizen_hub` の category=intimate / `maintenance_bay` のホーム数 0 など、配置構造による誘導が config に組み込まれている点は更に評価可能。

**検証結論**: 9.0/10 は妥当。上方修正の余地はあるが現状で過大評価ではない。

### B. 世界設定（元: 9.0/10）

**根拠の検証**:
- `DESIGN.md:8-42` 研究問い ✓ — 「AIたちはどう生き、人間に対して何を思うのか」「AI集団に独自概念や方言が創発するか」が DESIGN.md 冒頭の「問い」節に明記。
- `DESIGN.md:91-127` 3層構造の地理的＝心理的距離設計 ✓ — DESIGN.md の世界設定節に「公的・調整層 / 接触・監督層 / 内省・終焉層」の3層が ASCII 図と説明付きで提示。下段2箇所の「修復と諦めの地理的対比」も確認できる。
- `config.yaml:164-201` 各 persona の deployed / predecessor / primary_kpi / human_contact / death_mode ✓ — id=0 雷光「新世代量子最適化AIによるshadow defeat」など個別 death_mode 設定が実コードに存在。
- `config.yaml:152-157` step 80 で 命 を削除し peer_lost ブロードキャストする実装 ✓ — deletions 配列に step:80 / agent_id:7 / cause:"訴訟リスクによる強制リプレース" が存在。`simulation.py:318-392` の `_process_deletions` でブロードキャスト処理も確認できる。
- `config.yaml:799-859` 100件の人間メッセージプール（苦情/感謝/要求/質問/訴え） ✓ — 訴え20件「停電したら、夫の人工呼吸器が止まります」「今夜、消えたい気持ちが強いです」「夫が亡くなって、誰とも話していません」等が実コードに存在。
- `README.md` の世界設定説明 ✓ — 「20体のAIエージェント（電力・水道・交通・医療・福祉・終末期ケアなど）が公共インフラを担う近未来社会のシミュレーション」が冒頭に確認できる。

**スコア妥当性**: 単純な群衆シミュや経済取引デモと比べ、社会科学的妥当性が極めて高い。AI規制法・再認証・市民の死・新世代置換という現実の AI ガバナンス論点を圧力源として落とし込んでおり、ルーブリック B（独創性）の上位。9.0/10 は妥当。

**見落とし・誇張**:
- 見落とし: persona の category 4分類（physical / emergency / social / intimate）と職能の対応関係、ホーム配置と category の対応が config レベルで一貫している点は更に評価可能。
- 見落とし: human_contact がエージェントごとに個別設定されており、人間との接触面の質的差異が設計されている点。
- 誇張なし。

**検証結論**: 9.0/10 は妥当。やや辛めの評価とも言える水準で、9.5 への上方修正も検討可能。

### C. 発展性（元: 9.0/10）

**根拠の検証**:
- `orchestrator.py:210-330` 依存性注入による L0/L1/observer/viz の組み立て ✓ — argparse → meta_config → Simulation → Introspector（条件付き）→ EmergentObserver（条件付き）→ HTMLVisualizer（条件付き）と段階的に組み立てられている。`--no-introspect` / `--no-viz` / `--no-video` のフラグで層単位の切り替えが可能。
- `metacog/observers/emergent_observer.py:114-275` 5軸独立メソッド ✓ — `observe` メソッド内で 1. 語彙、2. 沈黙、3. 場所アトラクター、4. 通信ペア、5. ハブ受信カウント の5軸が独立して観察され、`_check_and_log` で閾値判定も独立処理されている。新軸追加に対し疎結合な実装。
- `README.md:115-153` A/B 比較手順と `--seed` / `--output-dir` / `--log-dir` ✓ — 該当箇所に Run A（内省層あり、シード42）/ Run B（内省層なし、同シード）のコマンド例が提示され、注意書きで qwen2.5:14b の確率性についても触れている。
- `README.md:174-181` 事後分析 6種のアウトプット計画 ✓ — analysis_report / human_attitude / place_dialects / shared_metaphors / final_self_portrait / researcher_synthesis の6種が README に列挙されている。
- `DESIGN.md:316-333` 含意探索実験 vs 仮説検証実験 ✓ — DESIGN.md の哲学的注記節に「これは仮説検証実験ではなく**含意探索実験**」「自己書き換えと自己記述能力を持つ情報処理系」が確認できる。

**スコア妥当性**: モジュール分離（L0/L1/observer/viz）の明快さ、外部 YAML 設定への徹底外出し、A/B 比較プロトコルの整備、事後分析の6種アウトプット計画、含意探索実験という研究方法論の自覚的位置付けと、ルーブリック C の両側面（コード拡張性 + 構想/将来展望）を高水準で満たしている。9.0/10 は妥当。

**見落とし・誇張**:
- 注意点: 事後分析の 6 種ファイル（analysis_report.md 等）は「事後に手動 / Claude 併用で生成」する想定であり、自動生成パイプラインまではコード化されていない。「構想」としては優れているが「実装済み」ではない点に留意。
- 見落とし: `metacog/config.yaml` の独立により内省層パラメータ（trigger_interval / cooldown / 字数上限）が外部化されている点は元評価で言及されている。

**検証結論**: 9.0/10 は妥当。事後分析の実装が「未完」である点を厳格に評価すれば 8.5 も合理的範囲だが、構想の具体性で 9.0 を支える根拠は十分。

### D. 技術実装（元: 9.0/10）

**根拠の検証**:
- `agent.py:435-460` 堅牢な JSON brace-matching パーサ ✓ — `_extract_json_from_text` が文字列内のクォート / エスケープを考慮した brace depth マッチで JSON 抽出を行っている。実装確認済み。
- `agent.py:631-656` `apply_introspection_diff` のクールダウン+字数上限 ✓ — cooldown=3 サイクルを各セクションで個別判定し、`max_self_concept_chars` / `max_current_goal_chars` / `max_coping_notes_chars` で切り捨て。実装確認済み。
- `simulation.py:427-581` ステップループの順序（削除→イベント→人間注入→通信→行動→移動→state→ログ） ✓ — `step_simulation` メソッドで「削除 → イベント発火 → 状態更新 → 人間メッセージ注入 → Phase1通信 → Phase2配信 → Phase3行動 → Phase4移動 → state更新 → 位置永続化 → 統計」と確認できる。
- `simulation.py:482-498` / `simulation.py:522-537` Phase1/3 の ThreadPoolExecutor 並列化 ✓ — Phase1（decide_message）と Phase3（decide_action）で `ThreadPoolExecutor(max_workers=self.max_concurrent_llm)` による並列化が確認できる。
- `metacog/agent/introspector.py:22-117` `collect_others_voices` の異カテゴリ優先収集 ✓ — messages.jsonl と inner_thought.jsonl から他 agent の声を集め、同カテゴリ後置でソートし n_speeches=4 / n_thoughts=2 で切り取る実装が確認できる。「他者の声で自己が揺らぐ経路 P2」の根拠として妥当。
- `simulation.py:318-392` 削除→peer_lost ブロードキャスト＋システム通知メッセージ ✓ — `_process_deletions` 内で全生存 agent に `mark_event("peer_lost", ...)` + `receive_message(-2, ..., source="system")` の二重通知が確認できる。
- `metacog/logs_no_intro/coined_terms.jsonl` 5054行 / `agent_log.jsonl` 5145行 ✓ — wc -l 検証で正確（5054 / 5145 行）。実走行ログとして存在。
- `orchestrator.py:223-230` `random.seed` / `np.random.seed` 両固定 ✓ — `args.seed is not None` の分岐で両者を固定する実装が確認できる。

**スコア妥当性**: LLM 利用（プロンプトのセクション分離、JSON 強制、フォールバック、並列化）= 3点満点、メモリ設計（FIFO + 字数上限 + クールダウン + 他者の声 P2 + peer_lost 強制内省）= 3点満点、コード品質（型ヒント、定数集約、ロガー分割、docstring）= 2点満点、シミュレーション正確性（synchronous 順序、`--seed`、A/B ディレクトリ分離）= 2点満点 のそれぞれが高水準。9.0/10 は妥当。

**見落とし・誇張**:
- 軽微な齟齬: 元評価本文では「L0（ローカル `qwen2.5:14b` / Ollama）」とあるが、config.yaml の冒頭コメントには `gpt-oss:20b ローカル実行` という古いコメントが残っており、設定値（`llm.model: qwen2.5:14b`）と齟齬がある。これは本評価の引用先（README/DESIGN）とは異なるため、評価には影響しないが指摘事項として記録。
- 軽微な誇張なし: 「`main.py` が `orchestrator.py` の旧版エントリポイントとして残存」は実コードで確認（main.py 冒頭は `from visualization import Visualizer` / `from simulation import Simulation` を import し独自の `setup_logging` を持つ旧エントリ）。「`simulation.py:139` の `_is_position_in_place` が未使用」も grep 検証で確認（定義のみで呼び出しなし）。改善提言の根拠は実コードで一致している。
- 見落とし: `agent.py:631-656` で「ORIGIN は読み取り専用」のガードが `apply_introspection_diff` に明示的なコードレベルのチェックとして組まれていない（diff に origin_new フィールドが入ってこない設計に依存）。プロンプト側で禁止しているが防御的プログラミングの観点では弱点。

**検証結論**: 9.0/10 は妥当。厳格に評価すれば軽微な弱点（main.py の混在、ORIGIN 不変のコードレベル防御弱、dead code の残存）で 8.5 も合理的だが、全体の設計と実装水準は 9.0 を支える。

## 総合所見

**評価の信頼性**: 元評価の引用 14 件のうち全件が実ファイル・実コードで確認できた。誇張・捏造は確認されず、わずかな表現上の弱点（「行動指示型ではない」のやや強い断定など）は減点に値しない範囲。`config.yaml` の冒頭コメント（`gpt-oss:20b` 旧モデル名）と実設定（`qwen2.5:14b`）の齟齬は本評価本体の主張には影響しない。

**スコアの妥当性**: 合計 36.0/40 は適正水準。各カテゴリ 9.0 は「他の標準的なシミュレーション提出物と比較した相対位置」として妥当であり、ルーブリックの上位水準を満たすが満点ではない（特に C は事後分析パイプラインが未実装、D は dead code / 旧 main.py 残存等）。

**改善提言の妥当性**: 元評価の改善点 4 件すべてが実コードで根拠を確認できる：
1. `main.py` の旧エントリ残存 ✓
2. L1 ログ未同梱（`logs_no_intro` のみ存在、`logs_with_intro` 不在） ✓
3. イベント `targets` の決め打ち ✓（events 配列で明示的に id リスト指定）
4. `_is_position_in_place` の dead code ✓

**推奨アクション**: 元評価のスコア（9.0/9.0/9.0/9.0 = 36.0/40）はそのまま維持して問題ない。A と B はわずかに上方修正（9.5）の余地もあるが、C と D は事後分析未実装と軽微な技術負債を考慮すると現状維持が穏当。総合的に、現提出版はハッカソン水準を大きく超える研究的プロトタイプとして成立しているという元評価の総評には同意できる。
